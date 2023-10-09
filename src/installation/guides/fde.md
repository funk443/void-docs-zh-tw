# 全磁碟加密

**警告**：你的儲存裝置資訊可能會有所不同，所以請自行確認你有沒有用對裝置。

## 分區

進入 live 映像並登入。

使用 [cfdisk](https://man.voidlinux.org/cfdisk) 在磁碟上建立一個物理分區，並將它
標示為可用於開機的（bootable）。對一個 MBR 系統來說，分區應該要長得類似下面這樣。

```
# fdisk -l /dev/sda
Disk /dev/sda: 48 GiB, 51539607552 bytes, 100663296 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x4d532059

Device     Boot Start       End   Sectors Size Id Type
/dev/sda1  *     2048 100663295 100661248  48G 83 Linux
```

UEFI 系統需要該磁碟有 GPT 標籤和一個 EFI 系統分區。EFI 系統分區的大小取決於你的
需求，不過多數時候來說，100M 就夠用了。對一個 UEFI 系統來說，分區會長得類似於下
面這樣。

```
# fdisk -l /dev/sda
Disk /dev/sda: 48 GiB, 51539607552 bytes, 100663296 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: EE4F2A1A-8E7F-48CA-B3D0-BD7A01F6D8A0

Device      Start       End   Sectors  Size Type
/dev/sda1    2048    264191    262144  128M EFI System
/dev/sda2  264192 100663262 100399071 47.9G Linux filesystem
```

## 加密磁區的設置


[Cryptsetup](https://man.voidlinux.org/cryptsetup.8) 預設使用 LUKS2，不過 GRUB
2.06 版之前只有支援 LUKS1。

GRUH 只部份支援 LUKS2，更確切地來說，只
[支援](https://git.savannah.gnu.org/cgit/grub.git/commit/?id=365e0cc3e7e44151c14dd29514c2f870b49f9755)
PBKDF2 金鑰衍生函數，而 LUKS2 預設使用的是 Argon2i，而非 PBKDF2。
([GRUB Bug 59409](https://savannah.gnu.org/bugs/?59409))
在此情況下，使用 Argon2i（或其他 KDF）所加密的分區會 *無法* 被解密。因此，本指南
只推薦使用 LUKS1。

如果你使用 UEFI 系統，在下面這裡，你需要輸入的是 `/dev/sda2` 而非 `/dev/sda1`。

```
# cryptsetup luksFormat --type luks1 /dev/sda1

WARNING!
========
This will overwrite data on /dev/sda1 irrevocably.

Are you sure? (Type uppercase yes): YES
Enter passphrase:
Verify passphrase:
```

建立好後，我們需要開啟它。請自行使用合適的名字替換 `voidvm`。再次提醒，如果你在
UEFI 系統上，你在這裡需要輸入的是 `/dev/sda2`。

```
# cryptsetup luksOpen /dev/sda1 voidvm
Enter passphrase for /dev/sda1:
```

開啟完 LUKS 容器後，使用該分區建立一個 LVM 磁區組。

```
# vgcreate voidvm /dev/mapper/voidvm
  Volume group "voidvm" successfully created
```

現在應該就有個名為 `voidvm`（或你剛取的任何名字）的空磁區組。

接下來，我們需要為這個磁區組建立一些邏輯分區。此處以 `/` 10G、`swap` 2G 並將剩餘
的分給 `/home` 為例，您可以自身需求進行調整。

```
# lvcreate --name root -L 10G voidvm
  Logical volume "root" created.
# lvcreate --name swap -L 2G voidvm
  Logical volume "swap" created.
# lvcreate --name home -l 100%FREE voidvm
  Logical volume "home" created.
```

接著這些分區建立檔案系統，此處以原文作者喜好使用 XFS 為例，您可依喜好自行選用任
何
[GRUB 所支援的檔案系統](https://www.gnu.org/software/grub/manual/grub/grub.html#Features)
使用。

```
# mkfs.xfs -L root /dev/voidvm/root
meta-data=/dev/voidvm/root       isize=512    agcount=4, agsize=655360 blks
...
# mkfs.xfs -L home /dev/voidvm/home
meta-data=/dev/voidvm/home       isize=512    agcount=4, agsize=2359040 blks
...
# mkswap /dev/voidvm/swap
Setting up swapspace version 1, size = 2 GiB (2147479552 bytes)
```

## 安裝系統

接著，我們設定好 chroot，並安裝好基本的系統。

```
# mount /dev/voidvm/root /mnt
# mkdir -p /mnt/home
# mount /dev/voidvm/home /mnt/home
```

在 UEFI 系統上，EFI 系統分區也需要掛載。

```
# mkfs.vfat /dev/sda1
# mkdir -p /mnt/boot/efi
# mount /dev/sda1 /mnt/boot/efi
```

將 RSA 金鑰從安裝媒介複製到安裝目標的根目錄：

```
# mkdir -p /mnt/var/db/xbps/keys
# cp /var/db/xbps/keys/* /mnt/var/db/xbps/keys/
```

在我們 chroot 前，別忘了要安裝系統。確保你有使用
[合適的鏡像下載點](../../xbps/repositories/index.md#the-main-repository)

```
# xbps-install -Sy -R https://repo-default.voidlinux.org/current -r /mnt base-system lvm2 cryptsetup grub
[*] Updating `https://repo-default.voidlinux.org/current/x86_64-repodata' ...
x86_64-repodata: 1661KB [avg rate: 2257KB/s]
130 packages will be downloaded:
...
```

在 UEFI 系統上要安裝的包會有點不同，具體安裝指令如下。

```
# xbps-install -Sy -R https://repo-default.voidlinux.org/current -r /mnt base-system cryptsetup grub-x86_64-efi lvm2
```

待安裝完好，我們可以使用 `xtools` 中的
[`xchroot(1)`](https://man.voidlinux.org/xchroot.1)
chroot 進系統，並將剩餘所需的設定做好。你也可以
[手動 chroot](../../config/containers-and-vms/chroot.md#manual-method)。

```
# xchroot /mnt
[xchroot /mnt] # chown root:root /
[xchroot /mnt] # chmod 755 /
[xchroot /mnt] # passwd root
[xchroot /mnt] # echo voidvm > /etc/hostname
```

只有使用 glibc 的系統需要下面這步：

```
[xchroot /mnt] # echo "LANG=en_US.UTF-8" > /etc/locale.conf
[xchroot /mnt] # echo "en_US.UTF-8 UTF-8" >> /etc/default/libc-locales
[xchroot /mnt] # xbps-reconfigure -f glibc-locales
```

### 設定檔案系統

下一步是編輯 `/etc/fstab`，詳細步驟會取決於你怎麼設定和命名你的檔案系統。以本教
學來說，此檔案會長得像下面這樣：

```
# <file system>   <dir> <type>  <options>             <dump>  <pass>
tmpfs             /tmp  tmpfs   defaults,nosuid,nodev 0       0
/dev/voidvm/root  /     xfs     defaults              0       0
/dev/voidvm/home  /home xfs     defaults              0       0
/dev/voidvm/swap  swap  swap    defaults              0       0
```

在 UEFI 上，EFI 系統分區也會有一行資料。

```
/dev/sda1	/boot/efi	vfat	defaults	0	0
```

### 設定 GRUB

然後我們需要讓 GRUB 能解鎖我們的檔案系統。在 `/etc/default/grub` 中加入以下的內
容：

```
GRUB_ENABLE_CRYPTODISK=y
```

我們還需要設定核心，讓它能找到加密裝置。首先要找到硬碟的 UUID。

```
[xchroot /mnt] # blkid -o value -s UUID /dev/sda1
135f3c06-26a0-437f-a05e-287b036440a4
```

找到後在 `/etc/default/grub` 中 `GRUB_CMDLINE_LINUX_DEFAULT=` 這行加上
`rd.lvm.vg=voidvm rd.luks.uuid=<UUID>`，並確定這個 UUID 跟你 `/dev/sda1` 裝置的
UUID 一樣（如上面的例子，使用 [blkid(8)](https://man.voidlinux.org/blkid.8) 來尋
找）。

## LUKS 金鑰設定

為了省去在開機時輸入兩次密碼的麻煩，我們可以設定一個金鑰讓加密的裝置在開機時自動
解鎖。先產生一個隨機金鑰。

```
[xchroot /mnt] # dd bs=1 count=64 if=/dev/urandom of=/boot/volume.key
64+0 records in
64+0 records out
64 bytes copied, 0.000662757 s, 96.6 kB/s
```

並將該金鑰加到加密分區中。

```
[xchroot /mnt] # cryptsetup luksAddKey /dev/sda1 /boot/volume.key
Enter any existing passphrase:
```

變更檔案權限來保護金鑰。

```
[xchroot /mnt] # chmod 000 /boot/volume.key
[xchroot /mnt] # chmod -R g-rwx,o-rwx /boot
```

我們需要將這個金鑰檔案加到 `/etc/crypttab` 中。以下例子在 UEFI 系統中會是
`/dev/sda2`。

```
voidvm   /dev/sda1   /boot/volume.key   luks
```

同時金鑰檔和 `crypttab` 需要被加到 initramfs 中。創建
`/etc/dracut.conf.d/10-crypt.conf` 並輸入以下內容：

```
install_items+=" /boot/volume.key /etc/crypttab "
```

## 完成系統安裝

安裝導引程式。

```
[xchroot /mnt] # grub-install /dev/sda
```

確定 initramfs 有正常產生：

```
[xchroot /mnt] # xbps-reconfigure -fa
```

離開 chroot，卸載檔案系統，然後重開機。

```
[xchroot /mnt] # exit
# umount -R /mnt
# reboot
```
