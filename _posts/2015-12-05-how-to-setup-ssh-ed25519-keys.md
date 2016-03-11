---
title: How To - Setup SSH Ed25519 keys
---

So you are still following those old how to guides online about generating ssh keys? I hope you are using the RSA algorithm because DSA has been deprecated starting with [OpenSSH 7](http://www.openssh.com/txt/release-7.0){:target="_blank"}.

>Support for ssh-dss, ssh-dss-cert-* host and user keys is disabled by default at run-time. These may be re-enabled using the instructions at http://www.openssh.com/legacy.html

With the introduction of OpenSSH 6.5 we gained a new algorithm, Ed25519 which is faster, smaller and offers better security. Ed25519 was developed by a team including Daniel J. Bernstein, Niels Duif, Tanja Lange, Peter Schwabe, and Bo-Yin Yang. Any old Unix admin's will instantly recognize Daniel J. Bernstein (djb) the developer of some of the more security focused software ever written including: qmail, djbdns, tinydns and daemontools.

I am going to specify the -o flag (or option) and increase the number of KDF (key derivation function) rounds from 16 to 128. The more rounds, the slower the passphrase verification will become, but the ability to brute-force my password will become harder and harder!

{% highlight text %}
box:~$ cd ~/.ssh

box:~$ ssh-keygen -t ed25519 -o -a 128
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/user/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/user/.ssh/id_ed25519.
Your public key has been saved in /home/user/.ssh/id_ed25519.pub.
The key fingerprint is:
SHA256:PlUk1UiXuJbIaY3pTke1jNRCZmFDM5o9kHqQSqvLWms user@box
The key's randomart image is:
+--[ED25519 256]--+
|          .oO=X++|
|      . o  o.@.oo|
|       + = .. o .|
|      . +..    ..|
|  o  .  S o    . |
| o o..     o  .  |
|  =.=.    . .. . |
| + Bo      o..+  |
|E.=..      .o  o |
+----[SHA256]-----+
{% endhighlight %}

My old RSA keys was 3.3K my new Ed22519 key is much smaller at 464B. One of the things we have gotten use to in the industry, is the longer or larger the key the more "secure" we felt. The new algorithm is signed with SHA512 which is superior to the previous methods and still much smaller.

Now that we have a our new new Ed25519 key, what about all the machines that have not been upgraded to take advantage? The old keys, the passphrase would have their cipher key hashed with MD5. It was fast but pretty insecure because with todays modern off-the-shelf hardware can brute-force a MD5 hash in no time.

Here is how I generate my legacy RSA key's. I force the bit size as large as I can to 4096, note that I don't for a Ed25519 key because it is a fixed size and ignored. In this example I set the KDF to 128 rounds as well.

{% highlight text %}

box:~$ ssh-keygen -t rsa -b 4096 -o -a 128
Generating public/private rsa key pair.
Enter file in which to save the key (/home/user/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/user/.ssh/id_rsa.
Your public key has been saved in /home/user/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:Np7xkMoRO2dmaHRXKbws0cNA/0o2B0istURQ/CUMwAA user@box
The key's randomart image is:
+---[RSA 4096]----+
|ooE.oo*B=+.oo    |
| +.. +o++=o.     |
|. o   ..++o .    |
| + . .. =o o     |
|. o . o+S..      |
|   . o BoO       |
|      + + .      |
|                 |
|                 |
+----[SHA256]-----+
{% endhighlight %}

Now comes the hard part, how to work with two different systems. You could edit your ssh config file on your local machine and specify per server like so:

{% highlight text %}
box:~$ vi ~/.ssh/config

Host old.server.tld
  IdentityFile ~/.ssh/id_rsa
Host new.server.tld
  IdentityFile ~/.ssh/id_ed25519
{% endhighlight %}

This is to much work for a lazy SA like me. So what I do is set the default key to my new Ed25519 private key and then when it fails I called it via ssh client's -i flag.

{% highlight text %}
box:~$ cat <<EOF >> ~/.ssh/config
Host *
        IdentityFile ~/.ssh/id_ed25519
EOF

box:~$ mv id_rsa old
box:~$ ssh -i old user@host
{% endhighlight %}

I find that it is a PITA and waste of time to wait for the failure. So since I am lazy at heart, it gives me incentive to upgrade sshd on the device I am connecting to, where possible.
