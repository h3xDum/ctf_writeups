# Solution

  
#### ___The extracted files provided are___  
1. goldenticket.png
2. logo.png
3. img/ -> emptry folder 
4. admin.html

_Taking a look in the admin.html file we can see a javascript function  
called_ `checkCreds()` _which handles authentication_  
```js
function checkCreds() {
	if (username.value == "Admin" && atob(password.value) == "goldenticket") 
	{
		var key = atob(encoded_key);
		var flag = "";
		for (let i = 0; i < key.length; i++)
		{
			flag += String.fromCharCode(key.charCodeAt(i) ^ password.value.charCodeAt(i % password.value.length))
		}
		document.getElementById("banner").style.display = "none";
		document.getElementById("formdiv").style.display = "none";
		document.getElementById("message").style.display = "none";
		document.getElementById("final_flag").innerText = flag;
		document.getElementById("winner").style.display = "block";
	}
	else
	{
		document.getElementById("message").style.display = "block";
	}
}
```
&nbsp;

_The function check for a username value of:  __"Admin"__  with the  
base64 decoded password value of: __"goldenticket"__ , and preforms   
a smiple algorithem to decode the key and build it as the flag_ 

#### ___Algo pseudo code logic___
```
flag = " "
loop over encoded_key:
    flag = encoded_key[counter] XOR password[couter] 
    # moding (%) with pass length wont change counter value  
    # since counter is always smaller than the length inside the loop
```  
&nbsp;  

_All we need to do is base64 encode the string "goldenticket" as the password   
to authenticate ourself as Admin and we will get the flag_  
```bash
$ echo -n 'goldenticket' | base64  #dont forget -n or else echo will append "\n"
```
## flag
![flare_on1](https://github.com/h3xDum/ctf_writeups/blob/master/Flare_on/cred_checker/flag.png)


