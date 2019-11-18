---
layout: post
title: "How to find the SSH key to your EC2 instance"
description: "I've got so many ssh keys that I can't remember anymore which one is for..."
#date: 2019-07-04 00:39:00 +0800
tags: aws
---

Like passwords, I've got so many ssh keys that I can't remember anymore which one is for what.
Recently, I needed to connect to an EC2 instance but I forgot the ssh key to access it.
I did not lose the key but I did not know which key, among the dozens I have, is used for this instance.
I know, I should use a password manager to save this information. I do have a password manager but I don't save this information. :(

Of course, I could try each one and see which one works. But sometimes this is not possible, 
or it could be too cumbersome if your keys use a passphrase.

Fortunately, Amazon shows the fingerprint of the keypair (public key) used by your instance.
This serves like a password hint or a clue to find out what key you used. Pretty cool.

Amazon documents how it computes the fingerprint of a keypair:
[https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#verify-key-pair-fingerprints](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#verify-key-pair-fingerprints)

All you need to do is get the fingerprint of all your keys and then compare. 
Depending on how you created or uploaded your public key, the fingerprint is derived in many ways.
Here's a summary of the 3 ways Amazon computes it:

```
openssl rsa -in path_to_private_key -pubout -outform DER | openssl md5 -c
```

```
openssl pkcs8 -in path_to_private_key -inform PEM -outform DER -topk8 -nocrypt | openssl sha1 -c
```

```
ssh-keygen -ef path_to_private_key -m PEM | openssl rsa -RSAPublicKey_in -outform DER | openssl md5 -c
```

It's probably still best to invest time and effort in using a password or key manager and make sure to also include the fingerprint information.
