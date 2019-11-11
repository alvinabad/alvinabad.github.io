---
layout: post
title: "How to TLS/SSL Certificates"
#description: ""
#date: 2019-07-04 00:39:00 +0800
tags: security
published: false
---

I know that there is already a ton of information out there about this topic. 
But I noticed that there is no one site that provides what I usually need. 
Some provide shortcuts or do not have exactly what I need so I always end up doing multiple searches. 
I do this often so this has become very exhausting for me.

For example, if I need to create a self-signed certificate, most documents out there will give you a
one-liner command to do everything, that is, it will create a new private key and produce the certificate.
But most of the time, I already have the private key or the private key already exists so I'll need to use
that private key to create a new CSR or self-signed certificate. 
I prefer that all steps are done separately so that I can be more flexible if I need to do things differently.

Other times, I need to do things programatically so I need to run the commands in a script instead of going 
through interactive prompts.

## The many ways to create a private key

TLS/SSL Certificates require a private key. It's the very first thing that is required to produce a certificate.
Here's the command to create an RSA private key using the openssl command:
```
$ openssl genrsa -out privatekey.pem 2048
```
This will create a private key and save it to a file named privatekey.pem. 
The size will be 2048 bytes.

But sometimes you don't want to save the private key to a file and instead you want to send it to the screen.
For example, you need to cut-and-paste the key to a website to save it. 
You want to save the step of doing a cat command of the file and
also want to avoid the step of removing it because for security reasons, you don't want it to be saved to your disk.
To do this, simply run the same command without the -out option:
```
$ openssl genrsa 2048

Generating RSA private key, 2048 bit long modulus
..................................+++
..........................................................................+++
e is 65537 (0x10001)
-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEA1IPN3KIvxiCeoemivcwsTNkpcQMGjzeuh34EJNmu6QTKTacI
6UUlavgYSA1MjVJqW7WZ7B/RlHUSeoIQ6FSU6YI2zuWjhShd2oaratBZ/GfoxwSo
nZZrjl1Y/0G+Y3o9BsOIaibypfyJejEhEBl6DFyO7PJWwi39ChoUZJj7rI9j/Y3Y
hSyl+a5Gl78vhIUdhWH6Op9oLQlKs2PeMZ5lW+JMuvcEspudBfTB9QhkmJYcZylS
rg7qtKfQFrsIp9ISo2UuHviYPsGtaQa2HTM0wBdhbVtiPL9mAPZTFd2OUy+GfsLF
dbto8Jnf1AJH6CfVCoQ4kJ039ebds/5ouZshSQIDAQABAoIBAQCBS4O3RdHtKDF7
bws9kHgvdTxabq3R+t2rv8bDqNFbIjf3ULYCPriKQVF8nOLDZK4jB/UTDTTUrvnE
IVgqEjPbcgbokByRykJ3ztGCFit5frrIQxRXdIoRvseD3br5CQkSEWrwsquUw3Xd
KwDjq6xu0u1+Sd7xG0vFlqJrpgwRKryiNde6i8lClBTF3N1pFqlVzKsPO6CeT+LQ
kcTR0I4pHtZTZ1SGeuNBoUGh43UEPPuz3KCjOfGbHkGev5d47b96+pie1nqpXotO
URh9kOyKrNoA1ohKtgX9WWztv9b+mvHmcoqD6RB0WjgbSp+HDsq8pJ6GG/nkSJW3
1cXccAWVAoGBAOw+8ksezDuNGmtNPKb/sIpnufQyvBQB4WPpomKPTXNVgf562Wjr
w7SxwY7lTmxUoDZOIvJggmb4EQTLSbTCtw0GfMvDQIQAmGAVLJmS4Yu3Z43kJWEZ
7N9t4lurvVKhXSB/p2gUZ3B9vgck5dctkaanIIIskWh0qw/s/8ga6Le3AoGBAOZI
37y6fkJIE1MddDbM7krBUX83WXijAiR1m0KIqG4C02Ng3kWY3UbTxQ5a6+HpYbrE
f5ikipbtQ0rSoQKvw3oz0yeSCO2CdccAS2aBXCGZNqz7ysh+a4M1NZTxqWZhgnln
HucTRYxIIhoL85uc1sWHGO7kVcX89tYGWrD3AO7/AoGBAIbsSEq40yFrq9v+Z5Zk
zzmsldouo5W1oTkDtPcfKrek7TIejU5L4CCxqH7o3UonZry9EV5l2fPe1zFqvLSc
xBiPTPS+lXkOMfgl/6vR5Dh8TYsO8n4rZUoRosaCJgUsHVizLzPU+2KWndHMs+uZ
neaU6o8NzxneD9hvnYF2RNSvAoGBAJjgGf/dQoJX/NQ5gnP62dqwuZydrvziIkL3
/ClQNZhKsfCQVx6W25bwcSoVe/COkX1+g0JfQU3ulrSuXYs+CaQvkWb8kIa0C+If
NO8Iw5Pedaiiwz0Uh+Ujxr1pLM81vns/1QkMByiYnmLyOje9B+s7w3acCMEWLPtq
XnyKjYkbAoGAdWDNytbn/5Pp3VIn65+a9s943YeeDCCo+HyxSaeNJE2Suc7aDMo1
XVEa+Ev6wYsDel428EaBFRlMx88fMQ/GdGWEutjHfaw+Yw7pGUWxdDTYCDCShTDq
qQXHVQLCUrt8X856YwdpKb6INk1KNDYHDNNRKgmY/jLIrJ6DxJrdNSg=
-----END RSA PRIVATE KEY-----
```

The commands above will create a private key that is not encrypted. 
But what if you want to create private key that is encrypted? 
Creating a private key and saving it to a file on your disk is like saving a password to a file in clear text.

To create a private key that is encrypted, use the -aes option:
```
$ openssl genrsa -aes256 2048
```
This command will encrypt the private key using AES256. Other types of encryption are available.
Run openssl genrsa --help to view other types.

This command will prompt you for the passphrase that will be used to encrypt the key.

While prompting for the passphrase is a secure way of supplying the passphrase, there are times
when this is not an option. For example, you need to do this programatically.

To create a private key without being prompted for the passphrase, run this command:
```
$ echo -n "secret" | openssl genrsa -passout stdin 2048
```

Instead of prompting for the passphrase, the passphrase here is the word "secret" and supplied in the command line.

Of course it is not a good idea to use the echo command and display the passhrase in clear text.
This is just an illustration that the passphrase can supplied to the STDIN of the openssl command.

You can make this secure by getting the passphrase from a file:
```
$ cat ~/.ssh/passphrase.txt | openssl genrsa -passout stdin 2048
```

It could be better if you could have a separate program that generates or retrieves the passphrase from a secure source:
```
$ getpassphrase | openssl genrsa -passout stdin 2048
```
Where *getpassphrase* is a command or application that outputs the passphrase and feeds it to the openssl command.

Supplying the passphrase in this manner provides better security since the 
passphrase is never exposed to the terminal or is never present in the filesystem.


## How to create a Certificate Signing Request (CSR)

## How to view a CSR

## How to create a self-signed certificate

## How to create a CA Certificate
