# Installing TrueNAS Scale on Proxmox

## Downloading TrueNAS Scale ISO

1. Download the **TrueNAS Community Edition (Stable Version)** from the [official website](https://www.truenas.com/download-truenas-community-edition/). You can also right-click to copy the ISO URL instead of downloading it manually.

## Uploading the ISO to Proxmox

2. On **Proxmox**, go to your local storage > `ISO Images` > `Upload` or `Download from URL`, and upload the TrueNAS Scale `.iso` file.

## Creating a TrueNAS VM in Proxmox

3. Create a VM for TrueNAS.

4. In the **General** tab, name your VM and enable `Start at boot` under `Advanced`.

5. In the **OS** tab, select the uploaded TrueNAS `.iso`.

6. In the **Disks** tab, if you're using an SSD for storage, enable `SSD emulation` to optimize performance.

7. In the **CPU** tab, assign at least 2 cores (you can increase this depending on your hardware).

8. In the **Memory** tab, TrueNAS recommends 8 GB of RAM, but you can assign less for testing purposes.

9. In the **Confirm** tab, review the settings and check `Start after created` to power on the VM immediately.

## Installing TrueNAS Scale

10. Select the VM and open the console.

11. Once the VM boots, the TrueNAS installer will start automatically. When the boot screen appears, press `Enter` to begin installation.

12. On the next screen, select `1. Install/Upgrade`.

13. Select the disk using the space bar, then press `Enter` to continue.

14. Proceed with the installation.

15. For the Web UI Authentication Method, choose `2. Configure using Web UI`.

16. On the Legacy Boot screen, you can select `Yes` for EFI boot, or adjust it based on your hardware.

17. Continue the installation. When prompted, choose `3. Reboot System`.

18. The VM will reboot and boot directly into TrueNAS.

## First Boot & Web UI Setup

19. In your browser, navigate to the IP address displayed in the VM console.

20. Set a root password and log in to the TrueNAS Web UI.

## Attaching Physical Disks to the TrueNAS VM

21. In Proxmox, go to the `Disks` section to view all physical disks attached to the host.

22. Proxmox requires direct access to disks. Instead of using device names like `/dev/sda`, which may change, it's better to use disk `Model` and `Serial` (which are unique and persistent). You can find this info in the disk list.

23. To list disk IDs, open the Proxmox shell and run:

```shell
ls /dev/disk/by-id
```

24. Copy the full IDs of the disks and confirm their serials match the ones you intend to use.

25. In your TrueNAS VM > `Hardware`, you can now remove the installation media (CD/DVD Drive). Then attach each disk using the following command in the Proxmox shell:

```shell
qm set <VM-ID> -scsi1 /dev/disk/by-id/<Disk-ID>

qm set <VM-ID> -scsi2 /dev/disk/by-id/<Disk-ID>
```

> Repeat the command for all disks, incrementing `-scsiX` and replacing `<Disk-ID>` accordingly.

## Attaching Virtual Disks to the TrueNAS VM

26. If you don't have physical disks, go to your TrueNAS VM > `Hardware` > `Add` > `Hard Disk`, and create virtual disks (e.g., two).

## Verifying Attached Disks

27. After attaching the disks, confirm their presence in `Hardware`. Then open the TrueNAS Web UI and go to `Storage` > `Disks` to verify they appear as available drives.

## Creating a Storage Pool

28. In the TrueNAS Web UI, go to `Storage` > `Disks` to view the list of available drives.

> A storage pool in TrueNAS allows you to combine multiple drives into a single managed storage unit, improving both performance and redundancy.

29. Click `Create Pool` to open the pool creation wizard. Only the first two sections are required; sections 3 to 7 are optional.

30. In the General Info section, provide a name for the storage pool.

31. In the Data section, choose a layout based on your number of drives and desired level of redundancy.

|           Layout           |                                                           Definiton                                                            | Minimum disks required |                Usable storage                 |                    Storage sacrificed                    |             Failure Impact             |
| :------------------------: | :----------------------------------------------------------------------------------------------------------------------------: | :--------------------: | :-------------------------------------------: | :------------------------------------------------------: | :------------------------------------: |
|      Stripe (RAID 0)       |        Combines multiple disks into a single large storage unit for maximum speed and capacity but offers no redundancy        |           1            |          100% of total disk capacity          |                           None                           |  If one disk fails, all data is lost   |
|      Mirror (RAID 1)       |          Duplicates data across two disks for redundancy. If one disk fails, the data is still safe on the other disk          |           2            |          50% of total disk capacity           | 50% (one disk's worth of storage if used for redundancy) |  Can survive the failure of one disk   |
| RAIDZ1 (RAID 5 equivalent) |       Provides redundancy by allowing one disk to fail without losing data. Best for a balance of storage and protection       |           3            |  Total capacity of all disks minus one disk   |               One disk's worth of storage                |  Can survive the failure of one disk   |
| RAIDZ2 (RAID 6 equivalent) | Offers higher redundancy, allowing two disks to fail while keeping data safe. Used in setups where data protection is priority |           4            |  Total capacity of all disks minus two disks  |               Two disk's worth of storage                |  Can survive the failure of two disks  |
|           RAIDZ3           | Designed for maximum redundancy, ensuring data remains safe even if three disks fails. Mostly used in enterprise environments  |           5            | Total capacity of all disks minus three disks |              Three disk's worth of storage               | Can survive the failure of three disks |

32. After selecting a layout, click `Save and Go to Review` to review your configuration. If everything looks correct, click `Create Pool` and confirm.
