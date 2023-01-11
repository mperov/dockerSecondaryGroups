# dockerSecondaryGroups
There is flaw of Docker which is associated with Linux secondary groups.  
Here I try to show this.

### Requirements
0. Linux based OS.
1. [Install Docker](https://docs.docker.com/engine/install/) and run it in [rootless mode](https://docs.docker.com/engine/security/rootless/).
2. Create group and add your user to one:
```console
user@localhost$ su root
root@localhost# groupadd -g 7777 secret
root@localhost# usermod -a -G secret user
root@localhost# exit
user@localhost$ id
uid=1001(launcher) gid=1001(launcher) groups=1001(launcher),7777(secret)
```
3. Create directory with secret permission:
```console
user@localhost$ mkdir ~/topSecretDirectory
user@localhost$ su root
root@localhost# chown root:secret /home/launcher/topSecretDirectory/
root@localhost# chmod 770 /home/launcher/topSecretDirectory/
root@localhost# exit
user@localhost$ ls -al ~/topSecretDirectory/
total 8
drwxrwx---  2 root     secret 4096 Jan 11 17:56 .
user@localhost$ mkdir ~/topSecretDirectory/share
user@localhost$ chmod 777 ~/topSecretDirectory/share
```

### Problem
Run Ubuntu container with mount this directory:
```console
user@localhost$ docker run --mount type=bind,source=/home/user/topSecretDirectory/,target=/topSecretDirectory/ -it ubuntu bash
root@9196aa8f1ad2:/# ls /topSecretDirectory/
ls: cannot open directory '/topSecretDirectory/': Permission denied
```  
As you can see user has access to */home/user/topSecretDirectory/* on host but in container there is not permission to work with mounted directory.

Also if you add container user to secondary groups it does not help:
```console
user@localhost$ docker run --add-groups 7777 --mount type=bind,source=/home/user/topSecretDirectory/,target=/topSecretDirectory/ -it ubuntu bash
groups: cannot find name for group ID 7777
root@6e1dc734b5c6:/# id
uid=0(root) gid=0(root) groups=0(root),7777
root@6e1dc734b5c6:/# ls /topSecretDirectory/
ls: cannot open directory '/topSecretDirectory/': Permission denied
```
