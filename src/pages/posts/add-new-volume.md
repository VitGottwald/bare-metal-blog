---
title: 'ZOS - Adding a volume'
pubDate: 2024-03-09
description: 'The fun of adding a volume to z/OS UNDER ZVDT.'
author: 'Baremetalfreak'
image:
    url: 'https://docs.astro.build/assets/rose.webp'
    alt: 'The Astro logo on a dark background with a pink glow.'
tags: ["z/OS", "MVS", "storage", "ICKDSF", "learning in public"]
---

```jcl
//ICKDSF   EXEC PGM=ICKDSF,REGION=0K
//SYSPRINT DD   SYSOUT=*
//SYSIN    DD   *
 INIT UNIT(<unit>) NOVERIFY VOLID(<volnam>) -
      VTOC(0,1,149) INDEX(15,0,100) -
      PURGE STORAGEGROUP
//
```
Then check the joblog and reply to the WTOR message with 'U'.

```
MOUNT FILESYSTEM('PRODUCT.ZFS')                  
      TYPE(ZFS)                                  
      MODE(RDWR) NOAUTOMOVE                      
      MOUNTPOINT('/u/users/zot') PARM('AGGRGROW')
```

```sh
zfsadm define -aggregate <dsn> -megabytes 4000 -storageclass <stgcls> 
zfsadm format -aggregate <dsn> -compat -overwrite
/usr/sbin/mount -t ZFS -o aggrgrow -f <dsn> </path/to/mount/dir>
```
