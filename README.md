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
Since VCF 9.1 retrieving data from the Depot requires to deactivate "HTTP Range Request". On Synology it's not very straight forward.

#### 1. SSH into Syno and switch to root provileges
''' ssh admin\






https://williamlam.com/2026/01/disable-http-range-requests-on-synology-webstation-apache-or-nginx.html




### Download the VCF Build Files

## Nested Host Preparation

## Install and cofigure the VCF Installer Appliance

