# Setting up a VCF9.x LAB

These instructions are covering the parts required for setting up a nested VCF Homelab. Main scope is to use a "manual" setup using the VCF Installer. However, it is also useful when you consider the use of Holodeck. 

## Offline Depot

There are several ways for hosting a Offline Depot. My preferred way is to use my Synology for it. It will be a HTTP Depot.

### Setting up Synology Depot

- Install and start *Web Station Package*
  
- Create a new *Web Service* with the folowing settings:
  - Service Type: *Static Website*
  - Name & Description: *vcf-depot* (example)
  - Document Root: *file location*
  - HTTP back-end server: *Nginx*
 
- Create a new *Web Portal* with the following settings:
  - Select *Web Service Portal*
  - Service: *vcf-depot*
  - Portal type: *Port-based*
  - Port: HTTP: (free portnumber -> e.g 9100)

### Disable HTTP Range Requests
Since VCF 9.1 retrieving data from the Depot requires to deactivate "HTTP Range Request". 

#### 1. SSH into Syno and switch to root provileges
```
ssh <adminaccount>@<syno-ip>
sudo su -
```
#### 2. Find Web Portal configuration file 

```
cd /usr/local/etc/nginx/sites-available
grep <port number of webservice> *
```
<img width="873" height="125" alt="image" src="https://github.com/user-attachments/assets/06851a1f-825a-476e-b8be-e40fdc82928f" />


#### 3. Retrieve content of file (*.w3conf)

```  cat <file name>  ```

and look for the very last include statement, which in my example is:
<img width="1086" height="462" alt="image" src="https://github.com/user-attachments/assets/83f60ab1-a5e5-4189-8e1b-adc643321fdc" />



#### 4. Retrieve path to the configuration file which needs to be created and modified

```  cat ./../<path from file>  ```

<img width="1436" height="179" alt="image" src="https://github.com/user-attachments/assets/e9508741-1346-49fe-8691-32a1236e3d24" />

This path is now the location the new configuration file needs to be created and modified


#### 5. Create Parent Diretory and user.conf file

```
mkdir -p /usr/local/etc/nginx/conf.d/<parentdirectory>
vi /usr/local/etc/nginx/conf.d/<parentdirectory>/user.conf
```

and paste the folwoing content into the file. Ajust the path as needed:

```
location /PROD/ {
    alias /volume1/web/PROD/;
    autoindex on;

    # Disable Byte-Range requests
    max_ranges 0;
    set $http_range "";
    add_header Accept-Ranges "none" always;

    # Performance
    gzip off;

    # Ensure Nginx looks for the file before giving up
    try_files $uri $uri/ =404;
}
```

remember to save the file with ``` ESC :wq  ```

#### 6: Run the following command to verify that our configuration file is correct:

```  nginx -t  ```

#### 7. Restart Nginx backend

```  synosystemctl restart nginx  ```

Source with full information:  https://williamlam.com/2026/01/disable-http-range-requests-on-synology-webstation-apache-or-nginx.html



### Download the VCF Build Files

The easiest way to download all the files and create the structure for an offline depot its to use the powershell script written by William Lam. I've modified it to be able to configure the release to be downloaded. See the first few lines of the script.

To run the script you have to be on the Broadcom VPN and have Powershell installed. On Mac install Powershell with Homebrew and start it with ``` pwsh  ```

Download the script [https://github.com/dannystettler/VCF-Lab/blob/main/setup_vcf9x_offline_depot.ps1]

After downloaded transfer the complete folder structure to the prepared webserver the direectory PROD needs to be the first layer under the document root. 


## Nested Host Preparation

### Prerequisites

The host hosting the nested ESX VMs must have:
- running ESX 8.0 U3 or later
- a trunc portgroup configured
  - VLAN ID: all (4095)
  - Promiscuous mode: Accept
  - MAC address changes: Accept
  - Forged transmits: Accept

### Create VM Object
- Guest OS: Other - VMware ESXi 8.0 or later
- CPU: 12
  - Hardware virtualization: activate Expose hardware assisted virtualization to the guest OS
- Memory: 96 GB
- Harddisk:
  - Disk1: 32 GB (thin)
  - Disk2: 900 GB (thin)
- Network:
  - Nic1: Trunk Portgroup (4095)
  - Nic2: Trunk Portgroup (4095)
- CD/DVD Drive: Datastore ISO with desired version of ESXi Build

### Base configuration of ESXi on console

- configure static IP address
- configure name with FQDN
- configure DNS Servers and DNS suffix
- activate SSH

### Configure NTP using Webclient

<img width="1540" height="707" alt="image" src="https://github.com/user-attachments/assets/e1b84ae8-04d1-4a22-a691-0cfa354f04c0" />


### Transfer VIB for VSAN ESA mockup
When you need to cheat the VCF System because you don't have a certified VSAN ESA Node there are a few options. In the past this was achieved by installing a small VIB on every host to satify the compatibility checks. The neweer version is to configure the VCF installer or the SDDC Manager to skip these tests. (see https://williamlam.com/2025/09/enhancement-in-vcf-9-0-1-to-bypass-vsan-esa-hcl-host-commission-10gbe-nic-check.html)

However, I'm still a fan to just installing the VIB, so I just stick with this solution for the moment. The procedure is quite simple. Transfer the VIB to the host ans install it with a SSH session. done.
Use this file: [https://github.com/dannystettler/VCF-Lab/blob/main/nested-vsan-esa-mock-hw.vib] and trasfer it to the VM Host

```   scp nested-vsan-esa-mock-hw.vib root@<esx-host-ip>:/tmp   ```

### SSH into host to install VIB, recreate the server certificate and perform a final reboot 
Login with root into ESX and perform the following commands

```
esxcli software acceptance set --level CommunitySupported
esxcli software vib install -v /tmp/nested-vsan-esa-mock-hw.vib --no-sig-check

/sbin/generate-certificates

reboot
```


## Install and cofigure the VCF Installer Appliance

