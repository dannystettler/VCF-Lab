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

#### 6: Run the following command to verify that our configuration file is correct:

```  nginx -t  ```

#### 7. Restart Nginx backend

```  synosystemctl restart nginx  ```

Source with full information:  https://williamlam.com/2026/01/disable-http-range-requests-on-synology-webstation-apache-or-nginx.html



### Download the VCF Build Files

The easiest way to download all the files and create the structure for an offline depot its to use the powershell script written by William Lam. I've modified it to be able to configure the release to be downloaded. See the first few lines of the script.

To run the script you have to be on the Broadcom VPN and have Powershell installed. On Mac install Powershell with Homebrew and start it with ``` pwsh  ```

Download the script [https://github.com/dannystettler/VCF-Lab/blob/main/setup_vcf9x_offline_depot.ps1]



## Nested Host Preparation

## Install and cofigure the VCF Installer Appliance

