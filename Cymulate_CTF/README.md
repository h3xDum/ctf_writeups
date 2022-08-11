# Solution

We are provided with the file *bin_bin_bin* , lets
run `file` to check it 

```
$ file ./bin_bin_bin
ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked,  
interpreter /lib64/ld-linux-x86-64.so.2,for GNU/Linux 3.2.0, not stripped
```
Its a 64-bit ELF file and its not stripped which means that  
we get function names, lets open it in IDA to check  what the file does.
&nbsp;
&nbsp;


* __Initial Static Analysis__  

  __$ Anti-debuggin__
   
   <img src="./screenshots/main_1.png" width="560" height="400">

    This is the start of our `main` function, immediately we see a call to a  `antidebug`  
    function and than a check if its return value is 0 to determine execution flow, lets check  
    what the function does

   <img src="./screenshots/anti_debug.png" width="560" height="400">
  
   The function simply call ptrace with the _"traceme"_ value to check if the file is being  
   debbuged, to work around it we can manually set `eax` value to 0 after it preforms the `ptrace`  
   call, or to reverse the _jz_ --> _jzn_  
   
    __$ File handling__  
   
   <img src="./screenshots/file_handle.png" width="700" height="400">  

   The binary open a file called bin_bin that needs to be in the current directory and  
   than calls
   *  fseek -> set the pointer to the end of the file (2/SEEK_END)
   *  ftell  -> get the number of bytes from start to pointer (end of file) and pass it to var_13350
   *  fseek -> set the pointer to the start of the file (0/SEEK_SET)  
   also `cmp var_13354 , 0` will always result in true which will go to a path that  
   prints a message and exit, thats a check that always prevents execution so jmp must be reversed  

   
   __$ Validation__  
   <img src="./screenshots/main_2.png" width="700" height="400">  
   I've modified some var names and and added comments to make the assembly easier  
   to read, there's a call to _memcpy_ to copy 78648 bytes from the data segment to 
   a diffrent location  
   (lets call the pointer to that address file_ptr), basically  
   duplicating the array so the values can be modifed without changed the original file.  
   Now thres a call a function called `validate` and if its return value is 0 than the  
   validation succeeded
   









