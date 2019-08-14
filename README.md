# Proxmox-LXC
The following is for creating LXC containers.

Network Prerequisites are:
- [x] Layer 2 Network Switches
- [x] Network Gateway is `192.168.1.5`
- [x] Network DNS server is `192.168.1.5` (Note: your Gateway hardware should enable you to a configure DNS server(s), like a UniFi USG Gateway, so set the following: primary DNS `192.168.1.254` which will be your PiHole server IP address; and, secondary DNS `1.1.1.1` which is a backup Cloudfare DNS server in the event your PiHole server 192.168.1.254 fails or os down)
- [x] Network DHCP server is `192.168.1.5`
- [x] A DDNS service is fully configured and enabled (I recommend you use the free Synology DDNS service)
- [x] A ExpressVPN account (or any preferred VPN provider) is valid and its smart DNS feature is working (public IP registration is working with your DDNS provider)

Other Prerequisites are:
- [x] Synology NAS, or linux variant of a NAS, is fully configured as per [SYNOBUILD](https://github.com/ahuacate/synobuild#synobuild)
- [x] Proxmox node fully configured as per [PROXMOX-NODE BUILDING](https://github.com/ahuacate/proxmox-node/blob/master/README.md#proxmox-node-building)
- [x] pfSense is fully configured on typhoon-01 including both OpenVPN Gateways VPNGATE-LOCAL and VPNGATE-WORLD.

Tasks to be performed are:
- [ ] 1.0 PiHole LXC - CentOS7
- [ ] 2.0 UniFi Controller - CentOS7
- [ ] 3.0 Jellyfin LXC - CentOS (*Not working*)
- [ ] 4.0 Jellyfin LXC - Ubuntu 18.04

>  **About LXC Installations:**
CentosOS7 is my preferred linux distribution for VMs and LXC containers. Although, some applications like Jellyfin work best on Ubuntu.
Proxmox itself ships a set of basic templates and to download a prebuilt distribution use the graphical interface `typhoon-01` > `local` > `content` > `templates` and select and download `centos-7-default`, `ubuntu-16.04-standard` and `ubuntu-18.04-standard` templates.

## 1.0 PiHole LXC - CentOS7
Here we are going install PiHole which is a internet tracker blocking application which acts as a DNS sinkhole. Basically its charter is to block advertisments, tracking domains, tracking cookies and all those personal data mining collection companies.

### 1.1 Create a CentOS7 LXC for PiHole
Now using the web interface `Datacenter` > `Create CT` and fill out the details as shown below (whats not shown below leave as default):

| Create: LXC Container | Value |
| :---  | :---: |
| **General**
| Node | `typhoon-01` |
| CT ID |`254`|
| Hostname |`pihole`|
| Unprivileged container | `☑` |
| Resource Pool | Leave Blank
| Password | Enter your pasword
| Password | Enter your pasword
| SSH Public key | Add one if you want to
| **Template**
| Storage | `local` |
| Template |`centos-7-default_xxxx_amd`|
| **Root Disk**
| Storage |`typhoon-share`|
| Disk Size |`8 GiB`|
| **CPU**
| Cores |`1`|
| CPU limit | Leave Blank
| CPU Units | `1024`
| **Memory**
| Memory (MiB) |`256`|
| Swap (MiB) |`256`|
| **Network**
| Name | `eth0`
| Mac Address | `auto`
| Bridge | `vmbr0`
| VLAN Tag | Leave Blank
| Rate limit (MN/s) | Leave Default (unlimited)
| Firewall | `☑`
| IPv4 | `☑  Static`
| IPv4/CIDR |`192.168.1.254/24`|
| Gateway (IPv4) |`192.168.1.5`|
| IPv6 | Leave Blank
| IPv4/CIDR | Leave Blank |
| Gateway (IPv6) | Leave Blank |
| **DNS**
| DNS domain | Leave Default (use host settings)
| DNS servers | Leave Default (use host settings)
| **Confirm**
| Start after Created | `☑`

And Click `Finish` to create your PiHole LXC.

Or if you prefer you can simply use Proxmox CLI `typhoon-01` >  `>_ Shell` and type the following to achieve the same thing (note, you will need to create a password for PiHole LXC):
```
pct create 254 local:vztmpl/centos-7-default_20171212_amd64.tar.xz --arch amd64 --cores 1 --hostname pihole --cpulimit 1 --cpuunits 1024 --memory 256 --net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.1.5,ip=192.168.1.254/24,type=veth --ostype centos --rootfs typhoon-share:8 --swap 256 --unprivileged 1 --onboot 1 --startup order=1 --password
```

### 1.2 Install PiHole
First Start your `254 (pihole)` LXC container using the web interface `Datacenter` > `254 (pihole)` > `Start`. Then login into your `254 (pihole)` LXC by going to  `Datacenter` > `254 (pihole)` > `>_ Console and logging in with username `root` and the password you created in the previous step 1.1.

Now using the web interface `Datacenter` > `254 (pihole)` > `>_ Console` run the following command:
```
curl -sSL https://install.pi-hole.net | bash
```
The PiHole installation package will download and the installation will commence. Follow the prompts making sure to enter the prompts and field values as follows:

| PiHole Installation | Value | Notes
| :---  | :---: | :--- |
| PiHole automated installer | `<OK>` | *Just hit your ENTER key*
| Free and open source | `<OK>` | *Just hit your ENTER key*
| Static IP Needed | `<OK>` | *Just hit your ENTER key*
| Select UPstream DNS Provider | `Cloudfare` | *And tab key to highlight <OK> and hit your ENTER key*
| Pihole relies on third party .... | Leave Default, all selected | *And tab key to highlight <OK> and hit your ENTER key*
| Select Protocols | Leave default, all selected | *And tab key to highlight <OK> and hit your ENTER key*
| Static IP Address | Leave Default | *It should show IP Address: 192.168.1.254/24, and Gateway: 192.168.1.5. And tab key to highlight <Yes> and hit your ENTER key*
| FYI: IP Conflict | Nothing to do here | *And tab key to highlight <OK> and hit your ENTER key*
| Do you wish to install the web admin interface | `☑` On (Recommended) | *And tab key to highlight <OK> and hit your ENTER key*
| Do you wish to install the web server |  `☑` On (Recommended) | *And tab key to highlight <OK> and hit your ENTER key*
| Do you want to log queries? |  `☑` On (Recommended) | *And tab key to highlight <OK> and hit your ENTER key*
| Select a privacy mode for FTL |  `☑` 0 Show Everything  | *And tab key to highlight <OK> and hit your ENTER key*
| **And the installation script will commence ...**
| Installation Complete | `<OK>` | *Just hit your ENTER key*

Your installation should be complete.

### 1.3 Reset your PiHole webadmin password
Now reset the web admin password using the web interface `Datacenter` > `254 (pihole)` > `>_ Console` run the following command:
```
pihole -a -p
```
You can now login to your PiHole server using your preferred web browser with the following URL http://192.168.1.254/admin/index.php

### 1.4 Enable DNSSEC
You can enable DNSSEC when using Cloudfare which support DNSSEC. Using the PiHole webadmin URL http://192.168.1.254/admin/index.php go to `Settings` > `DNS Tab` and enable `USE DNSSEC` under Advanced DNS Settings. Click `Save`.

## 2.0 UniFi Controller - CentOS7
Rather than buy a UniFi Cloud Key to securely run a instance of the UniFi Controller software you can use Proxmox LXC container to host your UniFi Controller software.

For this we will use a CentOS LXC container.

### 2.1 Create a CentOS7 LXC for UniFi Controller
Now using the web interface `Datacenter` > `Create CT` and fill out the details as shown below (whats not shown below leave as default):

| Create: LXC Container | Value |
| :---  | :---: |
| **General**
| Node | `typhoon-01` |
| CT ID |`251`|
| Hostname |`unifi`|
| Unprivileged container | `☑` |
| Resource Pool | Leave Blank
| Password | Enter your pasword
| Password | Enter your pasword
| SSH Public key | Add one if you want to
| **Template**
| Storage | `local` |
| Template |`centos-7-default_xxxx_amd`|
| **Root Disk**
| Storage |`typhoon-share`|
| Disk Size |`8 GiB`|
| **CPU**
| Cores |`1`|
| CPU limit | Leave Blank
| CPU Units | `1024`
| **Memory**
| Memory (MiB) |`1024`|
| Swap (MiB) |`256`|
| **Network**
| Name | `eth0`
| Mac Address | `auto`
| Bridge | `vmbr0`
| VLAN Tag | Leave Blank
| Rate limit (MN/s) | Leave Default (unlimited)
| Firewall | `☑`
| IPv4 | `☑  Static`
| IPv4/CIDR |`192.168.1.251/24`|
| Gateway (IPv4) |`192.168.1.5`|
| IPv6 | Leave Blank
| IPv4/CIDR | Leave Blank |
| Gateway (IPv6) | Leave Blank |
| **DNS**
| DNS domain | Leave Default (use host settings)
| DNS servers | Leave Default (use host settings)
| **Confirm**
| Start after Created | `☑`

And Click `Finish` to create your UniFi LXC.

Or if you prefer you can simply use Proxmox CLI `typhoon-01` >  `>_ Shell` and type the following to achieve the same thing (note, you will need to create a password for UniFi LXC):
```
pct create 251 local:vztmpl/centos-7-default_20171212_amd64.tar.xz --arch amd64 --cores 1 --hostname unifi --cpulimit 1 --cpuunits 1024 --memory 1024 --net0 name=eth0,bridge=vmbr0,firewall=1,gw=192.168.1.5,ip=192.168.1.251/24,type=veth --ostype centos --rootfs typhoon-share:8 --swap 256 --unprivileged 1 --onboot 1 --startup order=1 --password
```

**Note:** test CentOS UniFi package listing is available [HERE](https://community.ui.com/questions/Unofficial-RHEL-CentOS-UniFi-Controller-rpm-packages/a5db143e-e659-4137-af8d-735dfa53e36d).

### 2.2 Install UniFi
First Start your `251 (unifi)` LXC container using the web interface `Datacenter` > `251 (unifi)` > `Start`. Then login into your `251 (unifi)` LXC by going to  `Datacenter` > `251 (unifi)` > `>_ Console and logging in with username `root` and the password you created in the previous step 2.1.

Now using the web interface `Datacenter` > `251 (unifi)` > `>_ Console` run the following command:

```
yum install epel-release -y &&
yum install http://dl.marmotte.net/rpms/redhat/el7/x86_64/unifi-controller-5.8.24-1.el7/unifi-controller-5.8.24-1.el7.x86_64.rpm -y &&
systemctl enable unifi.service &&
systemctl start unifi.service
```

### 2.3 Move the UniFi Controller to your LXC Instance
You can backup the current configuration and move it to a different computer.

Take a backup of the existing controller using the UniFi WebGUI interface and go to `Settings` > `Maintenance` > `Backup` > `Download Backup`. This will create a `xxx.unf` file format to be saved at your selected destination on your PC (i.e Downloads).

Now on your Proxmox UniFi LXC, https://192.168.1.251:8443/ , you must restore the downloaded backup unf file to the new machine by going to `Settings` > `Maintenance` > `Restore` > `Choose File` and selecting the unf file saved on your local PC.

But make sure when you are restoring the backup you Have closed the previous UniFi Controller server and software because you cannot manage the APs by two controller at a time.

## 3.0 Jellyfin LXC - CentOS
>  **Under development.** I could'nt get Jellyfin working with a Proxmox CentOS7 LXC. The problems is with the VAAPI GPU Passthru. Also Jellyfin frontend WebGUI restart button function fails. So best use my Ubuntu LXC recipe [HERE](https://github.com/ahuacate/proxmox-lxc/blob/master/README.md#40-jellyfin-lxc---ubuntu-1604).

Jellyfin is an alternative to the proprietary Emby and Plex, to provide media from a dedicated server to end-user devices via multiple apps. 

Jellyfin is descended from Emby's 3.5.2 release and ported to the .NET Core framework to enable full cross-platform support. There are no strings attached, no premium licenses or features, and no hidden agendas: and at the time of writing this media server software seems like the best available solution (and is free).

### 3.1 Create a CentOS7 LXC for Jellyfin
Now using the web interface `Datacenter` > `Create CT` and fill out the details as shown below (whats not shown below leave as default):

| Create: LXC Container | Value |
| :---  | :---: |
| **General**
| Node | `typhoon-01` |
| CT ID |`111`|
| Hostname |`jellyfin`|
| Unprivileged container | `☐` |
| Resource Pool | Leave Blank
| Password | Enter your pasword
| Password | Enter your pasword
| SSH Public key | Add one if you want to
| **Template**
| Storage | `local` |
| Template |`centos-7-default_xxxx_amd`|
| **Root Disk**
| Storage |`typhoon-share`|
| Disk Size |`20 GiB`|
| **CPU**
| Cores |`2`|
| CPU limit | Leave Blank
| CPU Units | `1024`
| **Memory**
| Memory (MiB) |`4096`|
| Swap (MiB) |`256`|
| **Network**
| Name | `eth0`
| Mac Address | `auto`
| Bridge | `vmbr0`
| VLAN Tag | `50`
| Rate limit (MN/s) | Leave Default (unlimited)
| Firewall | `☑`
| IPv4 | `☑  Static`
| IPv4/CIDR |`192.168.50.111/24`|
| Gateway (IPv4) |`192.168.50.5`|
| IPv6 | Leave Blank
| IPv4/CIDR | Leave Blank |
| Gateway (IPv6) | Leave Blank |
| **DNS**
| DNS domain | Leave Default (use host settings)
| DNS servers | Leave Default (use host settings)
| **Confirm**
| Start after Created | `☐`

And Click `Finish` to create your JellyFin LXC. The above will create the Jellyfin LXC without any of the required local Mount Points to the host.

If you prefer you can simply use Proxmox CLI `typhoon-01` > `>_ Shell` and type the following to achieve the same thing PLUS it will automatically add the required Mount Points (note, have your root password ready for Jellyfin LXC):

**Script (A):** Including LXC Mount Points
```
pct create 111 local:vztmpl/centos-7-default_20171212_amd64.tar.xz --arch amd64 --cores 2 --hostname jellyfin --cpulimit 1 --cpuunits 1024 --memory 4096 --net0 name=eth0,bridge=vmbr0,tag=50,firewall=1,gw=192.168.50.5,ip=192.168.50.111/24,type=veth --ostype centos --rootfs typhoon-share:20 --swap 256 --unprivileged 1 --onboot 1 --startup order=2 --password --mp0 /mnt/pve/cyclone-01-music,mp=/mnt/music --mp1 /mnt/pve/cyclone-01-photo,mp=/mnt/photo --mp2 /mnt/pve/cyclone-01-transcode,mp=/mnt/transcode --mp3 /mnt/pve/cyclone-01-video,mp=/mnt/video
```
**Script (B):** Excluding LXC Mount Points:
```
pct create 111 local:vztmpl/centos-7-default_20171212_amd64.tar.xz --arch amd64 --cores 2 --hostname jellyfin --cpulimit 1 --cpuunits 1024 --memory 4096 --net0 name=eth0,bridge=vmbr0,tag=50,firewall=1,gw=192.168.50.5,ip=192.168.50.111/24,type=veth --ostype centos --rootfs typhoon-share:20 --swap 256 --unprivileged 1 --onboot 1 --startup order=2 --password
```

### 3.2 Configure and Install VAAPI - CentOS
> This section only applies to Proxmox nodes typhoon-01 and typhoon-02. **DO NOT USE ON TYPHOON-03** or any Synology/NAS Virtual Machine installed node.

Jellyfin supports hardware acceleration of video encoding/decoding/transcoding using FFMpeg. Because we are using Linux we will use Intel/AMD VAAPI.

But first you must configure VAAPI for your host system. VAAPI is configured for typhoon-01 and tyhoon-02 only because the machine hardware supports video encoding.

Your Jellyfin LXC **MUST BE** in the shutdown state before proceeding.

First verify that `render` device is present in `/dev/dri`, and note the permissions and group available to write to it, in this case `render`. Simply use Proxmox CLI `Datacenter` > `typhoon-01/02` >  `>_ Shell` and type the following first line only:

```
ls -l /dev/dri

# Results ...
total 0
crw-rw---- 1 root video 226,   0 Jul 26 14:24 card0
crw-rw---- 1 root video 226, 128 Jul 26 14:24 renderD128
```
**Note:** On some releases, the group may be `video` instead of `render`.

Now you want to install VAINFO on Proxmox nodes typhoon-01 and typhoon-02. Go to Proxmox CLI `Datacenter` > `typhoon-01/02` >  `>_ Shell` and type the following:
```
apt install vainfo -y
```

To validate your installation go to Proxmox CLI `Datacenter` > `typhoon-01/02` >  `>_ Shell` and type `vainfo` and the results should be similiar to whats shown below:
```
@typhoon-01:~# vainfo
error: XDG_RUNTIME_DIR not set in the environment.
error: can't connect to X server!
libva info: VA-API version 0.39.4
libva info: va_getDriverName() returns 0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so
libva info: Found init function __vaDriverInit_0_39
libva info: va_openDriver() returns 0
vainfo: VA-API version: 0.39 (libva 1.7.3)
vainfo: Driver version: Intel i965 driver for Intel(R) Kabylake - 1.7.3
vainfo: Supported profile and entrypoints
      VAProfileMPEG2Simple            : VAEntrypointVLD
      VAProfileMPEG2Simple            : VAEntrypointEncSlice
      VAProfileMPEG2Main              : VAEntrypointVLD
      VAProfileMPEG2Main              : VAEntrypointEncSlice
      VAProfileH264ConstrainedBaseline: VAEntrypointVLD
      VAProfileH264ConstrainedBaseline: VAEntrypointEncSlice
      VAProfileH264Main               : VAEntrypointVLD
      VAProfileH264Main               : VAEntrypointEncSlice
      VAProfileH264High               : VAEntrypointVLD
      VAProfileH264High               : VAEntrypointEncSlice
      VAProfileH264MultiviewHigh      : VAEntrypointVLD
      VAProfileH264MultiviewHigh      : VAEntrypointEncSlice
      VAProfileH264StereoHigh         : VAEntrypointVLD
      VAProfileH264StereoHigh         : VAEntrypointEncSlice
      VAProfileVC1Simple              : VAEntrypointVLD
      VAProfileVC1Main                : VAEntrypointVLD
      VAProfileVC1Advanced            : VAEntrypointVLD
      VAProfileNone                   : VAEntrypointVideoProc
      VAProfileJPEGBaseline           : VAEntrypointVLD
      VAProfileJPEGBaseline           : VAEntrypointEncPicture
      VAProfileVP8Version0_3          : VAEntrypointVLD
      VAProfileVP8Version0_3          : VAEntrypointEncSlice
      VAProfileHEVCMain               : VAEntrypointVLD
      VAProfileHEVCMain               : VAEntrypointEncSlice
      VAProfileHEVCMain10             : VAEntrypointVLD
      VAProfileHEVCMain10             : VAEntrypointEncSlice
      VAProfileVP9Profile0            : VAEntrypointVLD
      VAProfileVP9Profile0            : VAEntrypointEncSlice
      VAProfileVP9Profile2            : VAEntrypointVLD
```

### 3.3 Grant Jellyfin LXC Container access to the Proxmox host video device - CentOS
> This section only applies to Proxmox nodes typhoon-01 and typhoon-02. **DO NOT USE ON TYPHOON-03** or any Synology/NAS Virtual Machine installed node.

Here we edit the LXC configuration file with the line `lxc.cgroup.devices.allow` to declare your hardmetal GPU device to your Jellyfin LXC container so it can access your hosts GPU.

The command `lxc.cgroup.devices.allow: c 226:128 rwm` means its allowing Jellyfin LXC container to rwm (read/write/mount) your GPU device (Proxmox host) which has the major number of 226 and minor number of 128.

Granting the permission alone is not enough if the device is not present in Jellyfins LXC container's /dev directory. The second step is create corresponding mount points in the LXC container to your hosts /dev/dri/renderD128 folder.

Please note your Proxmox Jellyfin LXC **MUST BE** in the shutdown state before proceeding.

Now using the web interface go to Proxmox CLI `Datacenter` > `typhoon-01` >  `>_ Shell` and type the following:

```
echo -e "lxc.cgroup.devices.allow = c 226:128 rwm
lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file" >> /etc/pve/lxc/111.conf
```

### 3.4 Install Jellyfin - CentOS
This is easy. First start LXC 111 (jellyfin) with the Proxmox web interface go to `typhoon-01` > `111 (jellyfin)` > `START`.

Then with the Proxmox web interface go to `typhoon-01` > `111 (jellyfin)` > `>_ Shell` and type the following:

```
yum -y install epel-release &&
rpm -v --import http://li.nux.ro/download/nux/RPM-GPG-KEY-nux.ro &&
rpm -Uvh http://li.nux.ro/download/nux/dextop/el7/x86_64/nux-dextop-release-0-5.el7.nux.noarch.rpm &&
yum -y install ffmpeg ffmpeg-devel &&
yum -y install https://repo.jellyfin.org/releases/server/centos/jellyfin-10.3.7-1.el7.x86_64.rpm &&
systemctl enable jellyfin &&
systemctl start jellyfin
```

### 3.5 Setup Jellyfin Mount Points - CentOS
If you used **Script (B)** in Section 3.1 then you have no Moint Points.

Please note your Proxmox Jellyfin LXC **MUST BE** in the shutdown state before proceeding.

To create the Mount Points use the web interface go to Proxmox CLI `Datacenter` > `typhoon-01` > `>_ Shell` and type the following:
```
pct set 111 -mp0 /mnt/pve/cyclone-01-music,mp=/mnt/music &&
pct set 111 -mp1 /mnt/pve/cyclone-01-photo,mp=/mnt/photo &&
pct set 111 -mp2 /mnt/pve/cyclone-01-transcode,mp=/mnt/transcode &&
pct set 111 -mp3 /mnt/pve/cyclone-01-video,mp=/mnt/video
```

## 4.0 Jellyfin LXC - Ubuntu 18.04
>  This 100% works. 

Jellyfin is an alternative to the proprietary Emby and Plex, to provide media from a dedicated server to end-user devices via multiple apps. 

Jellyfin is descended from Emby's 3.5.2 release and ported to the .NET Core framework to enable full cross-platform support. There are no strings attached, no premium licenses or features, and no hidden agendas: and at the time of writing this media server software seems like the best available solution (and is free).

### 4.1 Download the Ubuntu LXC template - Ubuntu 18.04
First you need to add Ubuntu 18.04 LXC to Proxmox templates. Now using the Proxmox web interface `Datacenter` > `typhoon-01` >`Local (typhoon-01)` > `Content` > `Templates`  select `ubuntu-18.04-standard` LXC and click `Download`.

Or use a Proxmox typhoon-01 CLI `>_ Shell` and type the following:
```
wget  http://download.proxmox.com/images/system/ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz -P /var/lib/vz/template/cache && gzip -d /var/lib/vz/template/cache/ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz
```

### 4.2 Create a Ubuntu 18.04 LXC for Jellyfin - Ubuntu 18.04
Now using the Proxmox web interface `Datacenter` > `Create CT` and fill out the details as shown below (whats not shown below leave as default):

| Create: LXC Container | Value |
| :---  | :---: |
| **General**
| Node | `typhoon-01` |
| CT ID |`111`|
| Hostname |`jellyfin`|
| Unprivileged container | `☑` |
| Resource Pool | Leave Blank
| Password | Enter your pasword
| Password | Enter your pasword
| SSH Public key | Add one if you want to
| **Template**
| Storage | `local` |
| Template |`ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz`|
| **Root Disk**
| Storage |`typhoon-share`|
| Disk Size |`20 GiB`|
| **CPU**
| Cores |`2`|
| CPU limit | Leave Blank
| CPU Units | `1024`
| **Memory**
| Memory (MiB) |`4096`|
| Swap (MiB) |`256`|
| **Network**
| Name | `eth0`
| Mac Address | `auto`
| Bridge | `vmbr0`
| VLAN Tag | `50`
| Rate limit (MN/s) | Leave Default (unlimited)
| Firewall | `☑`
| IPv4 | `☑  Static`
| IPv4/CIDR |`192.168.50.111/24`|
| Gateway (IPv4) |`192.168.50.5`|
| IPv6 | Leave Blank
| IPv4/CIDR | Leave Blank |
| Gateway (IPv6) | Leave Blank |
| **DNS**
| DNS domain | Leave Default (use host settings)
| DNS servers | Leave Default (use host settings)
| **Confirm**
| Start after Created | `☐`

And Click `Finish` to create your JellyFin LXC. The above will create the Jellyfin LXC without any of the required local Mount Points to the host.

If you prefer you can simply use Proxmox CLI `typhoon-01` > `>_ Shell` and type the following to achieve the same thing PLUS it will automatically add the required Mount Points (note, have your root password ready for Jellyfin LXC):

**Script (A):** Including LXC Mount Points
```
pct create 111 local:vztmpl/ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz --arch amd64 --cores 2 --hostname jellyfin --cpulimit 1 --cpuunits 1024 --memory 4096 --net0 name=eth0,bridge=vmbr0,tag=50,firewall=1,gw=192.168.50.5,ip=192.168.50.111/24,type=veth --ostype centos --rootfs typhoon-share:20 --swap 256 --unprivileged 1 --onboot 1 --startup order=2 --password --mp0 /mnt/pve/cyclone-01-music,mp=/mnt/music --mp1 /mnt/pve/cyclone-01-photo,mp=/mnt/photo --mp2 /mnt/pve/cyclone-01-transcode,mp=/mnt/transcode --mp3 /mnt/pve/cyclone-01-video,mp=/mnt/video
```

**Script (B):** Excluding LXC Mount Points:
```
pct create 111 local:vztmpl/ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz --arch amd64 --cores 2 --hostname jellyfin --cpulimit 1 --cpuunits 1024 --memory 4096 --net0 name=eth0,bridge=vmbr0,tag=50,firewall=1,gw=192.168.50.5,ip=192.168.50.111/24,type=veth --ostype centos --rootfs typhoon-share:20 --swap 256 --unprivileged 1 --onboot 1 --startup order=2 --password
```

### 4.3 Configure and Install VAAPI - Ubuntu 18.04
> This section only applies to Proxmox nodes typhoon-01 and typhoon-02. **DO NOT USE ON TYPHOON-03** or any Synology/NAS Virtual Machine installed node.

Jellyfin supports hardware acceleration of video encoding/decoding/transcoding using FFMpeg. Because we are using Linux we will use Intel/AMD VAAPI.

But first you must configure VAAPI for your host system. VAAPI is configured for typhoon-01 and tyhoon-02 only because the machine hardware supports video encoding.

Your Jellyfin LXC **MUST BE** in the shutdown state before proceeding.

First verify that `render` device is present in `/dev/dri`, and note the permissions and group available to write to it, in this case `render`. Simply use Proxmox CLI `Datacenter` > `typhoon-01/02` >  `>_ Shell` and type the following first line only:

```
ls -l /dev/dri

# Results ...
total 0
crw-rw---- 1 root video 226,   0 Jul 26 14:24 card0
crw-rw---- 1 root video 226, 128 Jul 26 14:24 renderD128
```
**Note:** On some releases, the group may be `video` instead of `render`.

Now you want to install VAINFO on Proxmox nodes typhoon-01 and typhoon-02. Go to Proxmox CLI `Datacenter` > `typhoon-01/02` >  `>_ Shell` and type the following:
```
apt install vainfo -y
chmod 666 /dev/dri/renderD128
```

To validate your installation go to Proxmox CLI `Datacenter` > `typhoon-01/02` >  `>_ Shell` and type `vainfo` and the results should be similiar to whats shown below:
```
@typhoon-01:~# vainfo
error: XDG_RUNTIME_DIR not set in the environment.
error: can't connect to X server!
libva info: VA-API version 0.39.4
libva info: va_getDriverName() returns 0
libva info: Trying to open /usr/lib/x86_64-linux-gnu/dri/i965_drv_video.so
libva info: Found init function __vaDriverInit_0_39
libva info: va_openDriver() returns 0
vainfo: VA-API version: 0.39 (libva 1.7.3)
vainfo: Driver version: Intel i965 driver for Intel(R) Kabylake - 1.7.3
vainfo: Supported profile and entrypoints
      VAProfileMPEG2Simple            : VAEntrypointVLD
      VAProfileMPEG2Simple            : VAEntrypointEncSlice
      VAProfileMPEG2Main              : VAEntrypointVLD
      VAProfileMPEG2Main              : VAEntrypointEncSlice
      VAProfileH264ConstrainedBaseline: VAEntrypointVLD
      VAProfileH264ConstrainedBaseline: VAEntrypointEncSlice
      VAProfileH264Main               : VAEntrypointVLD
      VAProfileH264Main               : VAEntrypointEncSlice
      VAProfileH264High               : VAEntrypointVLD
      VAProfileH264High               : VAEntrypointEncSlice
      VAProfileH264MultiviewHigh      : VAEntrypointVLD
      VAProfileH264MultiviewHigh      : VAEntrypointEncSlice
      VAProfileH264StereoHigh         : VAEntrypointVLD
      VAProfileH264StereoHigh         : VAEntrypointEncSlice
      VAProfileVC1Simple              : VAEntrypointVLD
      VAProfileVC1Main                : VAEntrypointVLD
      VAProfileVC1Advanced            : VAEntrypointVLD
      VAProfileNone                   : VAEntrypointVideoProc
      VAProfileJPEGBaseline           : VAEntrypointVLD
      VAProfileJPEGBaseline           : VAEntrypointEncPicture
      VAProfileVP8Version0_3          : VAEntrypointVLD
      VAProfileVP8Version0_3          : VAEntrypointEncSlice
      VAProfileHEVCMain               : VAEntrypointVLD
      VAProfileHEVCMain               : VAEntrypointEncSlice
      VAProfileHEVCMain10             : VAEntrypointVLD
      VAProfileHEVCMain10             : VAEntrypointEncSlice
      VAProfileVP9Profile0            : VAEntrypointVLD
      VAProfileVP9Profile0            : VAEntrypointEncSlice
      VAProfileVP9Profile2            : VAEntrypointVLD
```
### 4.4 Create a rc.local
For FFMPEG to work we must create a script to `chmod 666 /dev/dri/renderD128` everytime a Proxmox host boots. Now using the web interface go to Proxmox CLI `Datacenter` > `typhoon-01/02` >  `>_ Shell` and type the following:
```
echo '#!/bin/sh -e
/bin/chmod 666 /dev/dri/renderD128
exit 0' > /etc/rc.local &&
chmod +x /etc/rc.local
```

### 4.5 Grant Jellyfin LXC Container access to the Proxmox host video device - Ubuntu 18.04
> This section only applies to Proxmox nodes typhoon-01 and typhoon-02. **DO NOT USE ON TYPHOON-03** or any Synology/NAS Virtual Machine installed node.

Here we edit the LXC configuration file with the line `lxc.cgroup.devices.allow` to declare your hardmetal GPU device to your Jellyfin LXC container so it can access your hosts GPU.

The command `lxc.cgroup.devices.allow: c 226:128 rwm` means its allowing Jellyfin LXC container to rwm (read/write/mount) your GPU device (Proxmox host) which has the major number of 226 and minor number of 128.

Granting the permission alone is not enough if the device is not present in Jellyfins LXC container's /dev directory. The second step is create corresponding mount points in the LXC container to your hosts /dev/dri/renderD128 folder.

Please note your Proxmox Jellyfin LXC **MUST BE** in the shutdown state before proceeding.

Now using the web interface go to Proxmox CLI `Datacenter` > `typhoon-01` >  `>_ Shell` and type the following:

```
echo -e "lxc.cgroup.devices.allow = c 226:128 rwm
lxc.cgroup.devices.allow = c 226:0 rwm
lxc.mount.entry: /dev/dri/renderD128 dev/dri/renderD128 none bind,optional,create=file" >> /etc/pve/lxc/111.conf
```

### 4.6 Install Jellyfin - Ubuntu 18.04
This is easy. First start LXC 111 (jellyfin) with the Proxmox web interface go to `typhoon-01` > `111 (jellyfin)` > `START`.

Then with the Proxmox web interface go to `typhoon-01` > `111 (jellyfin)` > `>_ Shell` and type the following:

```
sudo apt update -y &&
sudo apt install apt-transport-https &&
sudo apt install gnupg gnupg2 gnupg1 -y &&
wget -O - https://repo.jellyfin.org/ubuntu/jellyfin_team.gpg.key | sudo apt-key add - &&
echo "deb [arch=$( dpkg --print-architecture )] https://repo.jellyfin.org/ubuntu $( lsb_release -c -s ) main" | sudo tee /etc/apt/sources.list.d/jellyfin.list &&
sudo apt update -y &&
sudo apt install jellyfin -y &&
sudo systemctl restart jellyfin
```

### 4.7 Setup Jellyfin Mount Points - Ubuntu 18.04
If you used **Script (B)** in Section 4.2 then you have no Moint Points.

Please note your Proxmox Jellyfin LXC **MUST BE** in the shutdown state before proceeding.

To create the Mount Points use the web interface go to Proxmox CLI `Datacenter` > `typhoon-01` > `>_ Shell` and type the following:
```
pct set 111 -mp0 /mnt/pve/cyclone-01-music,mp=/mnt/music &&
pct set 111 -mp1 /mnt/pve/cyclone-01-photo,mp=/mnt/photo &&
pct set 111 -mp2 /mnt/pve/cyclone-01-transcode,mp=/mnt/transcode &&
pct set 111 -mp3 /mnt/pve/cyclone-01-video,mp=/mnt/video
```

### 4.8 Check your Jellyfin Installation
In your web browser type `http://192.168.50.111:8096` and you should see a Jellyfin configuration wizard page.

## 5.0 Sonarr LXC - Ubuntu 18.04
Sonarr is a PVR for Usenet and BitTorrent users. It can monitor multiple RSS feeds for new episodes of your favorite shows and will grab, sort and rename them. It can also be configured to automatically upgrade the quality of files already downloaded when a better quality format becomes available.

### 5.1 Create a Ubuntu 18.04 LXC for Sonarr
Now using the web interface `Datacenter` > `Create CT` and fill out the details as shown below (whats not shown below leave as default):

| Create: LXC Container | Value |
| :---  | :---: |
| **General**
| Node | `typhoon-01` |
| CT ID |`112`|
| Hostname |`sonarr`|
| Unprivileged container | `☑` |
| Resource Pool | Leave Blank
| Password | Enter your pasword
| Password | Enter your pasword
| SSH Public key | Add one if you want to
| **Template**
| Storage | `local` |
| Template | `ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz` |
| **Root Disk**
| Storage |`typhoon-share`|
| Disk Size |`10 GiB`|
| **CPU**
| Cores |`1`|
| CPU limit | Leave Blank
| CPU Units | `1024`
| **Memory**
| Memory (MiB) |`2048`|
| Swap (MiB) |`256`|
| **Network**
| Name | `eth0`
| Mac Address | `auto`
| Bridge | `vmbr0`
| VLAN Tag | `50`
| Rate limit (MN/s) | Leave Default (unlimited)
| Firewall | `☑`
| IPv4 | `☑  Static`
| IPv4/CIDR |`192.168.50.112/24`|
| Gateway (IPv4) |`192.168.50.5`|
| IPv6 | Leave Blank
| IPv4/CIDR | Leave Blank |
| Gateway (IPv6) | Leave Blank |
| **DNS**
| DNS domain | Leave Default (use host settings)
| DNS servers | Leave Default (use host settings)
| **Confirm**
| Start after Created | `☐`

And Click `Finish` to create your Sonarr LXC. The above will create the Sonarr LXC without any of the required local Mount Points to the host.

If you prefer you can simply use Proxmox CLI `typhoon-01` > `>_ Shell` and type the following to achieve the same thing PLUS it will automatically add the required Mount Points (note, have your root password ready for Sonarr LXC):

**Script (A):** Including LXC Mount Points
```
pct create 112 local:vztmpl/ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz --arch amd64 --cores 1 --hostname sonarr --cpulimit 1 --cpuunits 1024 --memory 2048 --net0 name=eth0,bridge=vmbr0,tag=50,firewall=1,gw=192.168.50.5,ip=192.168.50.112/24,type=veth --ostype centos --rootfs typhoon-share:10 --swap 256 --unprivileged 1 --onboot 1 --startup order=3 --password --mp0 /mnt/pve/cyclone-01-video,mp=/mnt/video --mp1 /typhoon-share/downloads,mp=/mnt/downloads --mp2 /mnt/pve/cyclone-01-backup,mp=/mnt/backup
```

**Script (B):** Excluding LXC Mount Points:
```
pct create 112 local:vztmpl/ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz --arch amd64 --cores 1 --hostname sonarr --cpulimit 1 --cpuunits 1024 --memory 2048 --net0 name=eth0,bridge=vmbr0,tag=50,firewall=1,gw=192.168.50.5,ip=192.168.50.112/24,type=veth --ostype centos --rootfs typhoon-share:10 --swap 256 --unprivileged 1 --onboot 1 --startup order=2 --password
```

### 5.2 Setup Sonarr Mount Points - Ubuntu 18.04
If you used **Script (B)** in Section 5.1 then you have no Moint Points.

Please note your Proxmox Sonarr LXC **MUST BE** in the shutdown state before proceeding.

To create the Mount Points use the web interface go to Proxmox CLI `Datacenter` > `typhoon-01` > `>_ Shell` and type the following:
```
pct set 112 -mp0 /mnt/pve/cyclone-01-video,mp=/mnt/video &&
pct set 112 -mp1 /typhoon-share/downloads,mp=/mnt/downloads &&
pct set 112 -mp2 /mnt/pve/cyclone-01-backup,mp=/mnt/backup
```

### 5.3 Install Sonarr
Th following Sonarr installation recipe is from the official Sonarr website [HERE](https://sonarr.tv/#downloads-v3-linux-ubuntu). Please refer for the latest updates.

During the installation, you will be asked which user and group Sonarr must run as. It's important to choose these correctly to avoid permission issues with your media files. I suggest you keep at least the group named `homelab` and username `storm` identical between your download client(s) and Sonarr. 

First start your Sonarr LXC and login. Then go to the Proxmox web interface `typhoon-01` > `112 (sonarr)` > `>_ Shell` and type the following:

```
sudo apt-get update -y &&
sudo apt install gnupg ca-certificates -y &&
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF &&
echo "deb https://download.mono-project.com/repo/ubuntu stable-bionic main" | sudo tee /etc/apt/sources.list.d/mono-official-stable.list &&
sudo apt update -y &&
wget https://mediaarea.net/repo/deb/repo-mediaarea_1.0-9_all.deb && dpkg -i repo-mediaarea_1.0-9_all.deb && apt-get update -y &&
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 2009837CBFFD68F45BC180471F4F90DE2A9B4BF8 &&
echo "deb https://apt.sonarr.tv/ubuntu bionic main" | sudo tee /etc/apt/sources.list.d/sonarr.list &&
sudo apt update -y &&
sudo apt install sonarr -y
```

Browse to http://192.168.50.112:8989 to start using Sonarr.

## 6.0 Radarr LXC - Ubuntu 18.04
Sonarr is a PVR for Usenet and BitTorrent users. It can monitor multiple RSS feeds for new episodes of your favorite shows and will grab, sort and rename them. It can also be configured to automatically upgrade the quality of files already downloaded when a better quality format becomes available.

### 6.1 Create a Ubuntu 18.04 LXC for Radarr
Now using the web interface `Datacenter` > `Create CT` and fill out the details as shown below (whats not shown below leave as default):

| Create: LXC Container | Value |
| :---  | :---: |
| **General**
| Node | `typhoon-01` |
| CT ID |`113`|
| Hostname |`radarr`|
| Unprivileged container | `☑` |
| Resource Pool | Leave Blank
| Password | Enter your pasword
| Password | Enter your pasword
| SSH Public key | Add one if you want to
| **Template**
| Storage | `local` |
| Template | `ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz` |
| **Root Disk**
| Storage |`typhoon-share`|
| Disk Size |`10 GiB`|
| **CPU**
| Cores |`1`|
| CPU limit | Leave Blank
| CPU Units | `1024`
| **Memory**
| Memory (MiB) |`2048`|
| Swap (MiB) |`256`|
| **Network**
| Name | `eth0`
| Mac Address | `auto`
| Bridge | `vmbr0`
| VLAN Tag | `50`
| Rate limit (MN/s) | Leave Default (unlimited)
| Firewall | `☑`
| IPv4 | `☑  Static`
| IPv4/CIDR |`192.168.50.113/24`|
| Gateway (IPv4) |`192.168.50.5`|
| IPv6 | Leave Blank
| IPv4/CIDR | Leave Blank |
| Gateway (IPv6) | Leave Blank |
| **DNS**
| DNS domain | Leave Default (use host settings)
| DNS servers | Leave Default (use host settings)
| **Confirm**
| Start after Created | `☐`

And Click `Finish` to create your Radarr LXC. The above will create the Radarr LXC without any of the required local Mount Points to the host.

If you prefer you can simply use Proxmox CLI `typhoon-01` > `>_ Shell` and type the following to achieve the same thing PLUS it will automatically add the required Mount Points (note, have your root password ready for Radarr LXC):

**Script (A):** Including LXC Mount Points
```
pct create 113 local:vztmpl/ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz --arch amd64 --cores 1 --hostname radarr --cpulimit 1 --cpuunits 1024 --memory 2048 --net0 name=eth0,bridge=vmbr0,tag=50,firewall=1,gw=192.168.50.5,ip=192.168.50.113/24,type=veth --ostype centos --rootfs typhoon-share:10 --swap 256 --unprivileged 1 --onboot 1 --startup order=3 --password --mp0 /mnt/pve/cyclone-01-video,mp=/mnt/video --mp1 /typhoon-share/downloads,mp=/mnt/downloads --mp2 /mnt/pve/cyclone-01-backup,mp=/mnt/backup
```

**Script (B):** Excluding LXC Mount Points:
```
pct create 113 local:vztmpl/ubuntu-18.04-standard_18.04.1-1_amd64.tar.gz --arch amd64 --cores 1 --hostname radarr --cpulimit 1 --cpuunits 1024 --memory 2048 --net0 name=eth0,bridge=vmbr0,tag=50,firewall=1,gw=192.168.50.5,ip=192.168.50.113/24,type=veth --ostype centos --rootfs typhoon-share:10 --swap 256 --unprivileged 1 --onboot 1 --startup order=2 --password
```

### 5.2 Setup Radarr Mount Points - Ubuntu 18.04
If you used **Script (B)** in Section 5.1 then you have no Moint Points.

Please note your Proxmox Radarr LXC **MUST BE** in the shutdown state before proceeding.

To create the Mount Points use the web interface go to Proxmox CLI `Datacenter` > `typhoon-01` > `>_ Shell` and type the following:
```
pct set 113 -mp0 /mnt/pve/cyclone-01-video,mp=/mnt/video &&
pct set 113 -mp1 /typhoon-share/downloads,mp=/mnt/downloads &&
pct set 113 -mp2 /mnt/pve/cyclone-01-backup,mp=/mnt/backup
```

### 5.3 Install Radarr
First start your Radarr LXC and login. Then go to the Proxmox web interface `typhoon-01` > `113 (radarr)` > `>_ Shell` and insert by cut & pasting the following:

```
sudo apt-get update -y &&
sudo apt install gnupg ca-certificates -y &&
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF &&
echo "deb https://download.mono-project.com/repo/ubuntu stable-bionic main" | sudo tee /etc/apt/sources.list.d/mono-official-stable.list &&
sudo apt update -y &&
sudo apt install mono-devel -y &&
cd /opt &&
curl -L -O $( curl -s https://api.github.com/repos/Radarr/Radarr/releases | grep linux.tar.gz | grep browser_download_url | head -1 | cut -d \" -f 4 ) &&
tar -xvzf Radarr.develop.*.linux.tar.gz &&
rm *.linux.tar.gz &&
sudo chown -R root:root /opt/Radarr &&

echo -e "[Unit]
Description=Radarr Daemon
After=syslog.target network.target

[Service]
# Change the user and group variables here.
User=root
Group=root

Type=simple

# Change the path to Radarr or mono here if it is in a different location for you.
ExecStart=/usr/bin/mono --debug /opt/Radarr/Radarr.exe -nobrowser
TimeoutStopSec=20
KillMode=process
Restart=on-failure

# These lines optionally isolate (sandbox) Radarr from the rest of the system.
# Make sure to add any paths it might use to the list below (space-separated).
#ReadWritePaths=/opt/Radarr /path/to/movies/folder
#ProtectSystem=strict
#PrivateDevices=true
#ProtectHome=true

[Install]
WantedBy=multi-user.target" > /etc/systemd/system/radarr.service &&
sudo systemctl enable radarr.service &&
sudo systemctl start radarr.service &&
sudo reboot
```

Browse to http://192.168.50.112:8989 to start using Sonarr.
