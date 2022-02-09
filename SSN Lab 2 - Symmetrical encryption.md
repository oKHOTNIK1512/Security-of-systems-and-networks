---
tags: Security of systems and networks
---
:::success
# SSN Lab 2 - Symmetrical encryption
Name: Ivan Okhotnikov
:::


## Task 1 - DES
:::warning
Watch this visualization:
1. Next, use the DES simulator (launch .jar file). Step through the process of encrypting your name with the key 0x0101010101010101 and write the internal state of the device at the 8th round.
2. Inspect the key schedule phase for the given key and explain how the sub keys are generated for each of the 16 steps.
3. Comment on the behavior of DES when using the given key.
:::

## Implementation
:::info
> 1. Next, use the DES simulator (launch .jar file). Step through the process of encrypting your name with the key 0x0101010101010101 and write the internal state of the device at the 8th round.

`IVAN` - in HEX is `49 56 41 4e`
Since the program can process text of 8 and 16 characters, I will encrypt the text `IVANIVAN`, is `4956414e4956414e` in hexadecimal format

<center>
    
![](https://i.imgur.com/Ey9rqdP.png)
Figure 1: Program window 'DES Block Cipher Calculator`
</center>

In the eighth round , the value will be: `db1ffce4`
```
Rnd8	f(R7=1316e43c, SK8=00 00 00 00 00 00 00 00 ) = db1ffce4
```
:::

:::info
> 2. Inspect the key schedule phase for the given key and explain how the sub keys are generated for each of the 16 steps.

Since the key after discarding the extra characters is 56 bits. Next, it will be necessary to compress it to 48 bits using the table (figure 2). Now you can use it in the DES algorithm. Since there are 16 steps, the value of the encryption key is constantly shifting. It depends on the table (figure 3)

<center>
    
![](https://i.imgur.com/IGwryts.png)
Figure 2: DES key compression table
![](https://i.imgur.com/bpxXKL0.png)
Figure 3: Key offset table for each round
</center>
:::

:::info
> 3. Comment on the behavior of DES when using the given key.

The key `0101010101010101` for all rounds (as seen in figure 1) gets the value `0000000000000000`. The reason for this is that every 8 bits equal to `1` was thrown out during the compression process from 64 ->56 bits.

By converting the key from 16 to 2
```
100000001000000010000000100000001000000010000000100000001
```
we remove every 8 and it will turn out
```
0000000000000000000000000000000000000000000000000
```

:::

## Task 2 - AES
:::warning
4. Identify the Shannon diffusion element(s).
5. Also identify the Shannon confusion element(s).
:::

## Implementation
:::info
> 4. Identify the Shannon diffusion element(s).

It means that if we replace 1 bit in plaintext, then more than half of the encrypted bits should change and vice versa. This is necessary in order to hide the static connection between the open and encrypted texts.

For example, I will encrypt my name with the AES algorithm in CBC mode with a key length of 128 bits and the key `0101010101010101`

**Ivan**
`D95019167C85BB4F728169C6F6E24E92`
Now I will change the text minimally - I will translate the last letter into uppercase:
**IvaN**
`8917C3C36004581FB684A01EAAC22E19`

When comparing, it turned out to set duplicate characters:
`-9------------- F--8--------2- E--`
Since there are more than half of the repeated characters, the `diffusion` requirement is met
:::

:::info
> 5. Also identify the Shannon confusion element(s).

It means that each character of the plaintext must depend on several parts of the key. This is necessary in order to hide the relationship between the key and the text.

To explain this, you need to refer to the AES encryption scheme.
Since the keys for each round are generated based on the original encryption key (and there are 10 of them), the text will depend on several parts of the original key.
:::

## Task 3 - RC4
:::warning
Follow these instructions, identify the URL and your personal archive accordingly, download it and inspect its contents. There are two files encrypted with the RC4 cipher. 
One of the files was encrypted using a 40 bit key that when represented in ASCII starts with the character a and contains only lowercase letters while the other uses a 48 bit key that can be written only with digits. Identify the encrypted files and using the brute force tool from this gist find the keys and decrypt the file. (Most likely you will have to install the pycrypto and
numpy libraries first.)
6. 
a. How did you identify the encrypted files?
b. What is the effective key strength for each of the keys?
c. Instrument the code to find out how many decryption attempts you can perform in one second. Where is the most time spent ?
d. Modify the code to support parallel execution and calculate the speedup.
e. If the same message would be encrypted with a key of length 48 bits but which uses all the printable characters, how much time would it take to explore the full key space ?
:::

## Implementation
:::info
> a. How did you identify the encrypted files?

To decipher the original message, I first found out what cipher it was encrypted with. To do this, I used the [service for determining](https://www.dcode.fr/cipher-identifier) (figure 4):

<center>
    
![](https://i.imgur.com/Gt8HB96.png)
Figure 4:
</center>

Thanks to the service, I found out that [`Affine Cipher`](https://www.dcode.fr/affine-cipher) is used

After deciphering the first part of the message, I found out that my number - `11` (figure 5)
<center>
    
![](https://i.imgur.com/DyuiHfu.png)
Figure 5: The decryption window of the first part of the message
</center>

After deciphering the second part of the message, I found out where to download the archive​ (figure 6)
<center>
    
![](https://i.imgur.com/bu4xyhC.png)
Figure 6: The decryption window of the second part of the message
</center>

<!-- 
```
Get a full zip archive at
https://drive.google.com/drive/folders/0B10C_u35YDgFMEMzVkZOTklvakU?resourcekey=0-VzaIrHN5S_UUsWpIBqtrcQ&usp=sharing
choose your personal archive according to your number in the list above
``` -->

After downloading the archive and unpacking it, I received the following files (figure 7):
<center>
    
![](https://i.imgur.com/tyvAmCl.png)
Figure 7: List of downloaded files
</center>

The remaining task is to determine which of these files are encrypted. To do this, I used the `ent program`. It will be necessary to determine the entropy value of encrypted files, since such files will have the most value.
`Entropy` is a concept that reflects the density of content in a file, defined as the ratio of the number of bits per character.
For each file and the result was placed in the file `ent` and then looked at which files have the highest entropy value (figure 8)
<center>
    
![](https://i.imgur.com/cTV2Nf6.png)
Figure 8: Entropy value for each file
</center>
    
The entropy of these files is greater, so I will continue working with these files:
```
10693284.in
15029836.in
```
    

:::

:::info
> b. What is the effective key strength for each of the keys?

The key strength for a key containing lowercase alphabet characters is `2^40`, and for a key for a digit key is `2^48`, which is more efficient and requires more time to iterate.
    
Figure 9,10 shows that the time for selecting a `40-bit` key was `189 seconds`, and for a `48-bit` key `473 seconds`
<center>

![](https://i.imgur.com/xDXGqjX.png)
Figure 9: The result of decrypting the key with letters
![](https://i.imgur.com/tWsm3q2.png)
Figure 10: The result of decrypting the key with numbers
</center>


:::

:::info
> c. Instrument the code to find out how many decryption attempts you can perform in one second. Where is the most time spent ?

For a `48-bit` key and a `40-bit` one, approximately the same number of keys per second comes out, about `1340 keys` per second (figure 11,12)
<center>
    
![](https://i.imgur.com/1R9bmJk.png)
Figure 11: The result of decrypting the key with letters
    
![](https://i.imgur.com/DMSHibX.png)
Figure 12: The result of decrypting the key with numbers
</center>


:::

:::info
> d. Modify the code to support parallel execution and calculate the speedup.

In the case of a parallel start of key selection using 2 streams, the following data was obtained (figure 13,14)
For a `40-bit key`, the execution time decreased by `124 seconds`
For a `48-bit key`, the execution time was reduced by `150 seconds`
<center> 
    
![](https://i.imgur.com/Zlimfzk.png)
Figure 13: The result of decrypting the key with letters
![](https://i.imgur.com/TXLaxHJ.png)
Figure 14: The result of decrypting the key with numbers
</center>

:::

:::info
> e. If the same message would be encrypted with a key of length 48 bits but which uses all the printable characters, how much time would it take to explore the full key space?

Since there are 95 printable characters, and 1 character weighs 8 bits (only 6 characters in the key) that number of possible options will be `95^6` or `735091890625`.
If you perform a brute force in 1 thread, then on average it turns out to process 1340 keys per second.
Then the time for selecting all keys will be `735091890625/1340` seconds or `548576038` seconds or `9142934` minutes or `152382` hours or `6349` days or `17 years`


:::

## Task 4 - AES - 2
:::warning
7. Modify the code to support AES brute-force in CBC mode. How many keys can you test per second? Julian Assange has released an insurance file encrypted with AES256. Assuming that no disruptive technological breakthrough will take place in the future and the performance of CPUs will double every 18 months, when will it be possible to brute-force the file in reasonable time, i.e. less than 1year, using a single computer?
:::

## Implementation
:::info
> Modify the code to support AES brute-force in CBC mode. How many keys can you test per second?

**Reworking the program:**
Because the key will consist of 16 characters. I specified this in the program settings.
I also changed the decryption process (now I use AES). It will be more convenient for me to generate the text in base64 for decryption (so I will have it pre-decrypted)

```
data = open(FILE_NAME, 'rb').read(2**16)
data = base64.b64decode(data)
...
...
...
decr = AES.new(key, AES.MODE_CBC).decrypt(data[16:])
```
I also changed the threshold value of entropy to determine the decrypted text at 6. In other words, if the value is below 6, then the key is found correctly
    
<center>
    
![](https://i.imgur.com/WRxSgl9.png)
Figure 15: Running key search
</center>
    
My program can iterate through an average of `11387` keys per second.

---
    
Next, I decided to check how long it would take to decrypt the message.
I took the key `aaaaaaaaaaasbasa` and encrypted the text.
I put the encrypted file in the `base64` format in the file and launched the brute force (with the alphabet set to lowercase letters)
After `731 seconds` the program was able to find the key to my text
<center>
    
![](https://i.imgur.com/pD3I7Hx.png)
Figure 16: Program output
</center>
:::

:::info
> Julian Assange has released an insurance file encrypted with AES256. Assuming that no disruptive technological breakthrough will take place in the future and the performance of CPUs will double every 18 months, when will it be possible to brute-force the file in reasonable time, i.e. less than 1year, using a single computer?
    
`2^256 /31536000` to get the required value of the number of keys per second, where `31536000` is the number of seconds in a 1 year

is `3,67 * 10^69` keys per second

    
`11500` - I have keys per second for my computer
`11500 * 2^x = 3,67 * 10^69`
x = `2,48 * 10^69`
Therefore, it will be necessary to wait for `2,48 * 10^69` periods
Since my power increase period is 2 times a year, I still need to multiply this value by 1.5

Answer: `3.72 * 10^69`

:::
    
## References
1. [AES animation](https://formaestudio.com/rijndaelinspector/archivos/Rijndael_Animation_v4_eng-html5.html)
1. [Chipher identifier](https://www.dcode.fr/cipher-identifier)
2. [Affine Cipher](https://www.dcode.fr/affine-cipher)
3. [Confusion and Diffusion](https://en.wikipedia.org/wiki/Confusion_and_diffusion)