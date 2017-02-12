---
layout: default
title:  "Rubber-hose attacks"
date:   2016-10-17 18:30:29 -0300
categories: security
---
*"What I have to do, Agent Brody, is... unthinkable."* H. [Unthinkable (2011)](http://www.imdb.com/title/tt0914863/)

In an ideal world we should not have to worry about rubber hose attacks, nevertheless the world out there is not so pretty and such scenarios are far from fiction.

Rubber hose attacks are an euphemism for torture used to obtain a password. So let's imagine the following scenario:

You have an ultra secure system, you invested hundreds of thousands of dollars in state of the art firewall and security appliances. You perform pentests periodically with the best security consulting firms, you forbid pendrives, unencrypted laptop disks among other very strict rules. It seems that you are secure, at least an attacker would have to invest a great sum of money to breach you. 

Now let's imagine you have an adversary, it can be a competitor, or someone that has a strong interest in the data you have. That adversary learns that your system admin will go on vacation to a foreign country, Argentina for instance, and the adversary hire an asset to grab the sysadmin when he was going for some drinks, lock him in a room and start working on him.

Althought the reliability of the intel acquired under duress is questionable it's common sense that almost every human being has their breaking point. Defending someone's country is a reason to offer a strong resistance, on the other hand, very few people would be willing to suffer pain to protect their employer. And you as an employer don't expect it from them, so you have to create another layer of protection.

Here's some suggestions:

 1) Allow admin or sensitive data access from within your office.

 2) Revoke admin password of anyone that goes on vacation. That's valid if you believe rubber-hose attacks won't happen in US soil.

 3) Encrypt your sensitive passwords using [Shamir Secret Sharing](https://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing)

 4) Implement a duress password system.

The duress password system, if well implemented is very interesting because it permits the captured sysadmin to handover a valid password and by entering it valid data will be delivered.

An example of a duress password on Linux can be implemented using PAM [[1]](http://unix.stackexchange.com/a/107746).

```
auth    optional        pam_exec.so debug expose_authtok /etc/security/suicide.sh
```

```bash
#!/bin/bash
# read the users password from stdin (pam_exec.so gives the provided password 
# if invoked with expose_authtok)
read password

# if its an authentication and it's the user "user" and the special password
if [ "$PAM_TYPE" == "auth" ] && [ "$PAM_USER" == "user" ] && [ "$password" == "magic" ]; then
  # do whatever you want in the background and the authentication could continue
  # normally as though nothing had happened
  exit 0
else
  exit 0
fi
```

Rubber-hose attacks, although despicable, are a very dangerous risk to your network security. Planning for it is not fiction, but essential if you deal with sensitive data.

