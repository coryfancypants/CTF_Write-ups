2024-10-14 20:30

Status: #completed_huntress_CTF

Tags: [[solved]] [[huntressctf]] [[ctf]] [[dnSpy]] [[Malware]] [[cyber security awareness month]] [[dll]]

# xray ctf
xray is an interesting malware challenge, the file we get was a quarantined malware file, there was a video by [[John Hammond]] that covers this really well [[Recover Quarantined Malware Video]] 
Using a software called DeXray we can recover the file.
![[HuntressCTF_xRay_dexray.png]]
We can check the file type of this recovered file, and see that it is a [[DLL]]
so I'm going to move the recovered file over to flare_vm, because lets open it in [[DnSpy]]

![[HuntressCTF_xray_dnspy.png]]
Loading up the [[dll]] into [[dnSpy]] and I went down to where the main function is, we can see this main function is probably our flag, and it uses 3 arrays, and calls 2 more functions. 
So I are going to look at the .load and .otp function 
Here is what those functions look like.
![[HuntressCTF_xray_loadandotpfunctions.png]]

So from here I just translated all the code that makes sense to python code, since that is what I am familiar with. 
```
def load(hex_string):
    length = len(hex_string)
    array = bytearray(length // 2)
    for i in range(0, length, 2):
        array[i // 2] = int(hex_string[i:i+2], 16)
    return bytes(array)


def otp(data, key):
    array = bytearray(len(data))
    for i in range(len(data)):
        array[i] = data[i] ^ key[i % len(key)]
    return bytes(array)


def main():
    array1 = load('15b279d8c0fdbd7d4a8eea255876a0fd189f4fafd4f4124dafae47cb20a447308e3f77995d3c')
    array2 = load('73de18bfbb99db4f7cbed3156d40959e7aac7d96b29071759c9b70fb18947000be5d41ab6c41')
    array3 = otp(array1, array2)
    print(array3)


if __name__ == "__main__":
    main()
```
This approach made sense to me, so I could have work on each function at a time. 
But this should be all we need, so let's run the code and get our flag which is.
```
flag{df26090565cb329fdc8357080700b621}
```

# References
[[Recover Quarantined Malware Video]]