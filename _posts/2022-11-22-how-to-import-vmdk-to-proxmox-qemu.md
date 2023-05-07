---
title: How to import .vmdk to Proxmox (qemu)
description: How to import .vmdk disk file to Proxmox by converting it to .raw using qemu tools. 
layout: post
permalink: how-to-import-vmdk-to-proxmox-qemu
image: "img/vmdk-proxmox-qemu/vmdk-proxmox-qemu.jpeg"
---
This is a short post on how to easily and quickly import a .vmdk disk type into your qemu powered virtualization manager (like Proxmox). While the solution may look VERY simple, this actually took me about half of a day to research/test/fail/troubleshoot/etc., so I really hope it may help other people (and me in future, when coming back to this post).


## Short intro 

While doing my on-demand SANS FOR509 training, I had to import the VM provided by them. And since there's already a Proxmox instance running on my beefed up home lab server, I thought that maybe it is a good idea to just import the VM there. Yeah, right... :D

{% include adsense.html %}

The initial file was an .iso, containing a .7z archive. The archive had 2 files: .vmdk and .vmx. The disk image (.vmdk) was about 8.8GB, with a 500GB of virtual space. After unpacking everything, it was time to start the import.


## Import .vmdk disk to Proxmox

These are the steps that I followed to properly import the .vmdk:

1. Create a new VM
    - General Tab: provide VM ID and VM Name
    - OS Tab: Select "Do not use any media" and "Other" for Guest OS Type
    - System: Leave everything by default
    - Disks: Remove any default disks (we don't need anything since we'll be importing)
    - Set CPU/Memory/Network to whatever you need
2. Copy the .vmdk file to your PVE host using **scp** (e.g.: /tmp directory should do the job)
3. Now convert and import the .vmdk disk to the newly created VM. For example, if the VM id is **509** and the .vmdk file name is **for509.vmdk**, then the command would be: 
```bash
qm importdisk 509 for509.vmdk local-lvm --format raw
```

4. IMPORTANT: Wait until the command completes executing. **qm importdisk** imports the disk relatively quickly, but then gets stuck at 100% progress for a while. DO NOT kill it!
    - I recommend running disk import command (step 3) in a separate screen session.

5. Once **qm importdisk** is finished, you need to rescan disks in order to see the newly imported disk in the Hardware section of your VM. Execute the following command: 
```bash
qm rescan
```

6. Attach the disk to VM (double click on the new disk and click "Add" in the pop-up window)

{% include adsense.html %}

That's it! You now have successfully imported a .vmdk disk into your Proxmox managed VM.