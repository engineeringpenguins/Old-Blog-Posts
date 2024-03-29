Bare Metal Server to ESXI
=========================

How (not) to convert a server into a virtual image
==================================================

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/sata-nvme/goal.png "goal")

This post is not going to be documenting how you would accomplish this task correctly, instead this post will showcase all of the ways I failed to accomplish my task (until I succeeded).

I wanted to repurpose the PC that my girlfriend was using to play games (12/24 core CPU, 47GB RAM, NVMe) for use as my server and give her my previous server (4/8 core CPU, 16GB RAM, SSD). To do this I had to backup/clone the server’s SSD to a backup 2TB drive and then verify it could boot and retain all the data on my server before cloning her NVMe to the SSD. I then verified the SSD could boot with all of her previous data. At this point the NVMe will not be talked about for the remainder of this post (assume it still has her original data on it).

This process would have been significantly easier if I had purchased a server motherboard that could read NVMe. I could have left her original drive in her new machine and left my original server drive in my new machine.

\*Disclaimer: I built the PC my Girlfriend is using so that I could play VR and 4K games before she moved in with me. I am not a monster stealing her personal PC. The server/her current PC has a 4th gen Ryzen5, DDR4, and 5700XT so more than enough to play Stardew Valley 🙂

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/sata-nvme/process-2.png "process")

My high level goal was to turn my “new” server into a hypervisor to efficiently use the additional resources. My secondary goal was to convert my Server OS (currently living on the backup drive) into a virtual image that could live on the hypervisor to avoid taking up another bare metal machine.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/sata-nvme/goals.png "goals")

My Primary goal was accomplished without any issues. I installed ESXI and created some VM’s and plan to add it to the cluster after completing my Secondary Goal.

My first thought to get a physical machine into a virtual enviornment is to create bootable media (ISO/IMG) of the host. On Windows I am a huge fam of Macrium Reflect ([link](https://www.macrium.com/reflectfree)), this is my go to tool for imaging and cloning drives and partitions. I created an image with Macrium of the first two partitions (EFI, Data) as I did not want a full 2TB backup when I am only using 200GB of data. Unfortunately Macrium images are exported as a propriatary “mrimg” format so I couldnt just boot from this like a normal .img. MR has a plethora of blog posts about how there is an mrimg to iso converter in the program. I could not find it in the current release, any previous releases I downloaded or the technician edition. At the time I did not think to just install windows on the VM and download Macrium and restore it to another partition or virtual disk.

After failing here I had a Eureka moment and realized I do not have to create an optical image, If I just want this in ESXI I can try and convert an OS into an ESXI readable virtual disk. After some Googling I found a really cool tool called vcenter converter ([link](https://customerconnect.vmware.com/downloads/info/slug/infrastructure_operations_management/vmware_vcenter_converter_standalone/6_2_0)). With this tool you can provide login credentials for any IP address and it will log in and start a live upload to whatever ESXI host you specify and also create the VM/RAM according to source.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/sata-nvme/vcenter-converter.png "vcenter converter")

This tool is fascinating, I would love to understand how it does what it does and perhaps I could make my own version. Unfortunately this product was EOL in 2017 and Ubuntu 16.x was the last version supported. I considered downgrading my OS from 20.04 but left that as a last resort. From the error provided above I infered that the Converter tool was using an outdated kernal and boot loader. I tried to boot into the VM as it was 97% and potentially viable. I hit the GNU/Grub recovery CLI which to me normally means hard failure and reflash. I tried to use an ubuntu 20.04 live CD to install grub2 instead of grub1 as well as any other boot specific functions. At this point I could hit a normal grub bootloader AND it could see my partitions/OS from earlier. I could not boot into the OS or any of the recovery modes though as it was saying there were very low level system things missing. At this point I had to move on from the migration tool, it is shocking they have not kept it in development as it seems to be invaluable.

I now go to my server with the 2TB drive attached and boot into a live CD of Ubuntu. I know that DD is the common way to clone drives in linux so I partitioned part of the NVMe drive to be able to store a backup (do not use FAT32 due to size limitations) and then used the following command from a terminal located in that backup partition:

sudo dd if=/dev/sda2 of=./ubuntubak.img

SDA2 was a partition of only ~200GB and I was trying to write into a 800GB partition (NTFS) but DD gave an error in the terminal that the filesize was too big and stopped after ~4GB. Even if this did work without the EFI partition on SDA1 I wouldnt be able to boot anyway. I found that there is no good way to only dd 2 partitions and not my third (empty/unallocated 1.8TB) but you can specify the amount of sectors to write.

sudo dd bs=1M count=300000 if=/dev/sda of=./ubtbak.img

This time it would write the whole 2TB drive but stop  after 300GB (100 extra for safety against dataloss). I got the exact same error as before and the file stopped after ~4GB. This may be related to the recovery partition being in NTFS but that should not have caused an issue.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/sata-nvme/clonezilla.png "clonezilla")

A linux SME that I know reccomended an opensource drive cloning tool called clonezilla ([link](https://clonezilla.org/)). I booted clonezilla as a liveCD and cloned partition’s 1 and 2 to my recovery partition on the other drive. I then copied the recovery file (which is actually a directory of database files) to my NAS to be accessed by the VM. I loaded clonezilla in a VM with a 500GB hard drive expecting to be able to restore the file but for some reason the clonezilla program itself was far more peared down in a VM than it is on bare metal. I could not recover from anything not locally attached. I do not have a 300GB+ USB that I could have used and to attach a drive via SATA to the ESXI host itself and then SSH into the host and add a mount point in the virtual hard drive to a partition of another drive without overwriting the data seemed excessive.

If not using in a VM I found the clonezilla clone/restore model to be excellent both used conventionally and having clonezilla on both machines and using the server/client model.

Clonezilla has a tool to convert their backup images into a .ISO file (back to square one) so I tried using this. Immediately clonezilla needed me to download a dependency for xorriso (it should have this by default, no need to add complexity to this process). It began the process of converting and when I came back to check on status it said that it completed with no errors and has read every directory. The resulting file was 0 bytes however so this failed.

![](https://github.com/engineeringpenguins/Old-Blog-Posts/tree/main/ImageReferences/sata-nvme/external-content.duckduckgo.com_.png "external-content.duckduckgo.com")

I found a tool called Rescuezilla ([link](https://rescuezilla.com/)) that seems to use Clonezilla as a base but it also has what appears to be a Debian OS on top of it. If there was some kind of issue with how an antiquated tool like clonezilla does something I wanted to see if a new tool would be more up to date with technology. Initially I tried to load the clonezilla backup with rescuezilla in a VM like before but rescuezilla said there was an issue with the backup.

I made a new backup using rescuezilla this time and stored it on my NAS. I loaded rescuezilla as a livecd on a VM and was able to access the NAS to restore the saved file. I am not sure if it was an issue with rescuezilla or my dns but when it requires a UNC path to the directory I wasnt able to connect with the hostname of the NAS I had to use ip address and then directory. It also threw some errors on previous attempts about CIFS formatting which was annoying because I use the same CIFS formatting in linux frequently.

On trying to boot from the hard drive on the VM now it was attempting to PXE boot and not able to see anything to boot from on the virtual hard drive. Using the same train of thought as before when my migration failed the bootloader process I installed a very small side partition of Ubuntu 20.04 on the same disk (dual boot) for the purpose of installing any needed boot loader technology. After doing this I can hit a grub boot menu and select the original OS and successfully login and have access to all my data previously on bare metal.
