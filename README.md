### The fastest remote directory rsync over ssh archival I can muster (40MB/s over 1gb NICs)

#### This creates an archive that does the following:

**rsync**
(Everyone seems to like -z, but it is much slower for me)

- a: archive mode - rescursive, preserves owner, preserves permissions, preserves modification times, preserves group, copies symlinks as symlinks, preserves device files.
- H: preserves hard-links
- A: preserves ACLs
- X: preserves extended attributes
- x: don't cross file-system boundaries
- v: increase verbosity
- --numeric-ds: don't map uid/gid values by user/group name
- --delete: delete extraneous files from dest dirs (differential clean-up during sync)
- --progress: show progress during transfer

**ssh**
- T: turn off pseudo-tty to decrease cpu load on destination.
- c arcfour: use the weakest but fastest SSH encryption. Must specify "Ciphers arcfour" in sshd_config on destination.
- o Compression=no: Turn off SSH compression.
- x: turn off X forwarding if it is on by default.

**Original**

```sh
rsync -aHAXxv --numeric-ids --delete --progress -e "ssh -T -c arcfour -o Compression=no -x" user@<source>:<source_dir> <dest_dir>
```


**Flip** 

```sh
rsync -aHAXxv --numeric-ids --delete --progress -e "ssh -T -c arcfour -o Compression=no -x" [source_dir] [dest_host:/dest_dir]
```

for a Mac to Linux transfer it's useful to use other options. Arcfour is not available for most new machines anymore and UTF-8 on OS X is different than UTF-8 on Linux (important if you have Umlauts like Germans, Samba/NFS will fail otherwise). My command if both (Mac & Linux) machines support AES on their processors and you want to transfer from Mac to Linux:
```sh
rsync -rltv --progress --human-readable --delete --iconv=utf-8-mac,utf-8 -e 'ssh -T -c aes128-gcm@openssh.com -o Compression=no -x' <local_mac_source> <remote_linux_dest>
reverse the iconv option if you want to transfer from Linux to Mac.
```

**To allow arcfour cipher:**
sshd config
Add at the end of /etc/sshd_config file:

```sh
# enable all ciphers!
# obtained with ssh -Q cipher localhost | paste -d , -s
Ciphers 3des-cbc,blowfish-cbc,cast128-cbc,arcfour,arcfour128,arcfour256,aes128-cbc,aes192-cbc,aes256-cbc,rijndael-cbc@lysator.liu.se,aes128-ctr,aes192-ctr,aes256-ctr,aes128-gcm@openssh.com,aes256-gcm@openssh.com,chacha20-poly1305@openssh.com
```

Use -P instead --progress, because this is equivalent for --progress --partial




Since -c arcfour is considered obsolete, using -c aes128-gcm@openssh.com is probably fastest.

rsync -aHAXxv --numeric-ids --delete --progress -e "ssh -T -c aes128-gcm@openssh.com -o Compression=no -x" [source_directory] user@hostname:[target_directory]/

See these encryption benchmark results: https://famzah.files.wordpress.com/2015/06/openssh-ciphers-performance-2015-chart.jpg

yo
