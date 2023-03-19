# wpa_supplicant for UDM and UDM Pro
## UPDATE 03.19.2023
unifi-os v2.x changed a lot of things. 
Please save your files in `/data/` instead of `/mnt/data/`

unifi-os v2.x also requires you to manually install podman. [Instructions here](https://github.com/unifi-utilities/unifios-utilities/tree/main/podman-install)

**IT IS RECOMMENDED TO INSTALL AND RUN VERSION 2.X OF UNIFI-OS on the UDM/UDMP**

For those using a UDM-SE on version 3, podman kernel extensions were removed.

## Overview
This guide has primarily been written for authenticating to AT&T U-Verse using wpa_supplicant on a UDM/UDM-Pro running vesion 1.x and 2.x. 

Version 3.x can not run podman as the kernel extensions were removed from unifi-os version 3 and up.

**Please pay attention to your UDM version (1.x or 2.x) and use the guide accordingly**
**Those running version 1.6.3 will use the old path(version 1.x) but replace docker with podman**

This guide assumes you have your certificates extracted from an AT&T provided modem.   

## Important Notes

### AT&T service and hardware related
- This only works if you have the [White Nokia ONT](https://i.imgur.com/60TvKYN.png).
- For those who have no ONT and were issued the BGW-320, there may be a workaround using an XGS-PON ONT SFP Module. Might work if you have the [Nokia XS-020X-A ONT](https://www.dslreports.com/forum/r33298178-Meet-the-Nokia-XS-020X-A)?. To date, certificates can not be extracted from the BGW320.

### Certificate Related
If you don't have certificates, you can fetch a used modem off ebay such as:
- NVG510 (cheapest option)
- NVG589
- BGW210 (best option?)
Then root it to get the certificates.

**The BGW320 has no publically available way to extract certificates**

I had success using the following guides below.

- https://github.com/iwleonards/extract-mfg (best guide)
- https://www.dupuis.xyz/bgw210-700-root-and-certs/
- https://github.com/bypassrg/att

If the above link no longer works, copies are listed below:
- https://github.com/drpoutine/extract-mfg
- https://github.com/pbrah/att
- https://web.archive.org/web/20230308043241/https://www.dupuis.xyz/bgw210-700-root-and-certs/

Before moving on to the next section, make sure you've copied your root certificate into the CA_*.pem file per the instructions of the 802.1x Credential Extraction Tool (mfg_dat_decode). 

iwleonards's extract-mfg will supply everything organized after completion of the python script.

### Unifi-OS version 3 and up.
If you are running on a UDM-SE please note that unifi-os version 3 and up removes podman kernel extensions and therefore can not run podman.

Future updates to from version 3 and up on the older hardware will run into the same problem.

Disable auto updates to avoid issues.

## Prerequisites
- Have AT&T modem certificates (check above)
- For unifi version 2.x, [Install podman](https://github.com/unifi-utilities/unifios-utilities/tree/main/podman-install)
- **Optional** but [Highly recommended, install unifios utilities on-boot scripts to survive reboots](https://github.com/unifi-utilities/unifios-utilities)
## Setup/Installation
### 1.  SCP your certs and wpa_supplicant.conf to the UDM Pro

`cd` into your directory where the contents are located and run: `scp -r *.pem root@192.168.1.1:/tmp/`

Successful output below:
```
root@192.168.1.1's password:
CA_001E46-xxxx.pem                                                          100% 3926     3.8KB/s   00:00
Client_001E46-xxxx.pem                                                      100% 1119     1.1KB/s   00:00
PrivateKey_PKCS1_001E46-xxxx.pem                                            100%  887     0.9KB/s   00:00

scp -r wpa_supplicant.conf root@192.168.1.1:/tmp/
wpa_supplicant.conf                                                         100%  680     0.7KB/s   00:00
```

### 2. SSH into UDM

Time to create a directory for the certs and wpa_supplicant.conf in the docker directory then copy the files over.

**unifi-os v2.x**
```
mkdir -p /data/podman/wpa_supplicant/
cp -arfv /tmp/*pem /tmp/wpa_supplicant.conf /data/podman/wpa_supplicant/
```
**unifi-os v1.x**
```
mkdir -p /mnt/data/docker/wpa_supplicant/
cp -arfv /tmp/*pem /tmp/wpa_supplicant.conf /mnt/data/docker/wpa_supplicant/
```

### 3. Update the wpa_supplicant.conf to reflect the correct paths for our container. 
**Do not run these more than once or you will end up with incorrect paths.**

**unifi-os v2.x**
```
sed -i 's,ca_cert=",ca_cert="/etc/wpa_supplicant/conf/,g' /data/podman/wpa_supplicant/wpa_supplicant.conf
sed -i 's,client_cert=",client_cert="/etc/wpa_supplicant/conf/,g' /data/podman/wpa_supplicant/wpa_supplicant.conf
sed -i 's,private_key=",private_key="/etc/wpa_supplicant/conf/,g' /data/podman/wpa_supplicant/wpa_supplicant.conf
```
**unifi-os v1.x**
```
sed -i 's,ca_cert=",ca_cert="/etc/wpa_supplicant/conf/,g' /mnt/data/docker/wpa_supplicant/wpa_supplicant.conf
sed -i 's,client_cert=",client_cert="/etc/wpa_supplicant/conf/,g' /mnt/data/docker/wpa_supplicant/wpa_supplicant.conf
sed -i 's,private_key=",private_key="/etc/wpa_supplicant/conf/,g' /mnt/data/docker/wpa_supplicant/wpa_supplicant.conf
```

After running the sed commands, verify your paths in wpa_supplicant.conf look something like this:

Run `cat` on `wpa_supplicant.conf` in the path you used 
- on unifi-os v2.x: `cat /data/podman/wpa_supplicant/wpa_supplicant.conf`
- on unifi-os v1.x: `cat /mnt/data/docker/wpa_supplicant/wpa_supplicant.conf`
```
# Generated by 802.1x Credential Extraction Tool
# Copyright (c) 2018-2019 devicelocksmith.com
# Version: 1.04 linux amd64
#
# Change file names to absolute paths
eapol_version=1
ap_scan=0
fast_reauth=1
network={
        ca_cert="/etc/wpa_supplicant/conf/CA_001E46-xxxxxxxx.pem"
        client_cert="/etc/wpa_supplicant/conf/Client_001E46-xxxxxx.pem"
        eap=TLS
        eapol_flags=0
        identity="10:05:B1:xx:xx:xx" # Internet (ONT) interface MAC address must match this value
        key_mgmt=IEEE8021X
        phase1="allow_canned_success=1"
        private_key="/etc/wpa_supplicant/conf/PrivateKey_PKCS1_001E46-xxxxxx.pem"
}
# WARNING! Missing AAA server root CA! Add AAA server root CA to CA_001E46-xxxxxx.pem
#
```

### 4. Pull the docker image while you have an internet connection on the UDM Pro.  
**This step is optional if you plan on running step 5 while you have an internet connection.**
#### unifi-os v2.x (or unifi-os 1.6.3 and higher)
```
podman pull pbrah/wpa_supplicant-udmpro:v1.0
```

#### unifi-os v1.x
```
docker pull pbrah/wpa_supplicant-udmpro:v1.0
```

### 5. Run the wpa_supplicant docker container 
#### UDM-Pro 
***The docker/podman run command below assumes you are using port 9 or eth8 for your wan.  If not, adjust accordingly.***

**For unifi-os version 2:**
```
podman run --privileged --network=host --name=wpa_supplicant-udmpro -v /data/podman/wpa_supplicant/:/etc/wpa_supplicant/conf/ --log-driver=json-file --restart unless-stopped -d -ti pbrah/wpa_supplicant-udmpro:v1.0 -Dwired -ieth8 -c/etc/wpa_supplicant/conf/wpa_supplicant.conf
```

**For unifi-os version 1.6.3 but not version 2.x:**
```
podman run --privileged --network=host --name=wpa_supplicant-udmpro -v /mnt/data/docker/wpa_supplicant/:/etc/wpa_supplicant/conf/ --log-driver=json-file --restart unless-stopped -d -ti pbrah/wpa_supplicant-udmpro:v1.0 -Dwired -ieth8 -c/etc/wpa_supplicant/conf/wpa_supplicant.conf
```

**For unifi-os verison 1.6.2 or lower:**
```
docker run --privileged --network=host --name=wpa_supplicant-udmpro -v /mnt/data/docker/wpa_supplicant/:/etc/wpa_supplicant/conf/ --log-driver=json-file --restart unless-stopped -d -ti pbrah/wpa_supplicant-udmpro:v1.0 -Dwired -ieth8 -c/etc/wpa_supplicant/conf/wpa_supplicant.conf
```

#### UDM Standard (Egg shape/Non-pro)
**unifi-os v2.x**
```
podman run --privileged --network=host --name=wpa_supplicant-udmpro -v /data/podman/wpa_supplicant/:/etc/wpa_supplicant/conf/ --log-driver=k8s-file --restart always -d -ti pbrah/wpa_supplicant-udmpro:v1.0 -Dwired -ieth4 -c/etc/wpa_supplicant/conf/wpa_supplicant.conf
```
**unifi-os 1.6.3 but not version 2.x:**
```
podman run --privileged --network=host --name=wpa_supplicant-udmpro -v /mnt/data/docker/wpa_supplicant/:/etc/wpa_supplicant/conf/ --log-driver=k8s-file --restart always -d -ti pbrah/wpa_supplicant-udmpro:v1.0 -Dwired -ieth4 -c/etc/wpa_supplicant/conf/wpa_supplicant.conf
```
**unifi-os 1.6.2 and lower:**
```
docker run --privileged --network=host --name=wpa_supplicant-udmpro -v /mnt/data/docker/wpa_supplicant/:/etc/wpa_supplicant/conf/ --log-driver=k8s-file --restart always -d -ti pbrah/wpa_supplicant-udmpro:v1.0 -Dwired -ieth4 -c/etc/wpa_supplicant/conf/wpa_supplicant.conf
```

### 6. Successful Connection?
Run `podman logs -f wpa_supplicant-udmpro` or`docker logs -f wpa_supplicant-udmpro`

Successful connection should result in:
```
ethX: CTRL-EVENT-EAP-SUCCESS EAP authentication completed successfully
ethX: CTRL-EVENT-CONNECTED - Connection to XX:XX:XX:XX:XX:XX completed [id=0 id_str=]
```

### 7. Optional, Setup reboot survival

[**This step requires you to have installed boostchicken's onboot utilities script**](https://github.com/unifi-utilities/unifios-utilities/tree/main/on-boot-script-2.x)

**For unifi-os verion 2.x**
- Save the following to `/data/on_boot.d/10-wpa_supplicant.sh`
```
#!/bin/bash

## create files like this with different numbers for execution order
## ala /etc/profile.d

## example command to run, please replace with your own.
podman start wpa_supplicant-udmpro
```

## Create your own docker/podman image
For anyone that wants to create their own docker image, I've provided brief instructions below.

1. Grab docker/Dockerfile and upload it to /root/docker/ on the UDM Pro
2. Build image
```
cd /root/docker/
docker build --network=host -t pbrah/wpa_supplicant-udmpro:v1.0 .
```
## Troubleshooting
If you are having issues connecting after starting your docker container, the first thing you should do is check your docker container logs.

`podman logs -f wpa_supplicant-udmpro` or`docker logs -f wpa_supplicant-udmpro`

From a recent case I assisted in troubleshooting, the user saw the following in their logs.  The was due to their wpa_supplicant.conf having incorrect paths to the certificates.  Refer to my example in the instructions to ensure yours are pointing to the correct location.
```
OpenSSL: tls_connection_ca_cert - Failed to load root certificates error:02001002:system library:fopen:No such file or directory
OpenSSL: pending error: error:2006D080:BIO routines:BIO_new_file:no such file
OpenSSL: pending error: error:0B084002:x509 certificate routines:X509_load_cert_crl_file:system lib
OpenSSL: tls_load_ca_der - Failed load CA in DER format error:02001002:system library:fopen:No such file or directory
OpenSSL: pending error: error:20074002:BIO routines:file_ctrl:system lib
OpenSSL: pending error: error:0B06F002:x509 certificate routines:X509_load_cert_file:system lib
TLS: Failed to set TLS connection parameters
EAP-TLS: Failed to initialize SSL.
```
