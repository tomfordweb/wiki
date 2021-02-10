# DRAC

## Helpful links
* https://archyver.blogspot.com/2016/11/the-dell-idrac6-part-1-idrac6-configuration-utility.html
# Ubuntu VM


# PVE

# Post Install


### Allow unsigned packages

Comment the offending packages in this file.

```
vim /etc/apt/sources.list.d/pve-enterprise.list

```
Add this to your `/etc/apt/sources.list`
```
# PVE pve-no-subscription repository provided by proxmox.com,
# NOT recommended for production use
deb http://download.proxmox.com/debian/pve buster pve-no-subscription
```

### PVE Shared 
Create a ZFS disk to store your pve images on.

```
Datacenter -> [host]: Disks -> ZFS -> Create: ZFS
```

