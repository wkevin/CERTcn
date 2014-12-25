[SEI](http://www.sei.cmu.edu/) (Software Engineering Institute,软件工程研究所)是美国国防部出资，在卡内基梅隆大学内创建的一个研究所。
SEI 官网中介绍其技术部门有3个：

1. CERT Division
2. Software Solutions Division
3. Emerging Technology Center

[CERT](https://www.cert.org) (Computer Emergency Response Team,计算机安全应急响应组)  位列第一，是1988年成立的，专门处理计算机网络安全问题。
CERT 的官方背景使其和 DHS（ Department of Homeland Security ，美国国土安全部）关系紧密。《Homeland》是我最喜欢的美剧之一，可惜演的是CIA，不是DHS。

中国也有个CERT：[CDCERT/CC](http://www.cert.org.cn/) （国家互联网应急中心），2002年成立的，网站上介绍是个非政府组织，貌似和 SEI/CERT 没啥关系，哦，除了竞争关系。

CERT 的主要任务当然是保卫美国的网络安全了，但也有一些副业，比如做个认证啥的，还有把它的成果公开一些赢得掌声和点赞 —— 其中 CERT Coding Standards （CERT 编码规范）就是其作品之一。

[CERT Coding Standards ](https://www.securecoding.cert.org/confluence/display/seccode/CERT+Coding+Standards) 包括了几门语言的编码规范：

1. [The CERT C Secure Coding Standard](https://www.securecoding.cert.org/confluence/display/seccode/CERT+C+Coding+Standard)
2. [The CERT C++ Secure Coding Standard](https://www.securecoding.cert.org/confluence/pages/viewpage.action?pageId=637)
3. [The CERT Oracle Secure Coding Standard for Java](https://www.securecoding.cert.org/confluence/display/java/The+CERT+Oracle+Coding+Standard+for+Java)
4. [The CERT Perl Secure Coding Standard](https://www.securecoding.cert.org/confluence/display/perl/CERT+Perl+Secure+Coding+Standard)

第1个已经存在了n多年，规则全面，后3个还都是一副弱不禁风的样儿。
所有的编程规范都在其网站上公开，可以看到CERT不断的修改和努力。

《The CERT C Secure Coding Standard》 虽然网站公开，但CERT还是为它出了印刷版，国内同仁还翻译了第一版：

|2014年第二版|2008年第一版|第一版中文翻译版<br>《C安全编码标准》|
|---|---|---|
|![](http://ecx.images-amazon.com/images/I/51ghwXvmjAL.jpg)|![](http://img3.douban.com/mpic/s4168060.jpg)|![](http://a3.att.hudong.com/54/02/01200000023844134376028348886_s.jpg)|

[《The CERT C Secure Coding Standard》](https://www.securecoding.cert.org/confluence/display/seccode/CERT+C+Coding+Standard) 试图从各个角度描述 C 编码过程中必须遵守和建议遵守的规范、规则，当前包括 17 个大类：


1. `01. Preprocessor (PRE) `
1. `02. Declarations and Initialization (DCL) `
1. `03. Expressions (EXP) `
1. `04. Integers (INT) `
1. `05. Floating Point (FLP) `
1. `06. Arrays (ARR) `
1. `07. Characters and Strings (STR) `
1. `08. Memory Management (MEM) `
1. `09. Input Output (FIO) `
1. `10. Environment (ENV) `
1. `11. Signals (SIG) `
1. `12. Error Handling (ERR) `
1. `13. Application Programming Interfaces (API) `
1. `14. Concurrency (CON) `
1. `48. Miscellaneous (MSC) `
1. `50. POSIX (POS) `
1. `51. Microsoft Windows (WIN) `


那么问题来了，本项目想干啥？

1. 零星的翻译一些 《The CERT C Secure Coding Standard》 的文章 —— 觉得有必要或好玩，就翻译一下，不奢望全部翻译，不做承诺，呵呵。
2. 等待志同道合的一起来进步