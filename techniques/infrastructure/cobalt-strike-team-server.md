---
description: Quick commands to set up a Cobalt Strike team server.
---

# Cobalt Strike Team Server

Firewall settings:

```bash
INBOUND
Allow all from lab IPs
Allow HTTP & HTTPS from HTTP(S) redirectors
Allow HTTP and DNS from DNS redirectors
Deny all 
```

As root on server:

```bash
apt update #update initial image
apt upgrade 

apt install openjdk-11-jdk #install OpenJDK Java environment
update-java-alternatives -s java-1.11.0-openjdk-amd64 #set Java environment 

#upload cobaltstrike binaries

tar xvzf /home/ubuntu/cobaltstrike-dist.tgz -C /root/  #extract to /root/cobaltstrike

cd /root/cobaltstrike/
./update #This can fail if not run from the correct directory 
#Enter licence key 

```

