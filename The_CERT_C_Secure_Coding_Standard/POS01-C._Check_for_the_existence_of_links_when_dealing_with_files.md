原文链接： https://www.securecoding.cert.org/confluence/display/seccode/POS01-C.+Check+for+the+existence+of+links+when+dealing+with+files

# 处理文件时先检查链接是否存在

常见操作系统（如：Windows、Unix）支持“硬链接”、“软链接（符号链接）”、和“虚拟磁盘”。

* 创建硬链接
    * Unix中使用ln命令
    * Windows中通过调用 `CreateHardLink()` 函数
* 创建符号链接
    * Unix中使用 `ln -s` 命令
    * Windows 则有几种方式：NTFS使用目录节点，或 Linked.exe ，或“junction”软件。
* 虚拟磁盘
    * Windows中使用 `subst` 命令创建。 

程序如果没有正确处理（考虑到）已打开的文件是对另一个文件的链接，会有一个安全告警，但这对运行在高级权限模式但健壮性不强的程序来说是非常危险的。 

通常不需要检查符号链接的目标是否存在，而是通过其他某种手段解决。比如：当打开一个存在的文件，最直接的方式是降低用户权限，当该用户对文件（包括一般文件，和链接文件的目标文件）权限不够时，系统会阻止访问。

创建新文件，通常只会在没有同名文件的地方创建，以阻止已有文件被覆盖。 

特殊情况下，有必要检查是否存在符号、或硬链接，以保证程序读取的文件是正确的，而不是另外一个目录下的另一个文件。但要注意的是：检查符号链接的时候需要避免产生竞争条件（参考 POS35-C）。 

## 不符合要求的代码示例

下面的代码示例打开一个指定的文件，读写，然后写入用户提供的数据。 

```c
char *file_name = /* something */; 
char *userbuf = /* something */; 
unsigned int userlen = /* length of userbuf string */;   

int fd = open(file_name, O_RDWR); 
if (fd == -1) {
    /* handle error */
} 

write(fd, userbuf, userlen); 
```

如果程序运行在高级权限下，黑客就能够利用此代码展开攻击，如：将指向 /etc/passwd 鉴权文件的符号链接文件替换为 file_name 指定的文件，运行程序会打开 passwd，黑客继而重写 password文件，修改root账户的密码。黑客将会得到root权限。 

## 符合要求的方式 (Linux 2.1.126+, FreeBSD, Solaris 10, POSIX.1-2008 O_NOFOLLOW)

系统会提供 O_NOFOLLOW 标志来减少此问题，该标志在 POSIX.1-2008 标准中定义，并且被定义为可移植的。使用了此标志位时，如果 file_name 是个符号链接，open 将会失败。 

```c
char *file_name = /* something */; 
char *userbuf = /* something */; 
unsigned int userlen = /* length of userbuf string */;   

int fd = open(file_name, O_RDWR | O_NOFOLLOW); 
if (fd == -1) {
   /* handle error */
} 

write(fd, userbuf, userlen); 
```

特别提示：此方案不能检查硬链接。 

## 符合要求的方式 (lstat-fopen-fstat)

这种方式使用 FIO05-C 推荐规范中的 lstat-fopen-fstat 组合。 

```c
char *file_name = /* some value */; 
  
struct stat orig_st; 
if (lstat( file_name, &orig_st) != 0) { 
  /* handle error */ 
} 
  
if (!S_ISREG( orig_st.st_mode)) { 
  /* file is irregular or symlink */ 
} 
  
int fd = open(file_name, O_RDWR); 
if (fd == -1) { 
  /* handle error */ 
} 
  
struct stat new_st; 
if (fstat(fd, &new_st) != 0) { 
  /* handle error */ 
} 
  
if (orig_st.st_dev != new_st.st_dev || 
    orig_st.st_ino != new_st.st_ino) { 
  /* file was tampered with during race window */ 
} 
  
/* ... file is good, operate on fd ... */
```

这段代码仍会有 time-of-check, time-of-use（TOCTOU） 竞争风险，但在任何文件操作之前，都会验证已经打开的文件是和前面验证的文件是同一个文件（通过检查文件的device和i-node）。代码将会辨认在竞争窗口中文件是否被黑客篡改。 

特别提示：此方案不能检查硬链接。 

## 硬链接

如果一个文件有多个硬链接，特别是被恶意的黑客创建的，区分链接文件和原始文件将非常困难。简单处理办法是“不允许打开有2个以上硬链接的文件”。下面的代码片段插入到前面的代码中，将可以识别一个文件是否有多个硬链接。

```c
if (orig_st.st_nlink > 1) 
{
   /* file has multiple hard links */
} 
```

由于硬链接要求在同一磁盘中，很多平台将系统关键文件放置在user不能编辑的地方（磁盘、目录）。比如， / 根目录包含了一些关键系统文件，如 /etc/passwd，最好放在一个独立的硬盘上。 /home 目录放在另外一个硬盘或磁盘上。以阻止用户创建指向 /etc/passwd 的硬链接。 

>Kevin 总结：
用 open() 读写一个符号链接文件时，存在链接被误改或篡改到其他文件（如password）的风险，如果程序的权限过高，误改或篡改得逞。所以要：
1. 降低 open() 的权限
2. 使用 O_NOFOLLOW 参数
3. open前 lstat，open后 fstat