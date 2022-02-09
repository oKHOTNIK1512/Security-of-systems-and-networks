---
tags: Security of systems and networks
---
:::success
# SSN Lab 1 - Classical crypto
Name: Ivan Okhotnikov
:::

:::warning
**Introduction**
In this assignment, you will look at classical methods of encoding/decoding.
Please keep your log detailed, give more than just an account of the
questions/answers posed in this assignment. Especially during group work it should be clear from the logs who did what and why.
:::


## Task 2
:::warning
Encrypt an English text of at least 80 words using the Vigenere cipher and exchange it with one of your fellow students
:::

## Implementation
:::info
To encrypt my text in Vigenere cipher, I used the website: https://www.dcode.fr/vigenere-cipher


Here I can specify the text, the encryption key, the alphabet and save punctuation marks with a separate flag (to make it easier to read after decryption) (Figure 1)

<center>

![](https://i.imgur.com/p82PLVU.png)
Figure 1: A window for encrypting text
</center>

I chose the word as the key: `MAZAHAKA`
Source text: 

```
Article 228 of the Criminal Code of the Russian Federation. Illegal acquisition, storage, transportation, manufacture, processing of narcotic drugs, psychotropic substances or their analogues, as well as illegal acquisition, storage, transportation of plants containing narcotic drugs or psychotropic substances, or their parts containing narcotic drugs or psychotropic substances

1. Illegal acquisition, storage, transportation, manufacture, processing without a purpose of selling of narcotics, psychotropic substances or their analogs in a significant amount, as well as illegal acquisition, storage, transportation, without a purpose of selling of plants containing narcotic drugs or psychotropic substances, or their parts containing narcotic drugs or psychotropic substances in large size
```

After encryption , the following text was obtained:

```
Mrsijlo 228 or tge Jrsmunzl Jone af shl Reseizn Menedasivn. Slxefas amqgiriaiyn, etnrhgo, tdamswobtmthou, mkngfzcaube, brnclscizg nf uabcathc krege, pryjhytdooij sebetznjec od tgepr knmlngbec, ae wdls ac ixldghl kccuhsptsoz, ssoyaqe, frznzpyrfasivn yf blznas moztziuixg zaqcvtsc prtgz ob peybhvtbobib sbbctmnbez, ob ttehr wabte cnnaasnunf nhrmofib dyuqs ar osfcrofrnppc cunssaucos

1. Ulkenav aoqtizidian, rtvrkgq, tqauszodtztpox, mmntfhcdude, orvcoseimg didhaus a wubpasd om solximg vf xadcntpcc, peybhvtbobib sbbctmnbez ob ttehr hnklagr iu a cisnhfpcknf alobnd, ae wdls ac ixldghl kccuhsptsoz, ssoyaqe, frznzpyrfasivn, gifhnua a zudpnsl op sqlkiug yf blznas moztziuixg zaqcvtsc prtgz ob peybhvtbobib sbbctmnbez, ob ttehr wabte cnnaasnunf nhrmofib dyuqs ar osfcrofrnppc cunssaucos un kaygo suzd
```

I did not include numbers in the encryption alphabet

And now about the encryption algorithm itself
The `Vigenere cipher` is a `multialphabetic cipher` that consists of several alphabets that are defined by the `Caesar cipher` with different shift values(figure 2)

<center>

![](https://i.imgur.com/UOcHukp.png)
Figure 2: Vigener Square
</center>


And now I'll move on to how it works

For example, I'll take a part from my source text and a key.

```
Article 228 of the Criminal Code of the Russian Federation.
```

To encrypt this text manually, you will need to take my key and duplicate it if necessary:

```
Article 228 of the Criminal Code of the Russian Federation.
MAZAHAK     AM AZA HAKAMAZA HAKA MA ZAH AKAMAZA HAKAMAZAHA
------
Mrsijlo 228 or tge Jrsmunzl Jone af shl Reseizn Menedasivn.
```

If the whole key does not fit completely, it's okay

Now for encryption, it is necessary to correlate the letter from the message text in the `Vigener Square` (select a column), and the cipher letter will show which line to select. The intersection of a row and a column will show which letter will be in the encrypted text

:::


## Task 3
:::warning
Crack the encrypted text of your fellow student using the Vigenere cipher tool.
:::

## Implementation
:::info

From `Alexander`, I received a text that he encrypted with `Vigenere cipher` using his key:
```
I mziosmaafy wwnidxopg xpwmtbbb, ezuca wn lwrigsy bg unvzplw ooimdvy mnhhcmj etnrzvl’e hhazegdk tgnqyzmxbo (eafh hf rqltonh kmjyilgdwf). M chakclqr lqdmfoe vzvak dejidzwe tas nbmpeghn bg po mvzqj tofsrwjw og hcm vqptfouwzt’l qjuhgtxf. Jvw etnrzvl zomwxmk fhth v awoogr nbmpegh cik zom fziv-brhhzklqd mvz naxe vcibsunbbb pwd hhazegdk tby kgbixg db. Zms xwopwd smiymff (ok vvdw nomv nbmpeghn) jjqavvzl kqcnfdbq?
Fhx fzbgdt mvvb lte ywmal gsxf xwmxd vckg lte ywgmk, mnw hcmjqfhfz bzq avhdwf us tzgwoqd, vcinmeel hcm eqcaoiqky wbhc xgxivm.
```

My task is to decrypt the text without knowing the key that was used in the encryption.

The first thing I did for the convenience of cracking the encrypted text was to remove all spaces and special characters (this will simplify my task)
```
ImziosmaafywwnidxopgxpwmtbbbezucawnlwrigsybgunvzplwooimdvymnhhcmjetnrzvlehhazegdktgnqyzmxboeafhhfrqltonhkmjyilgdwfMchakclqrlqdmfoevzvakdejidzwetasnbmpeghnbgpomvzqjtofsrwjwoghcmvqptfouwztlqjuhgtxfJvwetnrzvlzomwxmkfhthvawoogrnbmpeghcikzomfzivbrhhzklqdmvznaxevcibsunbbbpwdhhazegdktbykgbixgdbZmsxwopwdsmiymffokvvdwnomvnbmpeghnjjqavvzlkqcnfdbqFhxfzbgdtmvvblteywmalgsxfxwmxdvckglteywgmkmnwhcmjqfhfzbzqavhdwfustzgwoqdvcinmeelhcmeqcaoiqkywbhcxgxivm
```

The `CodeBook` program already has built-in programs to help crack this cipher (figure 3)
In it, I entered the encrypted text in the input field.
<center>

![](https://i.imgur.com/wIH1kXW.png)
Figure 3: Window for breaking the `Vigener cipher`
</center>

The next step in hacking is to find out the length of the key that was used in encrypting the message
To do this, I will go to the search box for repeated parts (the `Find repeated Sequences` button) (figure 4)

<center>

![](https://i.imgur.com/7uhLlHB.png)
Figure 4: Duplicate Parts Search Box
</center>

In this window, you can see the repeating parts themselves and the distance between them
For example 'BBB' has a distance of `238`
This number is divisible by `2` and `7` without remainder (therefore `2` and `7` are marked for it)

It will not be difficult to choose the length of the key here - choose the one that has the largest number of marks opposite the repeating parts
To select, you will need to click on the required key length - in my case, the key length is `7`

After selecting the length of the key, the program will return me to the previous window (only now the key selection will be available) (figure 5). The task of selecting the necessary key symbol is to bring part of the encrypted text to a state similar to the normal distribution of characters. For convenience, the normal distribution of characters in English is given

I'll start with the first character. I stopped at the letter `I` - because only this state is more similar to the normal distribution (figure 5)

<center>

![](https://i.imgur.com/dZ7bCNN.png)
Figure 5: A search box for duplicate parts, with the ability to select a key
</center>

I perform this procedure for the other letters in the encryption key as well
I eventually got the encryption key - `ISMATOV` (figure 6)

<center>

![](https://i.imgur.com/dLFUWza.png)
Figure 6: A window with a cracked encryption key and decrypted text
</center>


I can check the correctness of the key by the field with the decrypted text.
Since I previously got rid of special characters and spaces to crack the key more correctly, my decrypted text will be as follows:

```
auniversitydisallowscheatingwhichisdefinedtoincludecopyinganotherstudentshomeworkassignmentwithorwithoutpermissionacomputerscienceclassrequiresthestudentstodotheirhomeworkonthedepartmentscomputeronestudentnoticesthatasecondstudenthasnotreadprotectedthefilecontainingherhomeworkandcopiesithaseitherstudentorhavebothstudentsbreachedsecuritytheretortthatthefirstusercouldcopythefilesandthereforetheactionisallowedconfusesthemechanismwithpolicy
```

:::

## Task 4
:::warning
Go through the previous two steps again, this time using a cipher of your own choosing. Do not tell your fellow student what cipher you used!
:::

## Implementation


### Encrypt your text using any algorithm
:::info
For encryption, I decided to use the transpose method
For it to work, it is necessary that there is a text (which I want to encrypt) and an offset value.
It works as follows:
- The text is taken (for example, the one I indicated below)
```
Early in the morning on the fourteenth of the spring month of Nisan the
Procurator of Judaea, Pontius Pilate, in a white
```
- the text gets rid of spaces and special characters

```
Earlyinthemorningonthefourteenthofthespringmonthofnisanthe
procuratorofjudaeapontiuspilateinawhite
```
- then the encryption itself takes place 
 the text is taken and written from top to bottom, for example, the word Early will look like:
```
E
a
r
l
y
```

I want every 3 characters to be separated, so it will look like this:

```
elnernneuetfergnoineorofdanuitnhe
aytmngtfrehtsimtfstpcarjaptsleai
rihoiohotnohpnohnahrutoueoipaiwt
```

The final cipher will look like this:

```
elnernneuetfergnoineorofdanuitnheaytmngtfrehtsimtfstpcarjaptsleairihoiohotnohpnohnahrutoueoipaiwt
```

To decipher it, you will need to divide the text into 3 parts and perform the reverse actions


:::

### Breaking a cipher from a partner
:::info

`Ilya` encrypted some text for me and sent it to me
```
Urag lzaarazot ingd dpzghw 'dozqq' bii gabqqzh gwbg zd trzot zo bok
rjg rq ron urag (lzaara-drjahn) bok dnok b hruv rq gwrdn ubhfngd rjg
rq drln rgwna urag (lzaara-gbatng). Gwzd qnbgjan hbo en jdnk gr nbdziv
dng ju b 'gbu' knmzhn gwbg anhnzmnd bii gabqqzh gwbg trnd zo/rjg rq
drln dunhzqzh urag. Orgn gwbg lzaara-drjahn bok lzaara-gbatng uragd
wbmn gr enirot gr dbln dpzghw. (Dnn pwzhw urag enirot gr pwzhw dpzghw
zo /zognaqbhn ngwnaong lnoj). Bidr lzaara-gbatng hbo wbmn b dunhzbi
'huj' mbijn, pwzhw lnbod gwbg 'dozqqnk' ubhfngd dwrjik en dnog rjg
rq dpzghw hwzud huj urag. Urag lzaarazot wbuunod zoknunoknogiv
rq dpzghwzot tarjud gwbg wbmn ra wbmn org enno dng ju.
```

After examining the text, spaces and special characters were preserved in it. They are meaningful and are in the same way as in a regular text. From this I can conclude that the words are separated by a space. The encryption of this text is determined by the permutation of characters.
It remains only to understand - encryption has one alphabet for encryption, or several

I'll start with the simplest - I'll check the text using `frequency analysis` (figure 7)


<center>

![](https://i.imgur.com/DB9kF5P.png)
Figure 7: Frequency analysis of characters in encrypted text
</center>

This distribution makes it clear to me that one alphabet was used to encrypt the text (For example, I will also give encryption with multiple alphabets (figure 8))

<center>

![](https://i.imgur.com/3otm3qK.png)
Figure 8: Frequency distribution when using multiple alphabets when encrypting a message
</center>

The solution to the problem of hacking this message will be the selection of the correct text encryption alphabet.

I'll start with the most common symbols

In the `13%` case, the letter `E` occurs in English
In my cipher, the most common characters have `11%`
These are the characters` 'G` and `N`
To determine which of these characters is best suited, I decided to go by determining the vowel letter

Since the letter `E` is a vowel, I will look for the vowel with the highest frequency of neighborhood (since they can be next to almost every letter in the alphabet)

The letter with the largest number of neighbors is `N`. Since I have a choice between `G` and `N`, I will choose `N`

<center>

![](https://i.imgur.com/WydlDWO.png)
Figure 9: Table of the frequency distribution of neighbors for each letter in the cipher
</center>


Now that I have defined the letter `E`, I want to define `H`, because the combination `HE` is often found
for this I will refer to the table "Letter Pairs Help Tool" (figure 10)
<center>

![](https://i.imgur.com/RLVI9Xt.png)
Figure 10: The "Letter Pairs Help Tool" table prepared to search for the letter `H` in the cipher
</center>

In this table, you need to find the c column with the largest or smallest value, this column will denote the letter `H`
Since the largest number `11` occurs more than 1 time, I am looking for the column with the smallest. The minimum value is `-11` and it belongs to column W - hence this letter stands for `H`
I'll make a guess and mark it in the selection alphabet

<center>

![](https://i.imgur.com/HTwDdl4.png)
Figure 11: Intermediate state of the table of correlation of the alphabets of the cipher and the English language
</center>

It's time to find out what `T` looks like in encrypted form
`TH` combination is also a common one and you just need to see with whom the most `W`

<center>

![](https://i.imgur.com/cfexZiu.png)
Figure 12: The "Letter Pairs Help Tool" Table
</center>

It is clearly seen that the letter G has the most combinations with `W`, and after checking the Asymmetry table (figure 10) , it fits the role of the letter `T`
I'll make a guess and mark it in the selection alphabet

<center>
![](https://i.imgur.com/czU7Nm3.png)
Figure 13: Intermediate state of the table of correlation of the alphabets of the cipher and the English language
</center>


Next, I was interested in the word `th*t` in the decryption
It occurs several times in the source text, it will probably help in further decoding.
After studying all occurrences of `th*t`, it turned out that these words are the same in the cipher and look like `gwbg`. This word is the word `that` and `A` of the original alphabet fits in place of `B`.
I will note this in the selection alphabet

<center>
![](https://i.imgur.com/73cpOUu.png)
Figure 14: Intermediate state of the table of correlation of the alphabets of the cipher and the English language
</center>

Continuing further to distinguish words from the total mass, it turned out to decipher the text to the end, it looks like this:
```
Port mirroring lets switch 'sniff' all traffic that is going in and 
out of one port (mirror-source) and send a copy of those packets out 
of some other port (mirror-target). this feature can be used to easily
set up a 'tap' device that receives all traffic that goes in/out of
some specific port. note that mirror-source and mirror-target ports
have to belong to same switch. (see which port belong to which switch
in /interface ethernet menu). also mirror-target can have a special
'cpu' value, which means that 'sniffed' packets should be sent out
of switch chips cpu port. port mirroring happens independently of 
switching groups that have or have not been set up.
```

<center>

![](https://i.imgur.com/VSCwFyX.png)
Figure 15: The resulting alphabet, which is used to decipher the text received from the partner
</center>

:::

