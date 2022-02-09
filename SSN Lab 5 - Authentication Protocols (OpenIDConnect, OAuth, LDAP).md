---
tags: Security of systems and networks
---
:::success
# SSN Lab 5 - Authentication Protocols (OpenIDConnect, OAuth, LDAP)
Name: Ivan Okhotnikov
:::


## Task 1 - Requirements

## Implementation
:::info

List of requirements:
- Java (at least version 11)
- IntelliJ IDEA (sudo snapinstall intellij-idea-community --classic --edge)
- Docker for Keycloak (was installed earlier)
- Git (was installed earlier)
- Postman (sudo snap install postman)

By cloning the repository `https://github.com/andifalk/secure-oauth2-oidc-workshop ` I went to the folder of this directory
```
git clone https://github.com/andifalk/secure-oauth2-oidc-workshop.git
cd secure-oauth2-oidc-workshop
```

To run `Keycloak` in the docker container, I went to the `setup` folder and modified the sh script `run_keycloak_docker.sh ` to specify the root directory
<center>
    
![](https://i.imgur.com/1LxeVQK.png)
Figure 1: Contents of the `keycloak` startup script
</center>

After that, you can start the Keycloak server

```
./run_keycloak_docker.sh
```

<center>

![](https://i.imgur.com/ghmy13k.png)
Figure 2: Docker Container Launch window
</center>

After starting the container, you can go to the following address to make sure that everything works:

```
http://localhost:8080/auth/admin
```

After logging in with the username and password `admin`, the web interface of the administrator `Keycloak` will open

<center>
    
![](https://i.imgur.com/NnW14st.png)
Figure 3: Web admin interface `Keycloak`
</center>
:::

## Task 2 - Authorization Grant Flows in Action

## Implementation
:::info
Keycloak already has clients for authorization:
```
http://localhost:8080/auth/admin/master/console/#/realms/workshop/clients
```
From the main table you can find out their status, name and base url (figure 4)

<center>

![](https://i.imgur.com/IyW4bwc.png)
Figure 4: List of clients
</center>


By clicking on `edit` you can see the following window (figure 5)
<center>

![](https://i.imgur.com/Oeq5wJn.png)
Figure 5: Client Change Window
</center>

Here you can find a lot of settings
For example: Change its `client_id`, turn it off, change the `redirect_uri` endpoint and the main domain

By going to the `Credentials` tab, you can find out `client_secret` and generate a registration token for the user (figure 6), as well as select an authenticator

<center>

![](https://i.imgur.com/XnMCu2e.png)
Figure 6: The `Credentials` tab
</center>

### grant_type:client_credentials
For the client `library_client` using its `client_secret` and `grant_type:client_credentials` I will get a token using an `http request` in `postman`
 
To do this, I send a POST request to the endpoint:
```
http://localhost:8080/auth/realms/workshop/protocol/openid-connect/token
```
According to the openid specification:
`Content-Type` in the request header should be `application/x-www-form-urlencoded`

The request body will consist of:
```
grant_type:client_credentials
client_id:library-client
client_secret:9584640c-3804-4dcd-997b-93593cfb9ea7
scope:library_user email profile
```

After a request with these parameters, a token and additional information in the response (such as the token lifetime, scope, token type) will be output to me

<center>

![](https://i.imgur.com/xtIqtAp.png)
Figure 7: Query result in `Postman`
</center>

### grant_type:password

For the client `library_client` using its `client_secret` and `grant_type:password` I will get a token using an `http request` in `postman`
 
To do this, I send a POST request to the previous endpoint and specify `Content-Type:application/x-www-form-urlencoded` in the request header

The request body will differ from the previous one by the fields `username` and `password`, in which I entered data from one of the existing users:
```
grant_type:password
username:bwayne
password:wayne
client_id:library-client
client_secret:9584640c-3804-4dcd-997b-93593cfb9ea7
```

After a request with these parameters, a token and additional information in the response (such as the token lifetime, scope, token type) will be output to me

<center>

![](https://i.imgur.com/ez4E0uk.png)
Figure 8: Query result in `Postman`
</center>
 
 
### grant_type:code

This method assumes that we first log in to the server, and then, upon successful authorization, it will redirect to the service in which we want to log in.

To do this, you first need to form a link to authorization
I got it in the following form:
```
http://localhost:8080/auth/realms/workshop/protocol/openid-connect/auth?response_type=code&client_id=library-client&state=12321eafsdf&scope=library_user%20email%20profile&redirect_uri=http://localhost:9090/library-client/login/oauth2/code/keycloak
```
In the link, it is necessary to specify the type of response (I have this code)` 'client_id`, 'state` (but it is not required and is used for transmission when redirecting)` 'scope` (this field allows you to tell the server what fields we expect from it: for example, email and username), as well as 'redirect_uri` (it is very important that it matches the one that was specified when creating the client, since the server will return an error)

If the link was formed correctly, a window will open for entering the user's username and password (figure 9)


<center>

![](https://i.imgur.com/rvAQEPQ.png)
Figure 9: Authorization Window
</center>

After the server is sure that the data has been entered correctly, it will send us by `redirect_uri`, adding the authorization code to it:

```
http://localhost:9090/library-client/login/oauth2/code/keycloak?state=12321eafsdf&session_state=89f4d85f-73c9-4bc3-b6cc-47cfa5a041fb&code=c33a12da-af16-40a3-aa42-a3acca48de66.89f4d85f-73c9-4bc3-b6cc-47cfa5a041fb.3d97f958-fcf7-4ad6-a329-b2176729a0de
```
This code will need to be used in the request, as before, to get the token.

Request Body:

```
grant_type:authorization_code
client_id:library-client
client_secret:9584640c-3804-4dcd-997b-93593cfb9ea7
code:c33a12da-af16-40a3-aa42-a3acca48de66.89f4d85f-73c9-4bc3-b6cc-47cfa5a041fb.3d97f958-fcf7-4ad6-a329-b2176729a0de
redirect_uri:http://localhost:9090/library-client/login/oauth2/code/keycloak
```

After the request is executed, the token and additional information about it will be returned (figure 10)

<center>

![](https://i.imgur.com/4EAa7cZ.png)
Figure 10: The result of executing the request in `Postman`
</center>

You can get all the necessary endpoints and supported authorization types here:

```
http://localhost:8080/auth/realms/workshop/.well-known/openid-configuration
```

:::

## Task 3 - Authorization Code Grant Demo
## Implementation
:::info

For this task, I also need to enable the demo client application.
To do this, I opened a project in `IntelliJ IDEA` (figure 11)

<center>

![](https://i.imgur.com/1SARZCG.png)
Figure 11: The window of the `IntelliJ IDEA` program
</center>
In order to launch the client, I opened the client startup settings (figure 12) and added a task for the `Gradle` collector to launch `intro-labs:auth-code-demo:bootRun`, so I can easily start and stop the demo client using just one button
<center>

![](https://i.imgur.com/PgbbFyj.png)
Figure 12: Window for adding a collector
</center>

After the launch, the port `9095` will open and you can follow the link
```
http://localhost:9095/client
```

After opening the link, the main window will open (figure 13)

<center>
    
![](https://i.imgur.com/TsDqJ03.png)
Figure 13: Demo Client Window
</center>

Clicking on the authorization request button will be taken to the authorization page
<center>

![](https://i.imgur.com/72HKN2f.png)
Figure 14: Authorization page
</center>


After authorization, a redirect will be made to the demo client page with the authorization code (figure 14)
The page offers to get an authorization token using an authorization code. By clicking on the token request button
<center>

![](https://i.imgur.com/ovIsWUe.png)
Figure 15: The demo client's page for obtaining a token
</center>

After receiving the token, a page will open with the token you have already received (figure 16)
<center>

![](https://i.imgur.com/Z8T4NIe.png)
![](https://i.imgur.com/Phj0PWm.png)
Figure 16: The demo client window after receiving the token
</center>

Using this token, I can get the data of the user who was authorized by clicking on the `Token Introspection` button (figure 17)

<center>
    
![](https://i.imgur.com/JxrvLXt.png)
Figure 17: User information using the token
</center>

:::

## Task 4 - GitHub Client

## Implementation
:::info

Before you start, you will need to create an `Oauth application` in `Github` (figure 18)
<center>
    
![](https://i.imgur.com/3Ojxfoh.png)
Figure 18: Created Oauth application in Github
</center>

Then you need to fill in the Url of the home page, the endpoint of the redirect and generate `client_secret` (figure 19)
<center>

![](https://i.imgur.com/HyerZjv.png)

Figure 19: Setting up the application
</center>

Then I'll move on to configuring the client application

In the same project as before, opened in `IntelliJ IDEA`, I will correct the configuration file (figure 20)
`intro-labs/github-client/src/main/resources/application.yml`

In it I will specify the data `client-id` and `client_secret` that I received from `Github`
<center>

![](https://i.imgur.com/UBTT1hx.png)
Figure 20: Configuring the configuration file
</center>

And then I will configure the launch of the `intro-labs:github-client:boot Run` using Gradle, as I did when launching the demo application.

After the launch, the endpoint became active `http://localhost:9090`

After opening this endpoint, a window opens with a link to github authorization, which was formed from the `client_id` that I specified earlier (figure 21)
<center>

![](https://i.imgur.com/lLpHT6k.png)
Figure 21: Github Oauth Client Window
</center>

Since I was already logged in to Github earlier, when I clicked on the link, I didn't have to log in and was redirected back to the client. On the page, I was shown information about the user who was used for authorization
<center>

![](https://i.imgur.com/fcg5Jm5.png)
Figure 22: Page with user information
</center>


:::

## Task 5 - OpenLDAP

## Implementation
:::info

### Installing and configuring an LDAP server

In order for my `ldap-server` to work, you need to download the `slapd` `ldap-utils` packages
```
sudo apt-get install slapd ldap-utils
```

During the download process, I will be asked to enter the administrator password and confirm (figure 23, 24)

<center>

![](https://i.imgur.com/GijCKME.png)
Figure 23: Entering the LDAP Administrator password
![](https://i.imgur.com/WgChqwj.png)
Figure 24: Password Confirmation
</center>


Next I will need to reconfigure slapd
```
sudo dpkg-reconfigure slapd
```
There I will need to enter the domain name of my server (figure 25)

<center>

![](https://i.imgur.com/V3K2b4L.png)
Figure 25: Server domain name request during reconfiguration
</center>

Next, you will need to enter the name of the organization (I will leave it equal to the domain name)

<center>

![](https://i.imgur.com/1gQ51Px.png)
Figure 26: Entering the name of the organization
</center>

Next, I will also be asked for the administrator password - I will leave it the same as before


This completes the reconfiguration.

After that, I need to add uri and base information to the 
`/etc/ldap/ldap.conf`
configuration file
```
BASE     dc=ldap-server,dc=st11,dc=sne21,dc=ru
URI      ldap://192.168.122.240
```

### Installing and configuring phpldapadmin

To install it, run the command below:
```
sudo apt install phpldapadmin
```

After installation, you must first find out my ip address, for this I will use the command `ip addr` (figure 27)

<center>

![](https://i.imgur.com/FNeKGLv.png)
Figure 27: Output of the ip addr command
</center>

Then I will proceed to setting up the phpldapadmin configuration file:
```
sudo nano /etc/phpldapadmin/config.php
```

In it, I specify the name of the server (it will be displayed on the web page), the host address and information about the base (which I set up earlier in the configuration file)
```
$servers->setValue('server','name','st11 ldap server');
$servers->setValue('server','host','192.168.122.240');
$servers->setValue('server','base',array('dc=ldap-server,dc=st11,dc=sne21,dc=ru'));
```

After changing the configuration file, you can go to the phpldapadmin web page, but before that I will specify the host information in my `/etc/hosts` file:
```
192.168.122.240 ldap-server.st11.sne21.ru
```

Now I can go to the phpldapadmin web page at:
`http://ldap-server.st11.sne21.ru/phpldapadmin` (Figure 28)

To log in, I need to change the standard login DN to:
```
cn=admin,dc=ldap-server,dc=st11,dc=sne21,dc=ru
```
Where `cn` is the username
And also enter the user's password

After successfully logging in, I will be greeted by the main page (figure 28)

<center>

![](https://i.imgur.com/SExvGEb.png)
Figure 28: Home page after authorization
</center>
On it I need to open the contents on the left (figure 29) 

<center>

![](https://i.imgur.com/d7L7hDA.png)
figure 29: Content when revealing my main environment
</center>

First of all, I will create two organizational units (for convenience)
I'll call them `groups` and `users` (figure 30,31)

<center>
    
![](https://i.imgur.com/AhbKRS3.png)
Figure 30: Unit creation window for groups
![](https://i.imgur.com/DONxabK.png)
Figure 31: Unit creation window for users
</center>

After creating the unit, I will add the posix admin group to `groups`
To create it into groups in a unit, click on this unit (figure 32) and then click on `Create a child entry` to create a posix group

<center>

![](https://i.imgur.com/73f54Uf.png)
Figure 32: Window when clicking on groups unit
</center>

<center>

![](https://i.imgur.com/DE6K166.png)
Figure 33: Left navigation bar after creating posix group
</center>

Then I will create users in the same way in the unit `users` of users `iokhotnikov' and 'tokhotnikov`(figure 34б 35)

<center>
    
![](https://i.imgur.com/M3cEqkq.png)
Figure 34: User Creation Window
![](https://i.imgur.com/MqaDMpu.png)
Figure 35: Left navigation bar after creating users
</center>

It remains only to add users to the group to do this, I click on the created posix group `admin` and add the attribute `memberUid`, in which I specify the Uid of my created users (figure 36,37)

<center>

![](https://i.imgur.com/jlG3vk9.png)
Figure 36: Left navigation bar
![](https://i.imgur.com/i5g74XY.png)
Figure 37: A window with the created attribute `memberUid` and the listed users in the posix group `admin`
</center>

## Client

For the client, I first added `/etc/hosts' to the file
```
192.168.122.240 ldap-server.st11.sne21.ru
```
Then I installed the necessary packages for the client to work
```
sudo apt -y install libnss-ldap libpam-ldap ldap-utils nscd
```
During the installation process, I will be shown windows for configuring the connection to the LDAP server, in which I specified the server address, and the necessary paths to the database and the administrator (and his password)(figure 38-42)

```
ldap://ldap-server.st11.sne21.ru
dc=ldap-server,dc=st11,dc=sne21,dc=ru
cn=admin,dc=ldap-server,dc=st11,dc=sne21,dc=ru
```

<center>

![](https://i.imgur.com/kZMCl2p.png)
Figure 38: LDAP Server Connection Setup Window
![](https://i.imgur.com/D3rRx9y.png)
Figure 39: LDAP Server Connection Setup Window
![](https://i.imgur.com/yXnR4YN.png)
Figure 40: LDAP Server Connection Setup Window
![](https://i.imgur.com/ZnxewHi.png)
Figure 41: LDAP Server Connection Setup Window
![](https://i.imgur.com/ZbPs0BV.png)
Figure 42: LDAP Server Connection Setup Window
</center>

Then I will need to add lines to the end of the file `/etc/nsswitch.conf` that allow authorization via LDAP

```
sudo nano /etc/nsswitch.conf
```
```
passwd: compat systemd ldap
group:    compat systemd ldap
shadow: compat ldap
```


And also add to the end of the file `/etc/pam.d/common-session` information about the home directory during the installation of the session
```
sudo nano /etc/pam.d/common-session
```
```
session optional pam_mkhomedir.so skel=/etc/skel umask=0022
```

After all these manipulations, it will be necessary to restart the `nscd` service and authorization via LDAP will work

```
sudo service nscd restart
```

To check, I will use the command: `su - tokhotnikov` to verify authorization for the user (figure 43)

It can be seen that everything works and I can even execute the `pwd` command

In the same way, I configure the second client and log in under the second created LDAP user (figure 44)
<center>
    
![](https://i.imgur.com/Oe5bXk7.png)
Figure 43: The window for checking the connection and the possibility of authorization to the user on the first client
![](https://i.imgur.com/Bw9EVxE.png)
Figure 44: The window for checking the connection and the possibility of authorization to the user on the second client
</center>

:::