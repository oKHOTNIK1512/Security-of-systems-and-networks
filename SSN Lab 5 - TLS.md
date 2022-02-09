---
tags: Security of systems and networks
---
:::success
# SSN Lab 5 - TLS
Name: Ivan Okhotnikov
:::


## Task 1 - TLS handshake
:::warning
Inspect TLS handshake:
1. Using openssl , establish a TLS connection to facebook.com. Look at the parameters of your connection and briefly describe them in your report.

3. Establish the connection again and intercept TLS handshake using tcpdump/wireshark. Describe all packets involved in the TLS connection in your report.

5. What version of TLS is used? Describe which versions are not safe to use and under which conditions.

7. What ciphersuite is used? Make a small list of ciphersuite that you would recommend to use and why
:::

## Implementation:

:::info
> 1. Using openssl , establish a TLS connection to facebook.com . Look at the parameters of your connection and briefly describe them in your report.

I can establish a connection using the command:
```
openssl s_client -connect www.facebook.com:443
```

I can see the certificate chain in the response (figure 1):
<center>
    
![](https://i.imgur.com/3Xtn3Ok.png)
Figure 1: Part of the openssl program output
</center>

Server Certificate (figure 2):
<center>
    
![](https://i.imgur.com/95yrVUx.png)
Figure 2: Part of the openssl program output
</center>


Information about who issued the certificate and to whom it was issued (figure 3):

<center>
    
![](https://i.imgur.com/Loyrvzq.png)
Figure 3: Part of the openssl program output
</center>

Signature type (figure 4): 

<center>
    
![](https://i.imgur.com/T7Q3ykP.png)
Figure 4: Part of the openssl program output
</center>

Signature verification (figure 5):

<center>
    
![](https://i.imgur.com/GprRjdN.png)
Figure 5: Part of the openssl program output
</center>

What version of TLS is used and information about it (figure 6)
<center>
    
![](https://i.imgur.com/bWQLvTP.png)
Figure 6: Part of the openssl program output
</center>


And also `handshake` (figure 7)
<center>
    
![](https://i.imgur.com/D8v8KlL.png)
Figure 7: Part of the openssl program output
</center>
:::

:::info
> 2. Establish the connection again and intercept TLS handshake using tcpdump/wireshark. Describe all packets involved in the TLS connection in your report.

I intercepted the handshake using `Wireshark` (figure 8)
<center>
    
![](https://i.imgur.com/OHdWw0L.png)
Figure 8: Intercepted traffic in the `wireshark` program
</center>

From the captured traffic, it is clearly visible that the availability of the host is first checked and an attempt to establish a TLS connection (`Hello message` from the client)
is then sent by the server to its `Hello message` in response
These messages contain information about the handshake first from the user side, then from the server side. Once established, the information will already be transmitted in encrypted form.

:::

:::info
> 3. What version of TLS is used? Describe which versions are not safe to use and under which conditions.

`TLSv1 is used.3`

A good example of an unreliable version is `TLSv1.0` and some versions of `TLSv1.1`
**TLSv1** is the very first version. It was not protected from `cipher-block chaining` (CBC) attacks. Protection was added in the `TLSv1.1` version
They were replaced by **TLSv1.2**, in which:
- replaced in the pseudo-random combination function `sha1` and `md5` with `sha-256` 
- in the hash of the final message, the hash is generated using 'sha256`
- now more authentication ciphers are used


:::

:::info
> 4. What ciphersuite is used? Make a small list of ciphersuite that you would recommend to use and why?

A set is used:
```
TLS13-AES-256-GCM-SHA384
TLS13-CHACHA20-POLY1305-SHA256
TLS13-AES-128-GCM-SHA256
TLS13-AES-128-CCM-8-SHA256
TLS13-AES-128-CCM-SHA256
```

I would choose `TLS13-AES-256-GSM-SHA384` and `TLS-13-CHACHA20-POLY1305-SHA256` as they are more reliable

:::


## Task 2 - Man In The Middle

:::warning
1. Setup a TLS proxy - mitmproxy
a. There are several modes it can operate. You should use transparent mode.
b. Describe setup steps and show that are able to read transmitted data.

2. Intercept TLS handshake before proxy and after proxy and describe how MITM is performed.
a. When do you think TLS proxy is used for security purposes in the industry?
b. What are techniques that are used to protect applications (desktop, web, mobile, etc.) from TLS proxying?

3. Proxy allows you not only to read data, but also modify it on the fly. Think of a scenario when it can be dangerous and try to demonstrate that.
:::

## Implementation:

:::info
>1. Setup a TLS proxy - mitmproxy
>>a. There are several modes it can operate. You should use transparent mode.
>>b. Describe setup steps and show that are able to read transmitted data.

My client is inside gns3, I have Internet access through the `virbr0` interface
On my workstation, I downloaded mitmproxy and configured traffic redirection so that the client has a connection through my proxy.
I did it using the `iptables` rules:

For `HTTP`
```
sudo iptables -t nat -A PREROUTING -i virbr0 -p tcp --dport 80 -j REDIRECT --to-port 8080
```

For `HTTPS`
```
sudo iptables -t nat -A PREROUTING -i virbr0 -p tcp --dport 443 -j REDIRECT --to-port 8080
```

It remains only to launch
```
mitmproxy --mode transparent
```

To make sure that I can view the content, I will download the page `https://volsu.ru `
(figure 9)


<center>

![](https://i.imgur.com/xWcXAdc.png)
Figure 9: The `mitmproxy` program window
</center>
:::

:::info
>2. Intercept TLS handshake before proxy and after proxy and describe how MITM is performed.


<center>

![](https://i.imgur.com/OHdWw0L.png)
Figure 10: Before proxy
![](https://i.imgur.com/3e2HEVf.png)
![](https://i.imgur.com/MEET8fW.png)
Figure 11: After proxy
</center>

It is clearly visible from the requests that another TLS header has been added to the Server Hello response, which also contains encrypted data

MITM proceeds as follows:
- the request is redirected to the proxy server
- the proxy server reads the information from the request
- the proxy redirects the request to the requested server
- the proxy server receives a response from the server and gives it to the client

>>a. When do you think TLS proxy is used for security purposes in the industry?

Since TLS provides the ability to encrypt traffic over all protocols that are used for the connection, this can be applied when the real connection is not secure (for example, I am connected to a public wifi access point).

>>b. What are techniques that are used to protect applications (desktop, web, mobile, etc.) from TLS proxying?

A system called `SSL Pinning` is used, which is the embedding of a certificate that is used on the server into a client application. So when requests are made to the server, its certificate will be checked for compliance with the certificate that was embedded in the application and if they differ, the request will not be processed.
:::

:::info
>3. Proxy allows you not only to read data, but also modify it on the fly. Think of a scenario when it can be dangerous and try to demonstrate that.

This can be potentially dangerous when a customer makes a money transfer. If you replace the account number, the money will go to the attacker. You can also substitute a message on a social network (and quarrel with people, for example)
With your permission, I will use `burp` to substitute a message on a social network

This program was used earlier in the first laboratory work on INR

For example, I will send the message `Hello!`
Since I have traffic interception enabled in the program, the message will not be delivered immediately (figure 12)

<center>
    
![](https://i.imgur.com/mHhPOHi.png)
Figure 12: Sending the original message
</center>

Now I will move on to the program, where it kindly provides me with a window asking me to change it (figure 13)

Because `vk.com` sends the message in its pure form, I can replace it with my own text (for example, `Buy!`) (figure 14) and press the `Forward` button in the program
<center>
    
![](https://i.imgur.com/k9clrJX.png)
Figure 13: Burp program window for request modification
![](https://i.imgur.com/SzmNUBP.png)
Figure 14: A spoofed message in a request to the server
</center>

You can make sure that the message with the new text has been successfully delivered (figure 15)
<center>
    
![](https://i.imgur.com/HN9nePr.png)
Figure 15: A spoofed message sent
</center>
:::

## Task 3 - x509

:::warning
By default, mitmproxy uses its own certificate authority to issue certificates and if you
inspect the certificate chain (f.e in browser) you will see that it looks suspicious.
Here you will try to create a certificate chain that looks almost the same as the original
certificate chain for a website and, alongside, get familiar with common attributes that are
used in x509 certificates.
1. Obtain a cert chain for www.facebook.com (f.e. using openssl):
a. Does the cert chain contain the root cert? Where are trusted certs stored?
b. Extract certs and place: root CA in root.crt, intermediate CA in int_ca.crt, and server cert in server.crt.

2. Using the contents of obtained certificates, create your own chain that mimics the original one, they should look almost identical.
a. Document your steps and describe the meaning of certificates attributes.
b. Include created certificate chain into the report as appendix - both in base64 and human readable form.

3. BONUS: Provide a server key and certificate to mitmproxy and try to visit
www.facebook.com using the browser and check how the certificate chain is looking
now (do not forget to add the created root CA certificate to the trusted store).
:::

## Implementation:

:::info
>1. Obtain a cert chain for www.facebook.com (f.e. using openssl):

<center>
    
![](https://i.imgur.com/gRQpDph.png)
Figure 16: Certificate Chain
</center>

>>a. Does the cert chain contain the root cert? Where are trusted certs stored?

Certificates are stored in trusted certificate authorities. In the case of Facebook - in the Digitcert certification center `www.digitcert.com`

>>b. Extract certs and place: root CA in root.crt, intermediate CA in int_ca.crt, and server cert in server.crt

You can find out the contents of the certificates by simply adding the `-showcerts` flag to the ready-made command (figure 17)

This command outputs the server certificate and the intermediate certificate.

```
openssl s_client -connect www.facebook.com:443 -showcerts
```

<center>
    
![](https://i.imgur.com/wXXvePq.png)
Figure 17: Information about the server certificate
![](https://i.imgur.com/i7ga7CO.png)
Figure 18: Information about the intermediate certificate
</center>

Since there is no root certificate in the chain, you will need to download it from the official website to get it:

https://cacerts.digicert.com/DigiCertHighAssuranceEVRootCA.crt.pem
:::

:::info
>2. Using the contents of obtained certificates, create your own chain that mimics the original one, they should look almost identical.

>>a. Document your steps and describe the meaning of certificates attributes.
>>b. Include created certificate chain into the report as appendix - both in base64 and human readable form.


### Root certificate

To begin with, I will create a key for the root certificate with a size of 4096
```
openssl genrsa -out private/root.key 2048
```

After that I create a certificate based on the previously created key

```
openssl req -x509 -sha1 -new -nodes -key private/root.key -days 10000 -out certs/root.pem
```

<center>
    
![](https://i.imgur.com/CVEmkyC.png)
Figure 19: Creating a root certificate
</center> 


### Intermediate certificate
Next in the chain is an intermediate key. I will create its key and a certificate creation request, which I will sign using the root certificate in the future



```
openssl genrsa -out private/int.key 2048
openssl req -new -key private/int.key -out csr/int.csr -config configs/config_int.cnf
```

To create it, I used a configuration file with the fields `distinguished_name` filled in, so as not to enter them manually

```
# configs/config_int.cnf
[req]
prompt = no
distinguished_name = dn

[dn]
C=US
O=DigiCert Inc
OU=www.digicert.com
CN = DigiCert SHA2 High Assurance Server CA
```

It remains only to sign with the `root` certificate
```
openssl x509 -req -in csr/int.csr  -extfile external/v3_int.ext -CA certs/root.pem -CAkey private/root.key -CAcreateserial -out certs/int.pem -days 5000 -sha256 
```

Since browsers now require v3 certificate support, I will add dependencies to the `external/v3_int.ext` file
In this configuration, you must specify that this certificate will be used further to sign the next certificate.

```
# external/v3_int.ext
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer:always
basicConstraints = critical,CA:true
```

Next, I will start creating a server certificate (it will already be signed with an intermediate certificate)
### Server certificate

It is created in the same way as an intermediate certificate, the only difference is in the signature.
I also created a configuration file with the `distinguished_name` filled in for convenience.

```
openssl genrsa -out private/facebook.key 4096
openssl req -new -key private/facebook.key -out csr/facebook.csr  -config configs/config_server.cnf
```

```
[req]
prompt = no
distinguished_name = dn

[dn]
C=RU
ST=Kazan
L=Innopolis
O=Innopolis
OU=Innopolis   
emailAddress=i.okhotnikov@innopolis.university
CN = *.facebook.com
```

It's time to sign it. I filled in the `extfile` file for it a little differently
```
openssl x509 -req -in csr/facebook.csr -extfile external/v3_server.ext -CA certs/int.pem -CAkey private/int.key -CAcreateserial -out certs/facebook.pem -days 5000 -sha256 
```

In it, I indicated that it is the final one (it will not be used further to sign other certificates) and other certificates do not come from it
Also an important difference is the indication of dns records for which it will work. I specified the domain addresses there, which are specified in the server certificate `facebook.com`

```
# external/v3_server.ext
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
subjectAltName = @alt_names

[alt_names]
DNS.1 = *.facebook.com
DNS.2 = *.facebook.net
DNS.3 = *.fbcdn.net        
DNS.4 = *.fbsbx.com
DNS.5 = *.m.facebook.com
DNS.6 = *.messenger.com
DNS.7 = *.xx.fbcdn.net
DNS.8 = *.xy.fbcdn.net
DNS.9 = *.xz.fbcdn.net
DNS.10 = facebook.com
DNS.11 = messenger.com
```

This completes the creation of the chain.


Now I will assemble the chain of certificates into one file `cert/chain.pem`, which will include:
- Server Certificate
- Intermediate certificate
- Root certificate
- Server root key

```
cat certs/facebook.pem certs/int.pem certs/root.pem  private/facebook.key > certs/chain.pem
```
:::


:::info
>3. BONUS: Provide a server key and certificate to mitmproxy and try to visit www.facebook.com using the browser and check how the certificate chain is looking now (do not forget to add the created root CA certificate to the trusted store).

Now that I have a certificate chain ready, I will run mitmproxy using these chains:
```
mitmproxy --certs *=certs/chain.pem
```

At the moment, the browser does not perceive the mobile chain and it is necessary to add the root certificate to the trusted list (figure 20-22)

<center>
    
![](https://i.imgur.com/wsre1HY.png)
Figure 20: Initial browser response when opening the browser
![](https://i.imgur.com/PBVmjyl.png)
Figure 21: Window for adding a new certificate
![](https://i.imgur.com/ka3JRYc.png)
Figure 22: The window for confirming the trust in the new certificate
</center>

After adding the certificate to trusted - the page will open successfully (figure 23)

<center>
    
![](https://i.imgur.com/KayHwl8.png)
Figure 23: Successful opening of the page facebook.com using my certificate chain
</center>

If you look at the certificate used, the chain
that I created earlier will be displayed (figure 24)

<center>
    
![](https://i.imgur.com/IaKkiVI.png)
Figure 24: Certificate Trust Chain Output
</center>
:::


## References:
1. [Your Certification Authority](https://habr.com/ru/post/192446/)
2. [SSL Certificates](https://curl.se/docs/sslcerts.html)
3. [x509 Certificates](https://wiki.mozilla.org/SecurityEngineering/x509Certs)
4. [Mitmproxy ca certificate](https://docs.mitmproxy.org/stable/concepts-certificates/#installing-the-mitmproxy-ca-certificate-manually)