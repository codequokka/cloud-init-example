# Cloud-init example

## Environment
```
❯ cat /etc/redhat-release
Rocky Linux release 8.7 (Green Obsidian)
```

## Install the XLD
```
❯ sudo snap install lxd

❯ snap list lxd
Name  Version      Rev    Tracking       Publisher   Notes
lxd   5.9-76c110d  24164  latest/stable  canonical✓  -
```

## Setup the XLD
```
❯ sudo /var/lib/snapd/snap/bin/lxd init --minimal

❯ sudo reboot

❯ lxc profile show default
config: {}
description: Default LXD profile
devices:
  eth0:
    name: eth0
    network: lxdbr0
    type: nic
  root:
    path: /
    pool: default
    type: disk
name: default
used_by: []

❯ lxc storage list
+---------+--------+--------------------------------------------+-------------+---------+---------+
|  NAME   | DRIVER |                   SOURCE                   | DESCRIPTION | USED BY |  STATE  |
+---------+--------+--------------------------------------------+-------------+---------+---------+
| default | lvm    | /var/snap/lxd/common/lxd/disks/default.img |             | 2       | CREATED |
+---------+--------+--------------------------------------------+-------------+---------+---------+

❯ lxc network list
+---------+----------+---------+----------------+---------------------------+-------------+---------+---------+
|  NAME   |   TYPE   | MANAGED |      IPV4      |           IPV6            | DESCRIPTION | USED BY |  STATE  |
+---------+----------+---------+----------------+---------------------------+-------------+---------+---------+
| docker0 | bridge   | NO      |                |                           |             | 0       |         |
+---------+----------+---------+----------------+---------------------------+-------------+---------+---------+
| enp0s3  | physical | NO      |                |                           |             | 0       |         |
+---------+----------+---------+----------------+---------------------------+-------------+---------+---------+
| lxdbr0  | bridge   | YES     | 10.154.56.1/24 | fd42:aa8b:7cde:a4b7::1/64 |             | 1       | CREATED |
+---------+----------+---------+----------------+---------------------------+-------------+---------+---------+
| virbr0  | bridge   | NO      |                |                           |             | 0       |         |
+---------+----------+---------+----------------+---------------------------+-------------+---------+---------+
```

## Create the user data
```
❯ vi my-user-data

❯ cat my-user-data
#cloud-config
runcmd:
  - echo 'Hello, World!' > /var/tmp/hello-world.txt
```

## Launch the LXD container
```
❯ lxc launch ubuntu:focal my-test --config=user.user-data="$(cat ./my-user-data)"
Creating my-test
Starting my-test

❯ lxc list
+---------+---------+------+-----------------------------------------------+-----------+-----------+
|  NAME   |  STATE  | IPV4 |                     IPV6                      |   TYPE    | SNAPSHOTS |
+---------+---------+------+-----------------------------------------------+-----------+-----------+
| my-test | RUNNING |      | fd42:aa8b:7cde:a4b7:216:3eff:fe3b:49ba (eth0) | CONTAINER | 0         |
+---------+---------+------+-----------------------------------------------+-----------+-----------+
```

## Check the execution result of cloud-init
```
❯ lxc shell my-test
root@my-test:~# cloud-init status --wait

status: done

root@my-test:~# cloud-init query userdata
#cloud-config
runcmd:
  - echo 'Hello, World!' > /var/tmp/hello-world.txt

root@my-test:~# cloud-init schema --system --annotate
Valid cloud-config: system userdata

root@my-test:~# ls -l /var/tmp/hello-world.txt
-rw-r--r--. 1 root root 14 Dec 28 20:41 /var/tmp/hello-world.txt

root@my-test:~# cat /var/tmp/hello-world.txt
Hello, World!

root@my-test:~# exit
logout
```

## Delete the LXD container
```
❯ lxc stop my-test

❯ lxc list
+---------+---------+------+------+-----------+-----------+
|  NAME   |  STATE  | IPV4 | IPV6 |   TYPE    | SNAPSHOTS |
+---------+---------+------+------+-----------+-----------+
| my-test | STOPPED |      |      | CONTAINER | 0         |
+---------+---------+------+------+-----------+-----------+

❯ lxc rm my-test

❯ lxc list
+------+-------+------+------+------+-----------+
| NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS |
+------+-------+------+------+------+-----------+
```

## Password hash generation
```
❯ python3 -c "import crypt, getpass, pwd; print(crypt.crypt('password','\$6\$salt\$'))"
$6$salt$IxDD3jeSOb5eB1CX5LBsqZFVkJdido3OUILO5Ifz5iwMuTS4XMS130MTSuDDl3aCI6WouIL9AjRbLCelDCy.g.
```

## List public images
```
❯ lxc image list images: 'rocky'
+-------------------------------+--------------+--------+---------------------------------------+--------------+-----------------+----------+-------------------------------+
|             ALIAS             | FINGERPRINT  | PUBLIC |              DESCRIPTION              | ARCHITECTURE |      TYPE       |   SIZE   |          UPLOAD DATE          |
+-------------------------------+--------------+--------+---------------------------------------+--------------+-----------------+----------+-------------------------------+
| rockylinux/8 (3 more)         | 68c57826b76f | yes    | Rockylinux 8 amd64 (20221214_02:07)   | x86_64       | VIRTUAL-MACHINE | 660.31MB | Dec 14, 2022 at 12:00am (UTC) |
+-------------------------------+--------------+--------+---------------------------------------+--------------+-----------------+----------+-------------------------------+
| rockylinux/8 (3 more)         | dd7ac9352c30 | yes    | Rockylinux 8 amd64 (20221214_02:07)   | x86_64       | CONTAINER       | 128.49MB | Dec 14, 2022 at 12:00am (UTC) |
+-------------------------------+--------------+--------+---------------------------------------+--------------+-----------------+----------+-------------------------------+
| rockylinux/8/arm64 (1 more)   | eb90d51ab793 | yes    | Rockylinux 8 arm64 (20221214_02:07)   | aarch64      | CONTAINER       | 125.15MB | Dec 14, 2022 at 12:00am (UTC) |
+-------------------------------+--------------+--------+---------------------------------------+--------------+-----------------+----------+-------------------------------+
| rockylinux/8/cloud (1 more)   | 8eb86b9769c9 | yes    | Rockylinux 8 amd64 (20221214_02:50)   | x86_64       | CONTAINER       | 147.96MB | Dec 14, 2022 at 12:00am (UTC) |
+-------------------------------+--------------+--------+---------------------------------------+--------------+-----------------+----------+-------------------------------+
| rockylinux/8/cloud (1 more)   | c34a9ab54be7 | yes    | Rockylinux 8 amd64 (20221214_02:50)   | x86_64       | VIRTUAL-MACHINE | 679.02MB | Dec 14, 2022 at 12:00am (UTC) |
+-------------------------------+--------------+--------+---------------------------------------+--------------+-----------------+----------+-------------------------------+
| rockylinux/8/cloud/arm64      | 5fe7261c0542 | yes    | Rockylinux 8 arm64 (20221214_02:51)   | aarch64      | CONTAINER       | 144.13MB | Dec 14, 2022 at 12:00am (UTC) |
+-------------------------------+--------------+--------+---------------------------------------+--------------+-----------------+----------+-------------------------------+
| rockylinux/9 (3 more)         | 745dc14379f8 | yes    | Rockylinux 9 amd64 (20221214_02:06)   | x86_64       | VIRTUAL-MACHINE | 565.64MB | Dec 14, 2022 at 12:00am (UTC) |
+-------------------------------+--------------+--------+---------------------------------------+--------------+-----------------+----------+-------------------------------+
| rockylinux/9 (3 more)         | d0198d375d77 | yes    | Rockylinux 9 amd64 (20221214_02:06)   | x86_64       | CONTAINER       | 109.57MB | Dec 14, 2022 at 12:00am (UTC) |
+-------------------------------+--------------+--------+---------------------------------------+--------------+-----------------+----------+-------------------------------+
| rockylinux/9/arm64 (1 more)   | e5765d25731b | yes    | Rockylinux 9 arm64 (20221214_02:07)   | aarch64      | CONTAINER       | 105.57MB | Dec 14, 2022 at 12:00am (UTC) |
+-------------------------------+--------------+--------+---------------------------------------+--------------+-----------------+----------+-------------------------------+
| rockylinux/9/cloud (1 more)   | 8d68319b7775 | yes    | Rockylinux 9 amd64 (20221214_02:06)   | x86_64       | CONTAINER       | 125.32MB | Dec 14, 2022 at 12:00am (UTC) |
+-------------------------------+--------------+--------+---------------------------------------+--------------+-----------------+----------+-------------------------------+
| rockylinux/9/cloud (1 more)   | efb20626c922 | yes    | Rockylinux 9 amd64 (20221214_02:06)   | x86_64       | VIRTUAL-MACHINE | 585.42MB | Dec 14, 2022 at 12:00am (UTC) |
+-------------------------------+--------------+--------+---------------------------------------+--------------+-----------------+----------+-------------------------------+
| rockylinux/9/cloud/arm64      | 7359a5581e4a | yes    | Rockylinux 9 arm64 (20221214_02:07)   | aarch64      | CONTAINER       | 120.88MB | Dec 14, 2022 at 12:00am (UTC) |
+-------------------------------+--------------+--------+---------------------------------------+--------------+-----------------+----------+-------------------------------+
| rockylinux/9/cloud/ppc64el    | 40b7d00bc76c | yes    | Rockylinux 9 ppc64el (20221214_02:06) | ppc64le      | CONTAINER       | 127.28MB | Dec 14, 2022 at 12:00am (UTC) |
+-------------------------------+--------------+--------+---------------------------------------+--------------+-----------------+----------+-------------------------------+
| rockylinux/9/ppc64el (1 more) | 0fd56b849161 | yes    | Rockylinux 9 ppc64el (20221214_02:06) | ppc64le      | CONTAINER       | 111.31MB | Dec 14, 2022 at 12:00am (UTC) |
+-------------------------------+--------------+--------+---------------------------------------+--------------+-----------------+----------+-------------------------------+`
```
