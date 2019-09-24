# wpa_supplicant for UDM Pro
## overview
This guide has primary been written for authenticating to AT&T U-Verse using wpa_supplicant on a UDM Pro.  This guide assumes you've already retrieved your certificates from a modem supplied by AT&T.  If you have not, you can purchase a used modem on ebay, such as the NVG589 and then root it to get the certificates.  I had success using the following guide.


https://github.com/bypassrg/att


If the above link no longer works, I have also forked it to my GitHub below.

https://github.com/pbrah/att

Before moving on to the next section, make sure you've copied your root certificate into the CA_*.pem file per the instructions of the 802.1x Credential Extraction Tool (mfg_dat_decode).

## running the docker image
1.  scp your certs to the UDM Pro

```
scp -r *.pem root@192.168.1.1:/tmp/
root@192.168.1.1's password:
CA_001E46-xxxx.pem                                                          100% 3926     3.8KB/s   00:00
Client_001E46-xxxx.pem                                                      100% 1119     1.1KB/s   00:00
PrivateKey_PKCS1_001E46-xxxx.pem                                            100%  887     0.9KB/s   00:00

scp -r wpa_supplicant.conf root@192.168.1.1:/tmp/
wpa_supplicant.conf                                                         100%  680     0.7KB/s   00:00
```

2. ssh to the UDM Pro, create a directory for the certs and wpa_supplicant.conf in the docker directory then copy the files over.

```
mkdir /mnt/data/docker/wpa_supplicant/
cp -arfv /tmp/*pem wpa_supplicant.conf /mnt/data/docker/wpa_supplicant/
```

3. Update the wpa_supplicant.conf to reflect the correct paths for our container

```
sed -i 's,ca_cert=",ca_cert="/etc/wpa_supplicant/conf/,g' /mnt/data/docker/wpa_supplicant/wpa_supplicant.conf
sed -i 's,client_cert=",client_cert="/etc/wpa_supplicant/conf/,g' /mnt/data/docker/wpa_supplicant/wpa_supplicant.conf
sed -i 's,private_key=",private_key="/etc/wpa_supplicant/conf/,g' /mnt/data/docker/wpa_supplicant/wpa_supplicant.conf
```

4. Run the wpa_supplicant docker container, the docker run command below assumes you are using port 9 or eth8 for your wan.  If not, adjust accordingly.

```
docker run --privileged --network=host --name=wpa_supplicant-udmpro -v /mnt/data/docker/wpa_supplicant/:/etc/wpa_supplicant/conf/ --log-driver=json-file --restart unless-stopped -d -ti pbrah/wpa_supplicant-udmpro:v1.0 -Dwired -ieth8 -c/etc/wpa_supplicant/conf/wpa_supplicant.conf
```

## create your own docker image
For anyone that wants to create their own docker image, I've provided brief instructions below.

1. grab docker/Dockerfile and upload it to /root/docker/ on the UDM Pro
2. Build image
```
cd /root/docker/
docker build --network=host -t pbrah/wpa_supplicant-udmpro:v1.0 .
