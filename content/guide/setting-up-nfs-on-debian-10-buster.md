---
title: "Setting Up Nfs on Linux"
type: post
date: 2020-09-16T08:48:35-04:00
url: /setting-up-nfs-on-debian-10-buster/
image: /images/2020-thumbs/setting-up-nfs-on-debian-10-buster.jpg
categories:
  - Guide
tags:
  - Self-Hosting
  - Linux
draft: false
---
# I feel my neckbeard growing in here.

So there is a [video guide](https://youtu.be/Kg2XaTw66kc) I made a few months back.

## Required Packages
Debian/Ubuntu:  
- Server Only
> apt install nfs-kernel-server
- Client Only
> apt install nfs-common

CentOS/Redhat/Fedora (should come preinstalled)
> dnf install nfs-utils

## Setting up the file share
There's a few things that need done here. First things first we need to pick a directory, or make one. Personally I usually make one right in the root "/" directory.  
> mkdir /share  

Now that it's created we now need to setup read/write privelidges. For now we're just gonna open this share up to everyone for the sake of tutorial.

> chown nobody:nobody /share  
> chmod 755 /share

Now we have that setup we need to tell nfs-kernel-server to actually do something
> EDITOR /etc/exports

That'll open a nice text field, *on a debian system it comes comments with some nice examples like this*
>  /etc/exports: the access control list for filesystems which may be exported  
> 		to NFS clients.  See exports(5).
> 
>  Example for NFSv2 and NFSv3:  
>  /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)  
> 
>  Example for NFSv4:  
>  /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)  
>  /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)

What we need to do is now tell nfs-kernel-server a few things:  
1. What directory we're sharing
2. Who we're sharing it with
3. How we're [exporting it](https://linux.die.net/man/5/exports)

Here's a quck example:  
> /share	192.168.1.*(rw,sync,no_subtree_check)

1. We're sharing "/share"
2. We're sharing it with every computer with an IP address starting with 192.168.1.xxx (the * means any)
3. We're giving rw privelidges, syncing our drives on the server, without worry about our subtree.

And now that we're all setup it's time to tell it to export, but first we need to make sure nfs is running. I use systemd, but I'm sure some websearch results can get you an init script somewhere. NFS has been around for years now.  
> systemctl enable --now nfs-server.service

Simple enough. Now that we have nfs running on our server, it's time to actually tell it to do something other than sit around.
> exportfs -arv

Here's some copy paste form the man page for exportfs, which you can always get yourself from simply running 
> man exportfs

       -a     Export or unexport all directories.

       -r     Reexport all directories, synchronizing  /var/lib/nfs/etab  with
              /etc/exports   and  files  under  /etc/exports.d.   This  option
              removes entries in /var/lib/nfs/etab  which  have  been  deleted
              from /etc/exports or files under /etc/exports.d, and removes any
              entries from the kernel export table which are no longer valid.
       -v     Be verbose. When exporting or unexporting, show what's going on.
              When displaying the current export list, also display  the  list
              of export options.

If you're running an SELinux distro like CentOS or Redhat you do need to setup some quick firewall rules
> firewall-cmd --permanent --add-service=nfs  
> firewall-cmd --permanent --add-service=rpc-bind  
> firewall-cmd --permanent --add-service=mountd

Anyway that's it for the Server!

## Setting up the client
*Make sure that if your on a debian based distro you have nfs-common installed.*

So time to setup our client to receive the file share. First off lets see if our server is working properly. This command will only work if you run it as **root** or with ***sudo*** / ***doas***
> showmount -e ip.address.of.server

If you get an output showing you directorie/s, then it's working fine. These will only show if you're client's IP address is in the range you set in /etc/exports  
So now, we need to figure out where we want to find out /share folder at. Personally since my cleint systems only have a single user account I mount them right in /home/$user/ myself.  
But first we need to remember that everything must be a file or folder, we simply cannot tell somehthing it ecists when it's not here on our client afterall.  

> mkdir /home/$user/share

So now we can mount our share

> mount -t nfs  ip.address.of.server:/share /home/$user/share

And wahla, you now have a mounted share on your client. However this is not a permanent mount. To do that we need to edit our /etc/fstab file, I'll just drop this line right in the bottom

> ip.address.of.server:/share /home/$user/share nfs defaults,_netdev 0 0

It's generally a good idea to include _netdev to tell our client thta it's a network device. If this isn't here and your server is not connected to the network this can actually cause a no boot scenario.

## Why NFS?  
Well if your setting up a media share for [Kodi](https://kodi.tv/) NFS actually gives the best performance, so less network lag for your 4k picture. Plus it's a well established protocol and is very reliable.  
That said it can easily be a security nightmare as well. In our test scenario here we literally opened an unencrypted share to basically every device on the network. Not exactly a good security practice there.  
But, if your doing something liek giving all your raspberri Pi's a synced home folder, you can mount /home via NFS. So all your Pi's can share the same /home. Might not be the best use case, but it's something I do.  
Do be aware that windows doesn't really play all that well with NFS though, if you want to setup a network share, stick to Samba if your on Windows.

## Conclusion
Anyways, that's a nice simple setup for nfs, I hope you found it quite educational in the very least. If you have question you can always reach me using the social links locate on this website.