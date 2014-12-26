原文链接： https://www.securecoding.cert.org/confluence/display/seccode/POS02-C.+Follow+the+principle+of+least+privilege

# 遵循最小特权原则 

最小特权原则意指所有程序、所有用户，都应该使用完成工作所需要的最小权限来操作。搭建高安全性网站[DHS 2006]标准还附加了另外的内容。使用最小特权可以减轻问题代码出故障的几率。 

## 不符合要求的代码示例

某些程序不需要特殊权限的程序经常会在特殊权限下运行。比如，网络程序需要 superuser 权限去捕获网络原始包数据，但在分析包数据的时候并不需要同样高的权限。较好的设计策略是：“根据程序所需降低或提供权限”。 

请慎重考虑哪些绑定到公共端口（如1024）的用户服务。为了防止黑客发起的恶意链接，内核要求只有 superuser 才能调用 bind()系统调用。 

下面不规范的代码由setuid-superuser运行.调用 bind() 后又 fork 出子进程去完成记录任务。结果就是 bind() 后程序仍会以 superuser 权限继续执行。 

```c
int establish(void) { 
  struct sockaddr_in sa; /* listening socket's address */ 
  int s; /* listening socket */ 
  
  /*  Fill up the structure with address and port number  */ 
  
  sa.sin_port = htons(portnum); 
  
  /*  Other system calls like socket()  */ 
  
  if (bind(s, (struct sockaddr *)&sa, 
        sizeof(struct sockaddr_in)) < 0) { 
    /* Perform cleanup */ 
  } 
  
  /* Return */ 
} 
  
int main(void) { 
   int s = establish(); 
  
  /*  Block with accept() until a client connects  */ 
  
   switch (fork()) { 
      case -1 :  /* Error, clean up and quit */ 
      case  0 :  /* This is the child, handle the client */ 
      default :  /* This is the parent, continue blocking */
   } 
}
```

一个黑客可以随意执行的程序中一旦出现了这样的漏洞，那他肯定要提高权限来玩玩。

## 符合要求的方法

程序必须遵守最小权限原则，小心的分离 binding 和 bookkeeping 两个任务。 使用 superuser-level（superuser同级别）账户来降低程序泄漏问题的机会是很有必要的，它在完成操作后将会释放掉 superuser 权限。下面所示代码在 bind() 执行后立即释放权限，不仅能保证高级权限不会被再次获得，也遵守了规范： POS37-C 保证成功放弃权限 

```c
/*  Code with elevated privileges  */ 
  
int establish(void) { 
  struct sockaddr_in sa; /* listening socket's address */ 
  int s; /* listening socket */ 
  
  /* Fill up the structure with address and port number */ 
  
  sa.sin_port = htons(portnum); 
  
  /* Other system calls like socket() */ 
  
  if (bind(s, (struct sockaddr *)&sa, 
        sizeof(struct sockaddr_in)) < 0) { 
    /* Perform cleanup */ 
  } 
  
  /* Return */ 
} 
  
int main(void) { 
  int s = establish(); 
  
  /* Drop privileges permanently */ 
  if (setuid(getuid()) == -1) { 
     /*  Handle the error  */ 
  } 
  
  if (setuid(0) != -1) { 
    /* Privileges can be restored, handle error */ 
  } 
  
  /* Block with accept() until a client connects */ 
  
  switch (fork()) { 
     case -1: /* Error, clean up and quit */ 
     case  0: /* Close all open file descriptors 
               * This is the child, handle the client 
               */ 
     default: /* This is the parent, continue blocking */
  } 
}
```

>Kevin 总结：

>给权限粒度要小，需要高权限的事情做完了，立马用 setuid() 降低权限