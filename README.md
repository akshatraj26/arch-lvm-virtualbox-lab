# 🧱 Logical Volume Management (LVM) + Storage Management Lab

### Arch Linux UEFI Installation + Advanced Storage Operations

---

## 📌 Project Overview

This lab demonstrates a **production-style Arch Linux installation using LVM**, including:

* UEFI boot setup
* Physical Volume (PV) creation
* Volume Group (VG) configuration
* Multiple Logical Volumes (LVs)
* Filesystem formatting and mounting
* Xorg + LightDM + XFCE setup
* Advanced LVM resizing
* Snapshot creation & restore
* Swap management
* Basic system hardening

This project highlights **Linux storage engineering skills**, filesystem management, and real-world system administration tasks.

---

# 🏗 Architecture Overview

```
UEFI Firmware
     ↓
EFI Partition (/boot)
     ↓
LVM Physical Volume (/dev/sda2)
     ↓
Volume Group (archvg)
     ├── root   (/)
     ├── home   (/home)
     ├── var    (/var)
     └── swap
```

---

# 💽 Disk Layout

| Device             | Purpose              |
| ------------------ | -------------------- |
| `/dev/sda1`        | EFI System Partition |
| `/dev/sda2`        | LVM Physical Volume  |
| `/dev/archvg/root` | Root filesystem      |
| `/dev/archvg/home` | Home directory       |
| `/dev/archvg/var`  | Variable data        |
| `/dev/archvg/swap` | Swap                 |

---

# 🧰 LVM Setup Workflow

## 1️⃣ Create Physical Volume

```bash
pvcreate /dev/sda2
```

## 2️⃣ Create Volume Group

```bash
vgcreate archvg /dev/sda2
```

## 3️⃣ Create Logical Volumes

```bash
lvcreate -L 15G archvg -n root
lvcreate -L 8G archvg -n home
lvcreate -L 2G archvg -n swap
lvcreate -L 5G archvg -n var
```

---

# 📂 Filesystem Configuration

```bash
mkfs.ext4 /dev/archvg/root
mkfs.ext4 /dev/archvg/home
mkfs.ext4 /dev/archvg/var
mkswap /dev/archvg/swap
mkfs.fat -F32 /dev/sda1
```

Mount structure:

```bash
mount /dev/archvg/root /mnt
mount /dev/archvg/home /mnt/home
mount /dev/archvg/var /mnt/var
mount /dev/sda1 /mnt/boot
swapon /dev/archvg/swap
```

---

# ⚙️ Initramfs Configuration (Critical for LVM)

Edited `/etc/mkinitcpio.conf`:

```
HOOKS=(base udev autodetect modconf block lvm2 filesystems keyboard fsck)
```

Regenerated:

```bash
mkinitcpio -P
```

---

# 🖥 Bootloader Setup (UEFI)

Installed GRUB:

```bash
pacman -S grub efibootmgr
```

Installed to EFI:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

---

# 🖥 Desktop Environment

Installed:

* Xorg
* XFCE
* LightDM display manager

```bash
pacman -S xorg-server xfce4 xfce4-goodies lightdm lightdm-gtk-greeter
systemctl enable lightdm
```

---

# 🔄 Advanced LVM Operations (Core Lab Section)

---

## 🔹 Resize Logical Volumes Safely

### Example: Shrink `/home` and Extend `/`

### 1️⃣ Unmount `/home`

```bash
umount /home
e2fsck -f /dev/archvg/home
resize2fs /dev/archvg/home 6G
lvreduce -L 6G /dev/archvg/home
```

### 2️⃣ Extend Root

```bash
lvextend -L +2G /dev/archvg/root
resize2fs /dev/archvg/root
```

✔ Demonstrates live storage management.

---

# 📸 LVM Snapshots Lab

## Create Snapshot

```bash
lvcreate -L 2G -s -n rootsnap /dev/archvg/root
```

## Restore Snapshot

```bash
umount /
lvconvert --merge /dev/archvg/rootsnap
reboot
```

✔ Useful for rollback testing.

---

# 🔁 Dynamic Swap Management

Disable swap:

```bash
swapoff /dev/archvg/swap
```

Resize:

```bash
lvresize -L 4G /dev/archvg/swap
mkswap /dev/archvg/swap
swapon /dev/archvg/swap
```

---

# 🔐 Basic Hardening

## Disable Root Login (SSH)

Edit:

```
/etc/ssh/sshd_config
```

Set:

```
PermitRootLogin no
PasswordAuthentication no
```

Restart:

```bash
systemctl restart sshd
```

---

## 🛡 Firewall Setup (UFW)

```bash
pacman -S ufw
systemctl enable ufw
ufw default deny incoming
ufw default allow outgoing
ufw enable
```

---

## 🚫 Fail2Ban

```bash
pacman -S fail2ban
systemctl enable fail2ban
```

---

# 📊 LVM Commands Reference

| Command       | Purpose               |
| ------------- | --------------------- |
| `pvs`         | View physical volumes |
| `vgs`         | View volume groups    |
| `lvs`         | View logical volumes  |
| `lvextend`    | Increase LV size      |
| `lvreduce`    | Reduce LV size        |
| `lvcreate -s` | Create snapshot       |

---

# 🧠 Skills Demonstrated

* Linux storage engineering
* UEFI boot management
* LVM architecture design
* Filesystem resizing (ext4)
* Snapshot & rollback
* Swap tuning
* System hardening
* Service management (systemd)
* Desktop environment deployment

---

# 🚀 Why This Project Matters

This lab simulates real-world:

* Server provisioning
* Production storage scaling
* Incident recovery
* System rollback scenarios
* Enterprise Linux administration

It reflects **Systems Engineer / DevOps-level Linux expertise**.

---

# 📎 Future Improvements

* Migrate to LVM thin provisioning
* Add encrypted LUKS layer
* Automate via Bash script
* Convert to Ansible playbook
* Replace ext4 with Btrfs
* Add monitoring (Prometheus + Node Exporter)

---

# 🧾 Author

**Akshat Raj**
Arch Linux • LVM • Systems Engineering Lab
