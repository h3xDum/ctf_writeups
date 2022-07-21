# Solution


___We are provied with an executable file called "hidden" , lets  
run  `file` to check it___

```
$ file ./hidden
ELF 64-bit LSB shared object, x86-64,version 1 (SYSV), dynamically linked,
interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=f72fd726040a4c89ddd763a1ff4c1041ef323f12, 
for GNU/Linux 3.2.0, not stripped
```
&nbsp;  
___Its a 64-bit not stripped ELF file which means we have  
Symbols to work with, lets run the binary___  
```
$ ./hidden
testpass123
fail!
```
___All we get is an input prompt (which we filled with "testpass123")  
and a "fail" message, which is not that useful but "fail" can be used  
as a Xref to traverse execution path if necessary and the string is not being built at runtime.  
Lets open the binary in IDA to check whats going on___  

![disass_main](./screenshots/hidden_main.png)