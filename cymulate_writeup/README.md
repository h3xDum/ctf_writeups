# Solution

We are provided with the file *bin_bin_bin* , lets
run `file` to check it 

```
$ file ./bin_bin_bin
ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked,  
interpreter /lib64/ld-linux-x86-64.so.2,for GNU/Linux 3.2.0, not stripped
```
Its a 64-bit ELF file and its not stripped which means that  
we get function names, lets open it in IDA to check  
what the file does

![disass_main](./screenshots/main_1.png)

This is the start of our `main` function, immediately we see a call to  
a `antidebug` function and than it check if its return value is 0 to  
determine execution flow, lets check what the function does

![anti_debug](./screenshots/anti_debug.png)
  
  The function simply call ptrace with the _"traceme"_ value to check
  if the file is being debbuged, to work around it we can manually set  
`eax` value to 0 after it preforms the `ptrace` call, or to reverse the jump in `main`











