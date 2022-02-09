---
tags: Security of systems and networks
---
:::success
# SSN Lab 3 - Asymmetrical encryption
Name: Ivan Okhotnikov
:::


## Tasks
:::warning
> 1. Create a 2048 bit RSA key pair using openssl. Write your full name in a text file and encrypt it with your private key. Using OpenSSL extract the public modulus and the exponent from the public key. Publish your public key and the base64 formatted encrypted text in your report. Verify that it is enough for decryption.
>>Hint: study openssl params and encryption vs verification vs decryption vs signing

> 2. Assuming that you are generating a 1024 bit RSA key and the prime factors
have a 512 bit length, what is the probability of picking the same prime factor twice?
Explain your answer.
>>Hint: How many primes with length 512 bit or less exist

>3. Here you can find the modulus (public information) of two related 1024bit
RSA keys. Your keys are numbered using the list. Your task is to factor them i.e. retrieve p and q. You may use any tools for this. Explain your approach.
Hints: study the RSA algorithm. What private information can two keys share? What practical attacks exist? You may have to write code or use existing code for simple arithmetic operations. Be careful with bigInt values!

>4. Now that you have the p and q for both keys, recreate the first public and
private key using this script. Encrypt your name with the private key and post the public key and the base64 formatted encrypted data in your report.

> 5. Using GPG implement a small web of trust in your group of 4 people:<center>
    
![](https://i.imgur.com/SLVkub6.png)
</center>

>>Stage 1: create your PGP keys and exchange and sign each other's key as in the picture above (as you can see on the picture, at this stage the 1st participant exchanges with 2nd and 4th, but not with 3rd). After exchange, verify that you can communicate securely with that person.

>>Stage 2: obtain a public key of a person that you have not contacted yet and send them a private message alongside your public key. At this point you do not need to sign each other's keys, since your keys are already signed by 2 other trusted members, so you should be able to securely communicate without it.
Describe what you have done to implement this model.
(If your group size is < 4 - please some person from another group to also
participate in your group)
:::

## Implementation
:::info
> 1. Create a 2048 bit RSA key pair using openssl. Write your full name in a text file and encrypt it with your private key.


To generate the key, I used the `openssl genrsa` utility
I also wanted to additionally secure my private key using the `-des3` flag
(figure 1), but in this case the program will ask me to additionally enter a password
```
openssl genrsa -des3 -out private.pem 2048
```
<center>
    
![](https://i.imgur.com/0Oz4Waz.png)
Figure 1: Private key generation
</center>

I can also create a public key based on a private one using openssl (figure 2)
To confirm the operation, the program will also require a password
```
openssl rsa -in private.pem -outform PEM -pubout -out public.pem
```
<center>
    
![](https://i.imgur.com/lBHc1SQ.png)
Figure 2: Obtaining a public key based on a private one
</center>

<center>

![](https://i.imgur.com/snqo00T.png)
Figure 3: Information about the public key 
</center>

To encrypt and decrypt data based on private (in case of encryption) and public (in case of decryption) keys, there is a utility `openssl rsautl`
Using the `-sign` or `-check` flag, you can specify the operating mode (figure 4,5).
Since a private key is used in the case of encryption, it will be necessary to specify a password for it (figure 4)

```
openssl rsautl -sign -inkey private.pem -in name.txt -out name.txt.encrypted

openssl rsautl -verify -in name.txt.encrypted -inkey public.pem -pubin
```

<center>
    
![](https://i.imgur.com/DkaVybl.png)
Figure 4: Encrypting text from a file
    
![](https://i.imgur.com/RXrSAnL.png)
Figure 5: Decryption of the text from the file
</center>

---

> Using OpenSSL extract the public modulus and the exponent from the public key. 

To extract information about the public key, I will use the `openssl rsa` utility with the `-text` flags to output information and `-noout` so that the original key is not output
```
openssl rsa -pubin -inform PEM -text -noout < public.pem 
```

<center>

![](https://i.imgur.com/AgAJaXi.png)
Figure 6: Information about public key
</center>


> Publish your public key and the base64 formatted encrypted text in your report. Verify that it is enough for decryption.

```
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAz/Vv1tQc2okDKzQuNQT8
Q84tZ0xfyCAzWoM7dhUkIG+w+u9ol1BepoJjI8HTdxCouNab5XoJTbRJvZ6IPHOm
th3WYk+s85D+z+IgqQpJjdOYfuIe6/dtw8y6TwC7G5iCpgi4EqYLiN2zUOThRRHf
jYFBhi6TTduUZExZTkEkRsbmlsQsWQv9BLmNkTAma7t3aqRzinrImtaqMT+MpyWO
kNJMMhr/QeAvIvUAZKyfyFttzWVBbdet96cjKuhRxfuIEmeahNMUEDndrZCd+n5d
5zpuHEn4PUT9E9WBwkcWe5s7vy5uMilLAUb/U8gB2pC+pEUIStoi6tk0L7zyYCCF
iwIDAQAB
-----END PUBLIC KEY-----
```

To convert a message to base64, there is an `openssl base64` utility
(figure 7)

```
openssl base64 -in name.txt.encrypted
```

<center>
    
![](https://i.imgur.com/pom47Vm.png)
Figure 7: Receiving an encrypted message in base64 format
</center>

```
xkwFgtueLziIJMZLR8ka4rWjEefpLwetPLlfWvardAiq4Fb8s9c2u3TkPSxxci2n
kw3a2dgChS379//gKo/U2VPIZnT34jkWgxEodiTQWqLHzL9/eW1k9uHcUFhVkgFl
PMmgGENTQbnEdskI3QQ3XmR4v3zb7QB2I0292oqUB4t/s2A+nZ3WYBgros7W33ah
TZEvYUYSofXOaOSFy9XIjpwJ+/eKPXWn32Vt7uBkJv0tNNid5wkjQ5NYmqdltgso
Lq30SGdF8xt548eBh469S5hQWauuJHdturK1akNGwVlA1OtY/6cGOAw10ArkCiBm
gymdW+T7ljGFNcToh5ausQ==
```

I can check the data on any decryptor site by specifying the latter in base64 and specifying the public key (Figure 8)

<center>
    
![](https://i.imgur.com/4iqjskv.png)
Figure 8: Window on [site](https://www.devglan.com/online-tools/rsa-encryption-decryption) to decrypt the message in base64
</center>
:::

:::info
> 2. Assuming that you are generating a 1024 bit RSA key and the prime factors
have a 512 bit length, what is the probability of picking the same prime factor twice?
Explain your answer.
>>Hint: How many primes with length 512 bit or less exist

According to the prime numbers theorem, you can calculate their number using the formula
```
P(N) ~ N/log(N)
```
where N is the maximum prime number

Since we have a maximum length of 512, then the minimum minimum number will be `2^512`, the maximum number will be `2^513 - 1`

Then following the formula, the number of primes will be equal to `(2^513-1)/log(2^513-1)`
Then the probability that the same number will be chosen twice is equal to
```
1/((2^513-1)/log(2^513-1)) ~ 1,326 * 10^-152
```
:::


:::info
> 3. Here you can find the modulus (public information) of two related 1024bit RSA keys. Your keys are numbered using the list. Your task is to factor them i.e. retrieve p and q. You may use any tools for this. Explain your approach.
Hints: study the RSA algorithm. What private information can two keys share? What practical attacks exist? You may have to write code or use existing code for simple arithmetic operations. Be careful with bigInt values!

The module is the product of `p` and `q`
since my two modules are related, it follows that they have the same parameter `p` or `q`
In order to find this common parameter, I decided:
1 - convert both modules to decimal
2 - find their common multiplier (since this number will denote the very first parameter you are looking for)
3 - to find the second parameter, you will simply need to divide each module by the number from point 2

By converting the numbers to decimal format , I got the values:

1. `109235082831614248816073673668109874309960169076128842377093235910270905201907465216502802847302291130807172107624151309634769709766257772648091327841853746867785138360682998314862835468784278127385048506022219919164048326578695512246126116423079274202237364378621676455186636312755257842906325092047927362647`
2. `132499329687283072172239283880698544668660350390560495551992120214258242174679601536894051815088782681231052121115921785882117651634287038451522098531061905117129917905051698843233175358966506038588933150873496933571263946007226056757749235967568848263362211868330761930897508018090576201385648095183376429751`

Their common multiplier -`9960310372300775551236476561936856174647425153608776080196469930540289766820919518171498753949631248187148732399901465777883374212142760225683574591715637
`

The second number is found by dividing the total multiplier by these numbers:
- 10967036040904171184339517092344602323473306426218178115497405283484298570966374531975848797865576955844290005350111165320280034023975375223464728431936731
- 13302731012856627869637711271003270505013558149634507519274218325667024747796646376892041102988963627116327796048120869116681762008657326630198111535867323


:::


:::info
> 4. Now that you have the p and q for both keys, recreate the first public and
private key using this script. Encrypt your name with the private key and post the
public key and the base64 formatted encrypted data in your report.


I changed the program a bit because it didn't want to encrypt my text

I added support for `gmpy2` `PKSC1_OAEP`
And I also made the text encrypted with `PKDC1_OAEP`

```
publickey = key.publickey()

encryptor = PKCS1_OAEP.new(publickey)
data = encryptor.encrypt(string_to_be_encrypted)


print (key.exportKey('PEM'))
print("\n")
print (key.publickey().exportKey())
print (base64.b64encode(data))
```

I also specified my values for `p`, `q` and `n` parameters
```
q = 9960310372300775551236476561936856174647425153608776080196469930540289766820919518171498753949631248187148732399901465777883374212142760225683574591715637
p = 10967036040904171184339517092344602323473306426218178115497405283484298570966374531975848797865576955844290005350111165320280034023975375223464728431936731
n = q*p
```

<center>
    
![](https://i.imgur.com/uDCmvyR.png)
Figure 9: Program output
</center>

In the output of the program I get the keys:

```
-----BEGIN RSA PRIVATE KEY-----
MIICXQIBAAKBgQCbjlDd+dB9eFGd5Dfs1Aj00C+ivT1WoPdXq90MYklWD3T5ZiTp
qRRGnaLdxQM56/Fef/I3UYcn30rCFf9TRmL4d4o2Jutfdn9NPDtrPoVvY2Gf7mMG
gqlYP0M3pBZvyWyA3/TeETMXwypIJOwscmIXJyrRbxrqifIeCanAPyvEVwIDAQAB
AoGAKga3HmGhzGQ3WLsRyPA4QzwDwqnx6neum4cZP4FGYvPmHINWMbK2gaWRHO1f
Q8TU/zz+CagDJeiT3//lbXA0yJQ/Lf6wXkdkHuq3Ou0cY64BeNXIDl9UUpaTuWQ9
oT6tQGYnxVuHg7S2HEuPs3efiVyqOHvnDInFWXeddlCNb9ECQQDRZcFp4ZwHPoEE
0iS1hXIWhAakLw/AV/SgaB7KR2oShThoUCTRuqK4GOs3J34SKrrfIXH/j6odlBrA
srfr38TbAkEAviz7oxDOG7xaw5GPziZZvcw8kItn60/ULjERKNLJ99B7KsBYnkal
CrHOuI+/ph9YZ1SgYYatKtfv1snHYM35NQJBAMaM087QDCCaVb/6erBcLofG/H0l
2qupOt32nGt1N9ED3S6b/62WaMBjcHVFzzbuqW71yaBn2whc7NkXHWpdLc0CQQCW
h/6q5WtvstjZQofksqCIRniOJXqdXTPjWD1v5eGuQZysi1HZ/qs22uV5W3dkpB0S
tX65k6PQbNpQVql1q7QdAkBDYNnWTt7KWxTOok6ftV/pJP2EitMTGrcQjF0ChpFn
GdAnDRgZe3D2d/nrrkVocbCniyIv0ddLcZAMbFGWE/hc
-----END RSA PRIVATE KEY-----
```

```
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCbjlDd+dB9eFGd5Dfs1Aj00C+i
vT1WoPdXq90MYklWD3T5ZiTpqRRGnaLdxQM56/Fef/I3UYcn30rCFf9TRmL4d4o2
Jutfdn9NPDtrPoVvY2Gf7mMGgqlYP0M3pBZvyWyA3/TeETMXwypIJOwscmIXJyrR
bxrqifIeCanAPyvEVwIDAQAB
-----END PUBLIC KEY-----
```


And encryption text: 
```
CtIB1lFkmxP4vmdnhNrW8Uxp6iCEhbtZ1xrz1QLE+kxcBo3VG6yN4C1asn6fSrX5asZgmUCYoHhpe9/bXbhw87Ur8VNX1ZM6cWosauAVx3MvseLaIkJjM72tdVVbOuAPe1cK27CgIVKcWTAyF6KzzHKP2a6gtCDRKJTFmw9VA2s=
```
:::


:::info
> 5. Using GPG implement a small web of trust in your group of 4 people:<center>
    
![](https://i.imgur.com/SLVkub6.png)
</center>

>>Stage 1: create your PGP keys and exchange and sign each other's key as in the picture above (as you can see on the picture, at this stage the 1st participant exchanges with 2nd and 4th, but not with 3rd). After exchange, verify that you can communicate securely with that person.

>>Stage 2: obtain a public key of a person that you have not contacted yet and send them a private message alongside your public key. At this point you do not need to sign each other's keys, since your keys are already signed by 2 other trusted members, so you should be able to securely communicate without it.
Describe what you have done to implement this model.
(If your group size is < 4 - please some person from another group to also
participate in your group)

To generate, I used a program with the key `--gen-key`
The program will request my personal data in the process (figure 10)
```
gpg --gen-key
```
<center>
    
![](https://i.imgur.com/WCKjrJ6.png)
Figure 10: Program output when generating a key
</center>

To view a list of all my keys, I can use the flag `--list-keys`
Here I can see the keys, their owners and when they were generated (figure 10)
```
gpg --list-keys
```

<center>
    
![](https://i.imgur.com/bdb2MA6.png)
Figure 11: Program output
</center>

To upload your key to the storage server
To do this, I need to specify the `--send-keys` flag with the key id
Also an optional parameter is `--keyserver`, which allows you to manually specify the server.
```
gpg --keyserver keyserver.ubuntu.com --send-keys 5A9F476CB93AD0E7BA7BABD18EAEAC08F8856EF8
```

To get keys from other team members, I can specify `--recieve-keys` with the key id
If successful, I will receive a success message and information about the owner of the key (figure 12)

```
gpg --keyserver keyserver.ubuntu.com --receive-keys 46D48A64C97910D23E4ADED41518B420223D9541
```
<center>
    
![](https://i.imgur.com/mJztIlF.png)
Figure 12: Message about successful key import from the storage server
</center>


### Stage 2
After receiving the key, I need to establish trust in the messages signed with the key. In my case, I will do this for the keys `Alisher` and `Ilya`
as a demonstration, I will show the key signing command `Ilya`

```
gpg --sign-key 46D48A64C97910D23E4ADED41518B420223D9541
```

<center>
    
![](https://i.imgur.com/SMchOzZ.png)
Figure 13: Ilya's key signature
</center>
After signing, it will be necessary to upload information about its key to the server

#### Encryption: 
Now I will encrypt the message to the person with whom I have not established trust by signing the key
To do this, I will use the `--encrypt` flag
`--armor` - for output in `ascii` format
`--sign` - make a signature
`-r` - recipient

```
gpg --encrypt --sign --armor -r v.shelkovnikov@innopolis.university message.txt 
```

After that, I got a file in the directory `message.txt.asc`, which I will have to send to `Vladimir`

#### Decryption:
To decrypt, I only need to pass an encrypted file with the `--decrypt` flag
```
gpg --decrypt ~/Downloads/message.asc 
```
After entering the command, the initial message will be displayed. But it is very important that there is a key from the person who encrypted this message

Since I received the message from `Vladimir`, and I did not sign his key, I will receive a warning message, since the message was not delivered from a trusted user (figure 14)
<center>
    
![](https://i.imgur.com/wFOnIst.png)
Figure 14: Output of a decrypted message from a person with whom I have not established trust
</center>

If a message is received from a trusted sender, there will be no such errors (figure 15)

<center>
    
![](https://i.imgur.com/0EbY6zP.png)
Figure 15: Decryption of a message from a trusted user
</center>


:::


## References:

1. [Prime number theorem](https://en.m.wikipedia.org/wiki/Prime_number_theorem)
1. [100 digit bigint calculator](http://www.javascripter.net/math/calculators/100digitbigintcalculator.htm)
2. [How to use gpg to encrypt and sign messages](https://www.digitalocean.com/community/tutorials/how-to-use-gpg-to-encrypt-and-sign-messages)