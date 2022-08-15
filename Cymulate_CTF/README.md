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
   (lets call the pointer to that address enc_data),now theres a call a function called `validate`  
   and if its return value is 0 than the validation succeeded, lets have a look at the vlidate function  
   
   <img src="./screenshots/validate.png" width = "900" height="900">  
   
   Again i've added comments and changed var names to make it easier to follow, basically  
   thats a pseudo code of the function
   ```
   def validate(bin_bin , enc_data , file_len , num):

          if file_len != num:
              return 1 
          
          key = "h7_709_u0Y" # mind the order (big-endian so need to reverse this)
          for i in range(file_len):
                if (enc_data[i] ^ key[0 till 9] != bin_bin[i] ):
                      return 1
              
   ```
   In order to solve it we will need export the memory dump from IDA (78648 bytes from  
   the starting address) into a file called _bbbdump_
   ```
   # snippet of the dump format 
   26 75 39 19 3b 31 36 5f 37 68 59 30 75 5f 39 30 
   34 5f 9 68 58 30 75 5f 99 21 37 5f 37 68 59 30
   35 5f 39 30 37 5f 37 68 21 1b 74 5f 39 30 37 5f
   37 68 59 30 35 5f 1 30 3a 5f 77 68 46 30 6b 5f
   3f 30 37 5f 33 68 59 30 35 5f 39 30 37 5f 37 68 
   19 30 75 5f 39 30 37 5f 77 68 59 30 75 5f 39 30 
   ef 5d 37 68 59 30 75 5f e1 32 37 5f 37 68 59 30 

   ```  
   __$ Reversing the Validation__

   ```python

   """
   The validate function check for ->
        bin_bin_byte[index] ^ key[key_index] = enc_data[index]

   since xor is reverseable than ->
        enc_data[index] ^ key[key_index] = bin_bin_byte[index]

  that way we can get the file that can pass the validate function
  """

  key_counter = 0 # 0 ---- > 9  
  decryption_key = [0x59 , 0x30, 0x75, 0x5F, 0x39, 0x30, 0x37, 0x5F, 0x37, 0x68]


  # Create bin_bin file
  with open('../mem_dumps/bbbdump') as f , open('bin_bin' , 'wb') as out:
    
    # loop through the encrypted data
    for line in f:
      
      # create an array for the correct hex values
        bytes_output = []
        for hex_byte in line.split():

          # check for key index reset       
          if key_counter == 10:
              key_counter = 0
          
          # xor values to get the valid one
          correct_byte = int(hex_byte,16) ^ decryption_key[key_counter]
          bytes_output.append(correct_byte)
          key_counter +=1
        
        out.write(bytes(bytes_output))
            
   ```
&nbsp;
* __bin_bin Analysis__  

  I've run `file` on it and got the same result as earlier, this is a 64-bit  
  not stripped ELF file as well  
  
  ```
  $ file ./bin_bin
  ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, 
  interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, not stripped
  ```  
  I've opened the file in IDA and its the same as _bin_bin_bin_ and the only  
  thing changed is the mem_dump bytes and the bytes of the key so i've modified  
  the values in the script to generate the next binary file called _"bin"_
  ```python
  key_counter = 0 # 0 ---- > 9  
  decryption_key = [0x33 , 0x5F, 0x50, 0x61, 0x35, 0x35, 0x77, 0x30, 0x52, 0x44]


  # Create bin_bin file
  with open('../mem_dumps/bbdump') as f , open('bin' , 'wb') as out:

    # loop through the encrypted data
    for line in f:
      
      # build correct hex values for each line grabbed
        byte_output = []
        for hex_byte in line.split():
          
          # check for reset on key_counter
          if key_counter == 10:
              key_counter = 0

          correct_byte = int(hex_byte,16) ^ decryption_key[key_counter]
          byte_output.append(correct_byte)
          key_counter +=1
        
        out.write(bytes(byte_output))
  ```  
  &nbsp;
* __bin Analysis__  
  
  its a 64-bit not stripped ELF file as well  
  
  ```
  $ file ./bin
  ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked,
  interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, not stripped
  ```  
  It follows the same anti debugging, file handling (insted of bin/bin_bin it  
  tries to open _"flag.zip"_) and  control flow (cmp that prevernt execution)  
  as the previous  bin_bin_bin && bin_bin, although it handles the validation diffrently  
  (added comments && changed var names to easier read)  
  <img src="./screenshots/bin_flow.png" width="700" height="400">  
  so this is the flow of this validation section



  
 










