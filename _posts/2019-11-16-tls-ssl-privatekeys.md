---
layout: post
title: "TLS/SSL Certificates - The Private Key"
#description: ""
#date: 2019-07-04 00:39:00 +0800
tags: security
---

There is already a ton of information out there about this topic. 
But I noticed that there is no one site that provides exactly what I usually need. 
Some provide only a specific use-case and miss out on others. 
I often have to do multiple searches. Although searching Google is fast and hits are often accurate, 
it can still be an exhausting task. It's about time I create my own documentation.

# There are many ways to create a private key

TLS/SSL Certificates require a private key. It's the very first thing that is required to produce a certificate.

### Create an RSA private key and save it to a file

```
$ openssl genrsa -out privatekey.pem 2048
```
This will create a private key and save it to a file named *privatekey.pem*. 
The size of the created key is 2048 bits.

Sometimes you don't want to save the private key to a file and instead, you want to display it or send it to the screen.
For example, you need to cut-and-paste the key to a website or GUI application. 
You want to save the step of doing a cat command of the file and
avoid the step of removing it because for security reasons, you don't want it saved on your hard drive.

### Create an RSA private key and and display it on the screen

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

The command above will create a private key and send it to STDOUT.

At some point, you will need to save your private key to a file so that it can be used later.
I don't think anyone can memorize an RSA key. As you can see above, 
it is composed of a lot random characters that don't make any sense.

So if you need to save it to a file, clearly, it is not desirable to store it in the clear. 
It would be like saving a password to a file in clear text.

The solution to this problem is to store the private key in encrypted form. 
The key (passphrase) to encrypt the RSA key will then be smaller, hence, easier to memorize by a human.

### Create an encrypted RSA private key

```
$ openssl genrsa -aes256 2048
```

This command will create a private key and encrypt it. 
The password or passphrase to encrypt the key will be prompted.

Encryption will be done using the AES-256 algorithm. Other types of algorithm are available.
Run *openssl genrsa --help* to view other types.

By using a passphrase, it will be smaller in size and easier to memorize by a human.

However, although prompting for the passphrase is a secure way of supplying the passphrase, there are times
when this is not possible. For example, you need to do this programmatically or there is no human to supply the passphrase.

### Create an encrypted private key without prompting for passphrase

```
$ echo -n "secret" | openssl genrsa -aes256 -passout stdin 2048
```

Instead of prompting for the passphrase, the passphrase can be supplied in the command line through STDIN of openssl.
The passphrase used to encrypt the created key above is the word *secret*.

This command may not be a good idea to use in practice because the passphrase is displayed in clear text.
But this is just a demonstration because even with STDIN, you can still supply the passphrase in a secured way
and will not display to the screen or appear in the command in clear text.

For example, you can get the passphrase from a file and then pass the contents to the STDIN of the openssl command.

```
$ cat ~/.ssh/passphrase.txt | openssl genrsa -aes256 -passout stdin 2048
```

Instead of a file, you can use a command or application to derive the passphrase or retrieve it from a secured source.

```
$ getpassphrase | openssl genrsa -aes256 -passout stdin 2048
```

The *getpassphrase* command will send the passphrase to STDOUT and through the STDIN of openssl.

Supplying the passphrase in this manner provides better security since the 
passphrase is never displayed on the terminal or is never present in the filesystem.

However, eventually, to use a private key, you'll need it in unencrypted form.

### Decrypt an encrypted RSA private key

```
$ openssl rsa -in privatekey_e.pem
```

This command decrypts an encrypted key stored in *privatekey_e.pem* file. 
It will prompt for the passphrase to decrypt the key.
If the passphrase is correct, the decrypted key will be displayed (sent to STDOUT).

### Decrypt an encrypted RSA private key without prompting for the passphrase

Instead of prompting for the passphrase, you can supply the passphrase through STDIN of openssl.
This is achieved by using the *-passin stdin* option of openssl.

```
$ echo "secret" | openssl rsa -passin stdin -in privatekey_e.pem
```

```
$ cat ~/.ssh/passphrase.txt | openssl rsa -passin stdin -in privatekey_e.pem
```

```
$ getpassphrase | openssl rsa -passin stdin -in privatekey_e.pem
```

What if when you first created a private key you don't need it encrypted, 
but later on decided you want it encrypted?

### Encrypt an RSA private key

```
$ openssl rsa -aes256 -in privatekey.pem
```

This command will encrypt the private key stored in *privatekey.pem* file. 
It will prompt for the passphrase to use.

### Encrypt an RSA private key without prompting for the passphrase

```
$ echo "secret" | openssl rsa -aes256 -passout stdin -in privatekey.pem
```

```
$ cat ~/.ssh/passphrase.txt | openssl rsa -aes256 -passout stdin -in privatekey.pem
```

```
$ getpassphrase | openssl rsa -aes256 -passout stdin -in privatekey.pem
```

The commands above will encrypt the key in *privatekey.pem* file using a passphrase supplied through the STDIN of openssl.
The output (STDOUT) of the openssl command will be the encrypted key.

To save the encrypted key to a file, redirect the output to a file or use the -out option.

```
$ openssl rsa -aes256 -in privatekey.pem > privatekey_e.pem
```

```
$ openssl rsa -aes256 -in privatekey.pem -out privatekey_e.pem
```

