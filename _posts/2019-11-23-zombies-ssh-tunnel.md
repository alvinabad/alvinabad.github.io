---
layout: post
title: "Zombies and SSH"
description: 
tags: ssh
---

If you have watched a zombie movie, oftentimes there is a scene where the heroine
is being chased by zombies and to get to safety, she needs to get into a room.

However, access to this room is controlled by her friends and they set it up in such a way 
that this room is only accessible 
through another room. Before they can let her in, she must close the door behind her
and make sure no zombies have entered with her. If a zombie managed to enter with her
before closing the door, she won't be allowed in.
Her friends have no choice but to let the zombies feast on her. Scary.

That's pretty neat security. You go through a series
of doors and locks before gaining access to your final destination.
In the computing world, that kind of security is also widely used. To get to a server, 
you'll first need to log in to one server and from there, log in again to the final server.
Security engineers often call this, *jump server*. 
You need to jump through hoops before you can reach your destination.
This security works well because there are many layers of authentication before one can access the destination.
It works fine, except for the zombies part.

What has SSH got to do with zombies?

Not a long time ago, a friend sent me this email:

>
Hey Alvin,
>
I need help! I have a server named HOST_4. It runs a website using this URL: http://HOST4:8080/.
My laptop, let's call it HOST1, has a browser running but it cannot access this website. 
I was told HOST4 is blocked by a network firewall.
>
However, I was told that I can access the website using a server named HOST3.
HOST3 is not restricted.
I have access to HOST3 all right but that server has no remote display or desktop to run a browser.
HOST3 is not a desktop so there is no console.
My only access to HOST3 is SSH. But SSH a freaking terminal! How am I supposed to browse a website using a terminal?
>
And not only that, HOST3 is only accessible using another server, named HOST2. Again, access to HOST2
is only available using SSH.
So to get to HOST3, I will need to log in to HOST2 first.
>
Please help!
>

<br>
Sensing urgency, I replied immediately. He might do something not secure and let the zombies in.

>
>
Dear friend,
>
It's good that you have a secured environment. You must have a lot of zombies.
I'm glad you are still not among the walking dead. LOL
>
Fear not, there is a solution to your problem
and you can still keep the zombies away. All you have to do is use SSH tunneling.
>
Run this command from your laptop, HOST1:
>
```
$ ssh -t -L 127.0.0.1:8080:127.0.0.1:8080 HOST2 \
  ssh -t -L 127.0.0.1:8080:HOST3:8080 -N HOST3
```
>
After running this, point your browser to this URL:
```
http://127.0.0.1:8080
```

This is called SSH tunneling. If your only access is SSH, which is port 22,
you can still access other ports of your destination, even if 
you go through many layers of servers.

There is no mention of HOST4 in 
the ssh command above because the effect of running this command
is like you are on HOST3, which has access to HOST4.

---
