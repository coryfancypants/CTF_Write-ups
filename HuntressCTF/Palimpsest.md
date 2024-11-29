2024-11-28 10:59

Status: 

Tags: [[huntressctf]] [[solved]] [[malware]] [[obfuscation]] [[ctf]] [[Cyber Security]] [[cyber security awareness month]] [[Windows Event Logs]]

# Palimpsest
This challenge was incredibly fun, and there is a LOT of obfuscation, and just trying to figure out what this malware is even doing haha.
We are provided with some Windows Event logs.
We have
- Application.evtx
- Security.evtx
- System.evtx
So something about the logs here contains the flag, I won't look at that just yet as we were also given a file named Update Service.xml
Here is the important section of the xml file just due to the powershell command that is ran.
![HuntressCTF_Palimpsest_XMLFile](https://github.com/user-attachments/assets/3bc69ea2-a911-4f88-8a46-7b5d415f0979)

The powershell command reaches out to the web domain and views the TXT record for the [[DNS]] server. 
Honestly, I thought this was a REALLY neat way for some Malware to run code.
If we use [[MxToolbox]] to view the text record we can find this.
![HuntressCTF_Palimpsest_TXTRecord](https://github.com/user-attachments/assets/a0e59235-d341-431b-bd58-4f57f29a10f5)

And the base64 in the TXT record is this
```
LiggJFNIRWxsaURbMV0rJFNIRWxMaWRbMTNdKydYJykoIG5lVy1vQkpFQ1QgSW8uQ29tUHJlU3NJT04uZEVmbEFUZVN0UmVBbShbc3lTVGVtLklPLm1FTU9SWXNUUkVBTV1bc1lzdEVNLmNPbnZlUlRdOjpGUk9tQkFzRTY0c3RyaW5HKCAnWlZiTGF0eFFEUDBWTFZwbUp0d0o5MjE3bVQ1b0N5V0IwbDBJSmFWWkpKQVdzaWlCdHY5ZTZVaTY5clFMRzE5TDF1UG9TUEw1ZnYvdTd2UHg1KzNUL2NYWGoyOXBkL2I0N2Vsc2R6aS92SDI4dXk0aHBaQnZqZzlYOTk5M3U4TitmLzM2L2NYVDljME43U25sUkNIRmhRS2xPUE05eDlDbVFEWEtpOGl5M01MU1dTQlhUWHlGVkZpdFVoQ1ZOTEhHUktITVFiOHZtUzgrSkxaYnNnaGdIOC9RVEltZjJnTERLYk4yNjJ3bmk1Z042eHU1NER1TGkwSkRvY21uSkJwOG1xQ1FVb2U4U2NEWnpDYUpoRDMzaUpCY1lla2lSNjVOczViUVNHNTZTaERLUGNVSmVqV1lWTlhFQjdrVEpOZkx4Z1hCaDBValJ3M0Z2cHhENW5nVVh2RUFqSkZkbEJyQllaNFVQd2RUWEJDQ3cwR0JGWUdDQ1VlQ0tCbWtCTkFxeTVyR2xJRXJuWmpYYW51eFBFQXIwWDlHK2FFMnJWTkxva3NHL0ZxWVdkWElpZ0dIZUZFbDJ0WXBXQUc3Z3d3a1VxaHNRMHBOclVKUllSQmFaVWhBQnRiamNOaWtPT3d4aU5GV3lHdXBJdFI2STdFQ0JXRXFERGpZYklBNnFvSlRrdXp3RU1SY25pMW9NVmVNVDRKRlFaU0ZsSjZWOFpKQ21sL0xzeWgyakJyTHhVK2JrUy9MQmVVcXBHdU9ndHlVc095MUcweXdJeGxKMkxYb1M0RmRCSXdLQVpKNTFFamU4YXVHSWk0R3p4d1U4YVpJZ2lsS3FpVU1Tam1QUktDOXIrMHFvcWlaVDI1RHFXQmRKTFZubEtwenpkcVdBQXBqMC9Nb0JRME9haHNyTjFzWTdhNHNhY29GV2psQ3ErTlVsUWNWTUhNbVBUdWpORCtsTlhraWk3VU5VbHh6WXJwTHNxb3B3RWsrR2c5SzB5VjhwV1FQUmdDcG92UGFjbVNHMW1tMGxWdFlhTXQvQWFrNFBaZDVUSVl5TzJOdFBuRVh6RFlZbmNoSUxnWXJPRWdvR0NJc2IzZi9PbXRHcVdtRmM5VCtCTGVyY3dtblBDdUpxRVRGanNtdXRiRjRyTktrRVBGVEtSNFUraklxOURVWkdSSUdId3NhSUtHZWJUc2dRbytrVFpDV3lkNlg0Z3lLN21aMmNGWk1TbFFxcW1BWmJGQVZrRGViUXZMb3BSZlhUQmNGVHZPS2p1L3FRU0pmRFZpTGpCVzI5c1JDb3c5QW1NM095alpyMXNsV2ZOb0NxOEY2blljTms1OUJvVEh4Wk9BMWxXS1FKeHZGUlpzdjZTekI4a25sTkFNWlJyb0RFRDhXd1VTMlNSZWRaU1FqWmNGTUl0dTdZMCtDY2pIL0M2d0Rvd0dmN0hiYUxuZmJtcHQyMGhucE84aHpHc3ZNb1NUL2V2WU5OaUk0eGJjdVRycStkdGJZTnJJbHZOdld3UUpRdGwyNGdzNTJpazFPQTkyMlRLZkQ3NWYwaS9CREpMOURMNzdROGRYejFhZGRmRzV2ZG9jL2REZysvUGh3eVg5TmZ3RT0nICkgLFtJby5jb01QUkVzc0lvbi5DT01QcmVTU2lPTk1vREVdOjpkRWNvbXBSRVNzKXwgZm9yRWFjSC1vYkpFQ1Qge25lVy1vQkpFQ1Qgc3lzVEVtLklPLlNUUkVBTVJFYURlUiggJF8gLCBbVEV4VC5lbmNvRGlOR106OmFzQ2lpICkgfXwgZm9yZUFDSC1PQmpFQ1QgeyRfLlJFQWRUb0VuZCgpIH0gKQ==
```
Which decodes to this 
```
.( $SHElliD[1]+$SHElLid[13]+'X')( neW-oBJECT Io.ComPreSsION.dEflATeStReAm([sySTem.IO.mEMORYsTREAM][sYstEM.cOnveRT]::FROmBAsE64strinG( 'ZVbLatxQDP0VLVpmJtwJ9217mT5oCyWB0l0IJaVZJJAWsiiBtv9e6Ui69rQLG19L1uPoSPL5fv/u7vPx5+3T/cXXj29pd/b47elsdzi/vH28uy4hpZBvjg9X9993u8N+f/36/cXT9c0N7SnlRCHFhQKlOPM9x9CmQDXKi8iy3MLSWSBXTXyFVFitUhCVNLHGRKHMQb8vmS8+JLZbsghgH8/QTImf2gLDKbN262wni5gN6xu54DuLi0JDocmnJBp8mqCQUoe8ScDZzCaJhD33iJBcYekiR65Ns5bQSG56ShDKPcUJejWYVNXEB7kTJNfLxgXBh0UjRw3FvpxD5ngUXvEAjJFdlBrBYZ4UPwdTXBCCw0GBFYGCCUeCKBmkBNAqy5rGlIErnZjXanuxPEAr0X9G+aE2rVNLoksG/FqYWdXIigGHeFEl2tYpWAG7gwwkUqhsQ0pNrUJRYRBaZUhABtbjcNikOOwxiNFWyGupItR6I7ECBWEqDDjYbIA6qoJTkuzwEMRcni1oMVeMT4JFQZSFlJ6V8ZJCml/Lsyh2jBrLxU+bkS/LBeUqpGuOgtyUsOy1G0ywIxlJ2LXoS4FdBIwKAZJ51Eje8auGIi4GzxwU8aZIgilKqiUMSjmPRKC9r+0qoqiZT25DqWBdJLVnlKpzzdqWAApj0/MoBQ0OahsrN1sY7a4sacoFWjlCq+NUlQcVMHMmPTujND+lNXkii7UNUlxzYrpLsqopwEk+Gg9K0yV8pWQPRgCpovPacmSG1mm0lVtYaMt/Aak4PZd5TIYyO2NtPnEXzDYYnchILgYrOEgoGCIsb3f/OmtGqWmFc9T+BLercwmnPCuJqETFjsmutbF4rNKkEPFTKR4U+jIq9DUZGRIGHwsaIKGebTsgQo+kTZCWyd6X4gyK7mZ2cFZMSlQqqmAZbFAVkDebQvLopRfXTBcFTvOKju/qQSJfDViLjBW29sRCow9AmM3OyjZr1slWfNoCq8F6nYcNk59BoTHxZOA1lWKQJxvFRZsv6SzB8knlNAMZRroDED8WwUS2SRedZSQjZcFMItu7Y0+CcjH/C6wDowGf7HbaLnfbmpt20hnpO8hzGsvMoST/evYNNiI4xbcuTrq+dtbYNrIlvNvWwQJQtl24gs52ik1OA922TKfD75f0i/BDJL9DL77Q8dXz1addfG5vdoc/dDg+/PhwyX9NfwE=' ) ,[Io.coMPREssIon.COMPreSSiONMoDE]::dEcompRESs)| forEacH-obJECT {neW-oBJECT sysTEm.IO.STREAMREaDeR( $_ , [TExT.encoDiNG]::asCii ) }| foreACH-OBjECT {$_.REAdToEnd() } )  
```
And has another [[base64]] encoded message so let's see what that says.
Which decodes to this:
```
.((GeT-variAbLE '*mdr*').Name[3,11,2]-jOin'')(([CHAr[]] ( 121 ,109 , 108 , 20,57, 40, 100 ,125,96 , 6 , 41, 4,13, 24 ,0, 117,127 ,38,108 , 32, 38,111 ,32 ,38 ,109,32 ,127 ,112 ,59,125,122, 56, 122 ,113,122, 52, 50 ,122, 113 , 122 ,115 ,59 , 52 ,17,122,116 , 125, 102,125,121 , 38 ,60 , 32 , 125, 96,125 , 105,109 ,109, 109,109 ,115 , 115 ,107 , 104,109,109, 109, 102 ,125,121,38 ,63 , 32 , 125 , 96, 125, 125 ,121 , 109, 108,20 ,57, 40 ,100, 103 , 103,117 , 127, 38,108 , 32,38 , 109 ,32,38,111 , 32,127 ,125, 112 , 59,125, 122, 47 ,52 , 122,113, 117 , 127, 38 , 108, 32, 38 , 109, 32 ,127,125, 112 , 59,122 ,45, 56, 51, 10 ,122,113 , 122 ,18,122 , 116 , 113, 122 , 41 , 56 ,122 ,116 ,115 , 20 ,51,43 ,50 , 54 , 56,117 ,117,23 ,50, 52,51,112, 13 , 60,41 ,53 ,125 , 112, 13,60 ,41 ,53 ,125,121,38,24 ,51,11, 103 , 60, 61 , 13 , 61 ,45 , 61,25 ,28, 41 , 60 ,32,125,112 ,30 , 53 , 52, 49, 57, 13,60 , 41 , 53,125, 59,49,60, 58 ,115, 48 , 45,105,116 ,116 ,102,125 ,26 , 56 ,41 , 112,24 ,43, 56 , 51,41 ,17, 50, 58,125, 112,17,50,58 , 19 , 60,48,56 ,125 ,117,127 , 38,109,32,38 , 111 , 32, 38, 108 ,32 ,38, 110,32,127,125 ,112 , 59 , 125,122,28,45, 122, 113,122 , 49,52,62,60 ,41 , 52 , 122, 113 ,122, 45,122 ,113, 122 ,50 ,51, 122 ,116 , 125 ,112 ,14 ,50, 40 , 47 ,62, 56 ,125 ,117 , 127 , 38, 109, 32,38, 111,32 ,38, 108,32, 127 , 112,59 ,122, 48 , 46, 49 ,51,46,41 , 60,49,122 , 113,122 ,56,47, 122,113 ,122, 49 ,122 , 116 ,125,33 ,125 ,98 , 125 , 38,125 , 121 , 38, 28,32 ,125 , 112,62, 50,51,41 , 60 ,52 ,51,46 ,125, 121 , 38,2,32, 115,127 ,20, 51, 61 , 46 ,41 , 61 , 28 , 51, 30, 56 ,61, 52 , 25 , 127,125 , 32, 125 ,33,125 , 14 , 50, 47 ,41,112 , 18 ,63, 55 ,56, 62 , 41, 125, 20, 51, 57 ,56,37, 125, 33, 125,120, 125 ,38, 125 , 121 , 38, 30 ,32 , 125 ,96 , 125 , 121 ,38 , 2,32 , 115 , 127, 57,61 , 28 , 9, 60 , 127,102 ,125 ,121 , 38, 63, 32 , 115 ,117,127,38,108 ,32, 38, 109 ,32,127,112,59,125,122, 52 ,41 ,56 ,122 ,113 , 122,10, 47, 122, 116 , 115,20 , 51 ,43 ,50 ,54 ,56 , 117 ,121, 38 , 30,32,113,125,109,113,125 ,121 , 38 ,30, 32,115 ,127 ,17 , 56, 19 , 61, 26 ,9, 53, 127 ,116 , 125, 32 ,102 , 125 , 121 ,38, 63, 32, 115, 117,127 ,38, 108 , 32, 38 ,109, 32, 127 , 125,112, 59,125 , 117 , 127, 38, 109 ,32 , 38 ,108, 32,127, 125 , 112,59,125,122,49 , 50, 46 ,122 , 113 , 122 ,56,122 , 116, 113 ,122 ,30,122 , 116 ,115,20 , 51, 43, 50, 54 ,56 ,117 ,116 )|% { [CHAr] ( $_ -BxOR'0x5D')} )-joIN'')
```
This looks like some crazy obfuscation, but it is basically running a Bitwise  XOR on each character code with 0x5D
De-obfuscating this powershell command isn't too bad, definitely looks scarier. I did this in Python, 
```
char_codes = [
    121 ,109 , 108 , 20,57, 40, 100 ,125,96 , 6 , 41, 4,13, 24 ,0, 117,127 ,38,108 , 32, 38,111 ,32 ,38 ,109,
    32 ,127 ,112 ,59,125,122, 56, 122 ,113,122, 52, 50 ,122, 113 , 122 ,115 ,59 , 52 ,17,122,116 , 125, 102,125,121 ,
    38 ,60 , 32 , 125, 96,125 , 105,109 ,109, 109,109 ,115 , 115 ,107 , 104,109,109, 109, 102 ,125,121,38 ,63 , 32 , 125 ,
    96, 125, 125 ,121 , 109, 108,20 ,57, 40 ,100, 103 , 103,117 , 127, 38,108 , 32,38 , 109 ,32,38,111 , 32,127 ,125,
    112 , 59,125, 122, 47 ,52 , 122,113, 117 , 127, 38 , 108, 32, 38 , 109, 32 ,127,125, 112 , 59,122 ,45, 56, 51, 10 ,
    122,113 , 122 ,18,122 , 116 , 113, 122 , 41 , 56 ,122 ,116 ,115 , 20 ,51,43 ,50 , 54 , 56,117 ,117,23 ,50, 52,51,
    112, 13 , 60,41 ,53 ,125 , 112, 13,60 ,41 ,53 ,125,121,38,24 ,51,11, 103 , 60, 61 ,
    13 , 61 ,45 , 61,25 ,28, 41 , 60 ,32,125,112 ,30 , 53 , 52, 49, 57, 13,60 , 41 , 53,125, 59,49,60, 58 ,
    115, 48 , 45,105,116 ,116 ,102,125 ,26 , 56 ,41 , 112,24 ,43, 56 , 51,41 ,17, 50, 58,125, 112,17,50,58 ,
    19 , 60,48,56 ,125 ,117,127 , 38,109,32,38 , 111 , 32, 38, 108 ,32 ,38, 110,32,127,125 ,112 , 59 , 125,122,
    28,45, 122, 113,122 , 49,52,62,60 ,41 , 52 , 122, 113 ,122, 45,122 ,113, 122 ,50 ,51, 122 ,116 , 125 ,112 ,14 ,
    50, 40 , 47 ,62, 56 ,125 ,117 , 127 , 38, 109, 32,38, 111,32 ,38, 108,32, 127 , 112,59 ,122, 48 , 46, 49 ,51,
    46,41 , 60,49,122 , 113,122 ,56,47, 122,113 ,122, 49 ,122 , 116 ,125,33 ,125 ,98 , 125 , 38,125 , 121 , 38, 28,
    32 ,125 , 112,62, 50,51,41 , 60 ,52 ,51,46 ,125, 121 , 38,2,32, 115,127 ,20, 51, 61 , 46 ,41 , 61 , 28 ,
    51, 30, 56 ,61, 52 , 25 , 127,125 , 32, 125 ,33,125 , 14 , 50, 47 ,41,112 , 18 ,63, 55 ,56, 62 , 41, 125, 20,
    51, 57 ,56,37, 125, 33, 125,120, 125 ,38, 125 , 121 , 38, 30 ,32 , 125 ,96 , 125 , 121 ,38 , 2,32 , 115 , 127, 57,
    61 , 28 , 9, 60 , 127,102 ,125 ,121 , 38, 63, 32 , 115 ,117,127,38,108 ,32, 38, 109 ,32,127,112,59,125,122,
    52 ,41 ,56 ,122 ,113 , 122,10, 47, 122, 116 , 115,20 , 51 ,43 ,50 ,54 ,56 , 117 ,121, 38 , 30,32,113,125,109,
    113,125 ,121 , 38 ,30, 32,115 ,127 ,17 , 56, 19 , 61, 26 ,9, 53, 127 ,116 , 125, 32 ,102 , 125 , 121 ,38, 63, 32,
    115, 117,127 ,38, 108 , 32, 38 ,109, 32, 127 , 125,112, 59,125 , 117 , 127, 38, 109 ,32 , 38 ,108, 32,127, 125 , 112,
    59,125,122,49 , 50, 46 ,122 , 113 , 122 ,56,122 , 116, 113 ,122 ,30,122 , 116 ,115,20 , 51, 43, 50, 54 ,56 ,117 ,116
]

decoded = ''.join(chr(code ^ 0x5D) for code in char_codes)
print(decoded)
```
After running the code I created, the next command looks like this, which finally ties in the Windows Event logs, which is quite cool! There is still some more obfuscation in this script.
This will take some time to process through, it's not overly complicated, just messy.
```
$01Idu9 =[tYPE]("{1}{2}{0}"-f 'e','io','.fiL') ; ${a} = 40000..65000; ${b} =  $01Idu9::("{1}{0}{2}" -f 'ri',("{1}{0}" -f'penW','O'),'te').Invoke((Join-Path -Path ${EnV:a`P`p`DAta} -ChildPath flag.mp4)); Get-EventLog -LogName ("{0}{2}{1}{3}" -f 'Ap','licati','p','on') -Source ("{0}{2}{1}"-f'mslnstal','er','l') | ? { ${A} -contains ${_}."In`st`AnCe`iD" } | Sort-Object Index | % { ${C} = ${_}."d`ATa"; ${b}.("{1}{0}"-f 'ite','Wr').Invoke(${C}, 0, ${C}."LeN`GTh") }; ${b}.("{1}{0}" -f ("{0}{1}" -f 'los','e'),'C').Invoke()
```
And to make this not as messy and look much prettier.

```
$01Idu9 = [System.IO.File]
${a} = 40000..65000
${b} = [System.IO.File]::OpenWrite((Join-Path -Path ${Env:APPDATA} -ChildPath flag.mp4))
Get-EventLog -LogName "Application" -Source "mslnstaller"
| Where-Object { ${a} -contains $_.InstanceID } | Sort-Object Index | ForEach-Object {
    ${C} = $_.Data
    ${b}.Write(${C}, 0, ${C}.Length)
}
${b}.Close()
```
Okay so in Obsidian I have a nerd font installed and it's much easier to see than when I was working on this inside the VM and working off notepad haha.
So lets go over this script, because it's the final stage of the malware.
We are searching through the Application Event logs for a Source of mslnstaller, the normal event log is msinstaller. The i was replaced with an l. really subtle, easier to see on nerd fonts.
Anyways, We are looking through the application logs for event IDs in the range of 40000-65000, then we Sort the entries, grab the DATA field, and write that Data to %APPDATA%\flag.mp4

Well now that we know what the code does, lets go look at those Events in the Application logs.
![HuntressCTF_palimpsest_applicationLogs](https://github.com/user-attachments/assets/31eb7607-3e0e-4078-afa7-2d15fff1ca46)

Okay so, we care about the Event Data, and yes I did get distracted and read through all of the jokes that were written into the Event Data field, because they were funny, and it was super entertaining, I should compile all the jokes into a file of it's own. 
Back on track, the HEX string is probably what the powershell script is grabbing. 
So if this has clicked, we are basically grabbing Hex Data out of the Windows Event Logs, and compiling all of that data into 1 file called flag.mp4. 
This last challenge is building a video with data that is in Event Viewer. This was such an interesting and cool concept.

So from this point on, we just have to figure out how to build the mp4 file with the Application Event logs we got, one issue we have is the Get-EventLog command in the script is a local command and cannot be ran on a .evtx file. So I played with Get-WinEvent, but I could not for the life of me figure out how to make get-winevent work. Adam was talking about this after the CTF, and he also had trouble getting this command working, there was a guy in the [[Discord]] that did get it to work, and his solution is really cool. https://cyber-note.notion.site/2024-10-31-Palimpsest-Malware-1296c6e16f8680a3b2fcfcc0d61e0aed 
Props to Discord user Mishka.
I took a very simple approach, and replaced my devices event logs with the provided ones, and then ran the obfuscated powershell. 
This creates our flag.
![HuntressCTF_Palimpsest_flag](https://github.com/user-attachments/assets/b6a8eb1d-5152-4aba-a4df-872beabd7e5c)


Thank you Huntress for a very fun month, and some great challenges throughout. I did place much better than I thought I would for this being my first CTF. This was a great final challenge.

