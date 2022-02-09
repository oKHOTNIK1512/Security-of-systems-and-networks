---
tags: Security of systems and networks
---
:::success
# SSN Lab 2 - UEFI Secure Boot
Name: Ivan Okhotnikov
:::

:::warning
**Introduction**
UEFI Secure Boot ensures by means of digital signatures that the code that loads the operating system and the operating system itself have not been tampered with. It is used by Microsoft to guarantee safe startup, but also by Ubuntu and Fedora.
In this assignment you will learn how Ubuntu implements secure booting of the Linux kernel. You will need access to a Ubuntu machine that was installed with UEFI Secure Boot enabled (like your desktop PC).
:::


## Task 1 - Firmware Databases
:::warning
1. Extract the Microsoft certificate that belongs to the key referred to in Step 1 from the UEFI firmware, and show its text representation.
Hint: efitools, openssl x509
2. Is this certificate the root certificate in the chain of trust? What is the role of the Platform Key (PK) and other variables?
:::

## Implementation
:::info
> 1. Extract the Microsoft certificate that belongs to the key referred to in Step 1 from the UEFI firmware, and show its text representation.
>> Hint: efitools, openssl x509

The first thing I need to do is check if I have secure boot enabled. I can do this using the `--sb-state` flag in the mokutil program (figure 1)

<center>

![](https://i.imgur.com/6TlDNJb.png)
Figure 1: Checking the secure boot status on my workstation
</center>

The Microsoft certificate is located in 'KEK' because it is the public key of the operating system

**I can get it in two ways:**

- Using `efi-readvar`:
```
efi-readvar -v KEK -o ~/mscertificate.txt
```
- Using `mokutil`:
```
mokutil --kek > ~/mscert.txt
```

The difference between these approaches is that using mokutil I can get more readable information
<center>

![](https://i.imgur.com/75GT7rz.png)
Figure 2: Output of the `efi-readvar` program
![](https://i.imgur.com/Q4FDGQB.png)
Figure 3: Output of the `mokutil` program
</center>
:::

:::info
> 2. Is this certificate the root certificate in the chain of trust? What is the role of the Platform Key (PK) and other variables?

No, it is not. The root of trust is the PK key.
`PK (Platform Key)` - a key from the hardware manufacturer (in my case, this is Lenovo)
It is needed to update UEFI and download new KEK keys

`KEK  (Key Enrollment Key)` - the public keys of the operating system are stored here

There is also `DB` and `DBX`. These are signature and hash databases for trusted and untrusted applications. in other words, `DBX` is a blacklist, `DB` is a whitelist

<center>

![](https://i.imgur.com/jBcMiiv.png)
Figure 4: example of my `PK certificate`
</center>
:::

## Task 2 - SHIM
:::warning
3. Verify that the system indeed boots the shim boot loader in the first stage. What is the full path name of this boot loader?
4. Verify that the shim bootloader is indeed signed with the “Microsoft Corporation UEFI CA” key.
Hint: sbsigntool, PEM format
5. What is the exact name of the part of the binary where the actual signature is stored?
6. In what standard cryptographic format is the signature data stored?
7. Extract the signature data from the shim binary using dd. Add 8 bytes to the location as given in the data directory to skip over the Microsoft WIN CERTIFICAT structure header (see page 14 of the specification if you are interested).
8. Show the subject and issuer of all X.509 certificates stored in the signature data.
Draw a diagram relating these certificates to the “Microsoft Corporation UEFI CA” certificate
Now we know that the system indeed boots the shim, and that this OS loader is indeed signed by Microsoft and where this signature is stored
:::

## Implementation
:::info
> 3. Verify that the system indeed boots the shim boot loader in the first stage. What is the full path name of this boot loader?

In order to check the boot sequence, I used the `efiboot` program
```
sudo efibootmgr
```

After checking the output of the program, I made sure that the first in the queue is the ubuntu download (figure 5)
<center>

![](https://i.imgur.com/4RDxwAL.png)
Figure 5: Output of the program loading sequence `efiboot`
</center>

In order to find out the path to the loader, I used the `bootctl` program
```
sudo bootctl status
```
After studying the output of the program, I found out that the full path to the operating system boot file is in - `/boot/efi/EFI/ubuntu/shim64.efi` (figure 6)

<center>

![](https://i.imgur.com/ETDTjtX.png)
Figure 6: Part of the output of the `bootctl` program
</center>
:::

:::info
> 4. Verify that the shim bootloader is indeed signed with the “Microsoft Corporation UEFI CA” key.
Hint: sbsigntool, PEM format

`Shim` is a preloader that allows you to boot an operating system based on `SecureBoot`

This preloader was signed as an application by the Microsoft Certification Authority, so the certificate for its validation should be searched in `db`.

You can find out the certificate with which you can check `shim` using the `sbverify` command and the `--list` flag

```
sudo sbverify --list /boot/efi/EFI/ubuntu/shimx64.efi
```
<center>

![](https://i.imgur.com/ka3dWM6.png)
Figure 7: Information about `shim` certificates
</center>
Here I found out that I need a `Microsoft Corporation UEFI CA 2011` certificate, which is not difficult to find in the list of db keys using `mokutil --db` (figure 8)
It is necessary to remember the number of his key - later this will help in identifying him when exporting. His number is `3`
<center>

![](https://i.imgur.com/ACwL569.png)
Figure 8: The required key for shim verification is in the output of the `mokutil` command
</center>

To get all the db keys, you need to add the `--export` flag to the previous command (figure 9)

```
mokutil --export --db
```

<center>

![](https://i.imgur.com/w1liNal.png)
Figure 9: List of received `db` keys as a result of their export
</center>

As I said earlier, I need a 3 key, and I will continue to work with it.
To work with it and verify with it, you will need to convert it to the `PEM` type using the `openssl` program:

```
sudo openssl x509 -inform der -outform pem -in DB-0003.der -out DB-0003.pem
```

Now it is possible to verify shim. This is done by forcibly specifying the certificate `--cert` (figure 10)
```
sudo sbverify --cert DB-0003.pem /boot/efi/EFI/ubuntu/shimx64.efi
```
<center>

![](https://i.imgur.com/eLU1w4C.png)
Figure 10: The result of successful verification of `shim`
</center>
:::

:::info
> 5. What is the exact name of the part of the binary where the actual signature is stored?

It is called `SignedData`
:::

:::info
> 6. In what standard cryptographic format is the signature data stored?

The signature data is stored in the cryptographic format 'PKCS#7'. This data is signed with `X.509` certificates

The `PKCS#7` format is described in figure 11
<center>

![](https://i.imgur.com/uKG2xqr.png)
Figure 11: The `PKCS#7` format
</center>
:::

:::info
> 7. Extract the signature data from the shim binary using dd. Add 8 bytes to the location as given in the data directory to skip over the Microsoft WIN CERTIFICAT structure header (see page 14 of the specification if you are interested).

To output the information, I used the program `pyew`.
After studying the `efi file`, I can find out information about the information inside (figure 12)
<center>

![](https://i.imgur.com/mz6krmE.png)
Figure 12: Output information about the contents of the 'shimx64' file
</center>

The size of the block I am interested in is `2140` Its beginning is located at `0xe73cb` (`947147`).
Using this data I can make a dump.
The task asks you to make an offset of `+8`, so the following command is obtained:
```
sudo dd if=/boot/efi/EFI/ubuntu/shimx64.efi skip=947155 bs=1 count=2140
```
<center>

![](https://i.imgur.com/OXC9SJI.png)
Figure 13: Dump `shimx64.efi` in the program `dumpasn1`
</center>
:::

:::info
> 8. Show the subject and issuer of all X.509 certificates stored in the signature data.
Draw a diagram relating these certificates to the “Microsoft Corporation UEFI CA” certificate
Now we know that the system indeed boots the shim, and that this OS loader is indeed signed by Microsoft and where this signature is stored

I can get a list of certificates for shim using the command (figure 14):

```
sudo sbverify --list /boot/efi/EFI/ubuntu/shimx64.efi
```
<center>

![](https://i.imgur.com/KHKwj1z.png)
Figure 14:List of certificates for `shimx64.efi`
</center>

Having drawn up a dependency diagram, the following turned out (figure 15)

<center>

![](https://i.imgur.com/yWDGwYJ.png)
Figure 15: Certificate Dependency Scheme (Issue/Subject)
</center>

:::

## Task 3 - GRUB

:::warning
In Step 2 of the Ubuntu description it says that “the second stage bootloader (grub-efiamd64-signed) is signed with Canonical’s “Canonical Ltd. Secure Boot Signing” key.” This boot loader binary is called grubx64.efi and is located in the same directory as the shim binary.
9. Using your new knowledge about Authenticode binaries, extract the signing certificates from the GRUB boot loader, and show the subject and issuer.
You now know that the GRUB binary is indeed signed, but how does shim verify this signature? To do so shim needs a certificate that it trusts and which must not be modifiable by an external party without detection. The solution chosen by Ubuntu is to store a certificate in the shim binary. We will call this certificate X.
10. Why is storing the certificate X in the shim binary secure?
11. What do you think is the subject CommonName (CN) of this X certificate?
12. Obtain the X certificate used by the shim to verify the GRUB binary.
There are two ways to obtain it: from the source code or from the binary directly.
Hints for the latter case:
- The certificate is in X.509 DER/ASN.1 format (see openssl asn1parse)
- DER/ASN.1 leaves the CommonName readable
- The certificate is 1080 bytes long
13. Verify that this X certificate’s corresponding private key was indeed used to sign the GRUB binary.
:::

## Implements:

:::info
> 9. Using your new knowledge about Authenticode binaries, extract the signing certificates from the GRUB boot loader, and show the subject and issuer.

The first step is to find out the beginning and size of `DIRECTORY_ENTRY_SECURITY` using pie (figure 16, 17)

<center>

![](https://i.imgur.com/tCGkjR6.png)
Figure 16: Information about the contents of `grubx64.efi` in the `pyew` program
</center>

After that, I can now make a dump and see the contents of the certificates (figure 17)
<center>

![](https://i.imgur.com/EL1tuoP.png)
Figure 17:  Information about the `Issuer` in the `dumpasn1` program for the `grubx64.efi` file

![](https://i.imgur.com/ZO2PlIV.png)
Figure 18: Information about the `Subject` in the `dumpasn1` program for the `grubx64.efi` file
</center>


Issuer: `/C=GB/ST=Isle of Man/L=Douglas/O=Canonical Ltd./CN=Canonical Ltd. Master Certificate Authority`
Subject: `/C=GB/ST=Isle of Man/O=Canonical Ltd./OU=SecureBoot/CN=Canonical Ltd. Secure Boot Signing (2017)`

Information about `Issuer` and `Subject` could be obtained using dbverify (figure 18)
```
sudo sbverify --list /boot/efi/EFI/ubuntu/grubx64.efi
```

<center>

![](https://i.imgur.com/Z7VWfxV.png)
Figure 18: Information about `grubx64.efi` certificates
</center>
:::

:::info
> 10. Why is storing the certificate X in the shim binary secure?

Because it will be necessary to check GRUB before starting it (since shim loads it)
:::

:::info
> 11. What do you think is the subject CommonName (CN) of this X certificate?

CN can be seen in the information given by `dumpasn1` - `Canonical Ltd. Secure Boot Signing (2017)`

<center>

![](https://i.imgur.com/VaiBS25.png)
Figure 19: `СommonName` for `Subject` in the dump
</center>
:::

:::info
> 12. Obtain the X certificate used by the shim to verify the GRUB binary.
There are two ways to obtain it: from the source code or from the binary directly.
>>Hints for the latter case:
>> - The certificate is in X.509 DER/ASN.1 format (see openssl asn1parse)
>> - DER/ASN.1 leaves the CommonName readable
>> - The certificate is 1080 bytes long

To get a certificate, I used the program 'mokutil`:
```
mokutil --export
```
After executing the command, all certificates will be unloaded. I will need a certificate `MOK` (Machine Owner Key)

Since the contents are in `DER` format, it will be necessary to convert them to `PEM` for further verification:
```
sudo openssl x509 -inform der -outform pem -in MOK-0001.der -out MOK-0001.pem
```

:::


:::info
> 13. Verify that this X certificate’s corresponding private key was indeed used to sign the GRUB binary.

To do this, I will use the already familiar `sbverify` program with the certificate (figure 19)

```
sudo sbverify --cert MOK-0001.pem /boot/efi/EFI/ubuntu/grubx64.efi
```

<center>

![](https://i.imgur.com/TxLoEUX.png)
Figure 20: The result of `grubx64.efi' verification using a certificate
</center>
:::


## Task 4 - The Kernel
:::warning
GRUB allows the user to select from a list of kernels. According to Step 3, GRUB will check if the chosen kernel is signed. If so, GRUB leaves the system in UEFI mode before loading and running the kernel, so that it still has access to the UEFI services. As with shim, GRUB needs a trusted certificate against which it can verify the signed kernel. This is the same certificate as the shim uses.
14. Verify the kernel you booted against the X certificate.
15. BONUS: Where does GRUB get its trusted certificate from? Hint: It is not stored in the binary, and it is not stored on the file system.
BONUS: We have now verified Steps 1–3 of the Ubuntu chain of trust.
Verifying Step 4 is left as a bonus.
16. Draw a diagram that shows the chain of trust from the UEFI PK key to the signed kernel. Show all certificates, binaries and signing relations involved
:::

## Implementation
:::info
> 14. Verify the kernel you booted against the X certificate.

To do this, I verified the latest version `vmlinuz-5.11.0-38-generic` (figure 21) 
But you can also check other versions, since they are signed with the same key

<center>

![](https://i.imgur.com/n1dbDuE.png)
Figure 21: Verification result
</center>
:::

:::info
> 15. BONUS: Where does GRUB get its trusted certificate from? 
> >Hint: It is not stored in the binary, and it is not stored on the file system.
>BONUS: We have now verified Steps 1–3 of the Ubuntu chain of trust.
Verifying Step 4 is left as a bonus.

`GRUB` gets it from `MOK` (Machine Owner Key)
:::

:::info
> 16. Draw a diagram that shows the chain of trust from the UEFI PK key to the signed kernel. Show all certificates, binaries and signing relations involved

<center>

![](https://i.imgur.com/Vb626SB.png)
Figure 22: A diagram that shows the chain of trust from the UEFI PK key to the signed kernel
</center>
:::


## References:
1. [UEFI with secure boot](https://ichip.ru/tekhnologii/uefi-s-secure-boot-bezopasnaya-zagruzka-pk-2069)
2. [Managing EFI in Linux](http://rus-linux.net/MyLDP/boot/managing-EFI-in-Linux-2.html)
3. [PEM, DER, CRT and CER](https://www.ssl.com/ru/%D1%80%D1%83%D0%BA%D0%BE%D0%B2%D0%BE%D0%B4%D1%81%D1%82%D0%B2%D0%BE/%D0%BA%D0%BE%D0%B4%D0%B8%D1%80%D0%BE%D0%B2%D0%BA%D0%B8-%D0%B8-%D0%BF%D1%80%D0%B5%D0%BE%D0%B1%D1%80%D0%B0%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F-pem-der-crt-%D0%B8-cer-x-509/)
4. [We use Secure Boot in Linux to the fullest](https://habr.com/ru/post/308032/)