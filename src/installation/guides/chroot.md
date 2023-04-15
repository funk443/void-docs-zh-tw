# 通過 chroot 安裝（x86/x86_64/aarch64）

本指南詳細說明了在 x86、x86\_64 和 aarch64 架構上透過 chroot 安裝 Void
的過程。使用本指南不需要使用 chroot 安裝 Linux 的經驗，不過您必須對
Linux 有一定的認識。照著本說明可以創建一個「典型」的系統，使用單一個
SATA/IDE/USB 裝置上的單一個分區。每個步驟都可以按需調整，比如[開啟硬碟
加密](./fde.md)。

Void 提供兩種安裝方法：**XBPS 方法**使用一個在宿主系統上的 [XBPS 套件
管理員](../../xbps/index.md)來安裝基本系統。**ROOTFS 方法**透過解壓縮
ROOTFS 壓縮檔來安裝基本系統。

**XBPS 方法**需要一個安裝有 XBPS 的宿主系統。這可以是一個已經安裝好的
Void、官方的 [live 鏡像](../live-images/prep.md)或任何裝有[靜態連結
XBPS](../../xbps/troubleshooting/static.md)的 Linux 系統。

**ROOTFS 方法**只要求宿主系統可以進入 Linux chrrot，並安裝有
[tar(1)](https://man.voidlinux.org/tar.1) 和
[xz(1)](https://man.voidlinux.org/xz.1)。如果你想從別的 Linux 發行版安
裝 Void，我們推薦這個方法。

## 準備檔案系統

[將你的硬碟分區](../live-images/partitions.md)，並使用
[mke2fs(8)](https://man.voidlinux.org/mke2fs.8)、
[mkfs.xfs(8)](https://man.voidlinux.org/mkfs.xfs.8)、
[mkfs.btrfs(8)](https://man.voidlinux.org/mkfs.btrfs.8)或任何其他檔案
系統的必要工具將他們格式化。

也可以使用 [mkfs.vfat(8)](https://man.voidlinux.org/mkfs.vfat.8) 來創
建 FAT32 分區。不過因為 FAT 檔案系統的許多缺陷，你只應該在沒有其他適合
檔案系統的時候才使用它（像是 EFI 系統分區）。

在 live 映像上，可以使用
[cfdisk(8)](https://man.voidlinux.org/cfdisk.8) 和
[fdisk(8)](https://man.voidlinux.org/fdisk.8) 來進行分區。不過你可能會
比較想用[gdisk(8)](https://man.voidlinux.org/gdisk.8)（在套件
`gptfdisk` 中）或[parted(8)](https://man.voidlinux.org/parted.8)。

以 UEFI 系統來說，別忘了創建一個 EFI 系統分區（ESP）。此分區應有「EFI
System」（代碼 `EF00`）的類別，並使用
[mkfs.vfat(8)](https://man.voidlinux.org/mkfs.vfat.8) 格式化為 FAT32。

如果不確定要創建哪些分區，創建一個 1GB 大小，類型「EFI System」（代號
`EF00`）的分區。接著用剩餘的硬碟空間創建第二個類型為「Linux Filesystem」
（代號 `8300`）的分區。

將這兩個分區分別格式化為 FAT32 和 ext4：

```
# mkfs.vfat /dev/sda1
# mkfs.ext4 /dev/sda2
```

### 創建新 Root 並掛載檔案系統

這篇指南假設你將新的 root 檔案系統掛載在 `/mnt`。你可能會想將它掛載到
其他地方。

如果你使用 UEFI，將 EFI 系統分區掛載到 `/mnt/boot/efi`。

舉例來說，如果要將 `/dev/sda2` 掛載為 `/`，且 `/dev/sda1` 是 EFI 系統
分區：

```
# mount /dev/sda2 /mnt/
# mkdir -p /mnt/boot/efi/
# mount /dev/sda1 /mnt/boot/efi/
```

如果想要使用 swap，使用
[mkswap(8)](https://man.voidlinux.org/mkswap.8) 將該分區初始化。

## 基本系統安裝

請選擇以下兩小節的其中一種方法進行安裝。

如果您要在 aarch64 架構上進行安裝，您除了需要安裝 `base-system` 外，還
需要一個額外的核心套件。例如說 `linux` 是 Void 中提供最新穩定版本的核
心套件。

### 使用 XBPS

選擇一個[鏡像站](../../xbps/repositories/mirrors/index.md)，並根據你想
安裝的系統類型**選擇**合適的[網
址](../../xbps/repositories/index.md#the-main-repository)。方便起見，
將這個網址存到 shell 變數裡。以一個 glibc 安裝來說：

```
# REPO=https://repo-default.voidlinux.org/current
```

XBPS 還需要知道要安裝哪種架構的系統。有這些可用的選項 `x86_64`、
`x86_64-musl`、`i686` 和 `aarch64`。同樣將您的選擇存到一個變數裡：

```
# ARCH=x86_64
```

您所選的架構可以不用跟您目前的作業系統架構一致，不過必須要相容。如果宿
主機在使用 x86_64 的作業系統，則上述三種架構都可用（不論宿主是 musl 還
是 glibc）。但是如果宿主機使用 i686 系統，則它只能用來安裝 i686 的版本。

將 RSA 金鑰從安裝媒介中複製到目標的根目錄中：

```
# mkdir -p /mnt/var/db/xbps/keys
# cp /var/db/xbps/keys/* /mnt/var/db/xbps/keys/
```

使用 [xbps-install(1)](https://man.voidlinux.org/xbps-install.1) 安裝
`base-system` 套件組合：

```
# XBPS_ARCH=$ARCH xbps-install -S -r /mnt -R "$REPO" base-system
```

### 使用 ROOTFS

[下載符合你要架構的 ROOTFS 壓縮
檔
](https://voidlinux.org/download/#download-installable-base-live-images-and-rootfs-tarballs)

將該壓縮檔解壓縮到新創建好的檔案系統中：

```
# tar xvf void-<...>-ROOTFS.tar.xz -C /mnt
```

## 設定系統

除了「安裝基礎系統（限 ROOTFS 方法）」這個章節外，其他部份可適用於
XBPS 方法和 ROOTFS 方法。

### 進入 Chroot

[xchroot(1)](https://man.voidlinux.org/xchroot.1)（由 `xtools` 套件提
供）可被用來設置和進入 chroot。除此之外，你也可以自行[手動配置並進
入](../../config/containers-and-vms/chroot.md#manual-method) chroot。

```
# xchroot /mnt /bin/bash
```

### 安裝基礎方法（限 ROOTFS 方法）

因為 ROOTFS 映像檔是其被創建時的快照，因此其中包含的軟體通常都是過時的，
並且，它並不包含完整的 `base-system`。更新套件管理員並安裝
`base-system`：

```
[xchroot /mnt] # xbps-install -Su xbps
[xchroot /mnt] # xbps-install -u
[xchroot /mnt] # xbps-install base-system
[xchroot /mnt] # xbps-remove base-voidstrap
```

### 安裝設置

在 `/etc/hostname` 中設置主機名稱。並檢查看看
[`/etc/rc.conf`](../../config/rc-files.md#rcconf) 中有沒有什麼需要設置
的選項。如果您安裝的是 glibc 版本，編輯 `/etc/default/libc-locales`，
並取消想要使用的 [locales](../../config/locales.md) 的註釋。

[nvi(1)](https://man.voidlinux.org/nvi.1) 可以在 chroot 中使用，不過你
也可以在此時安裝你喜歡的文字編輯器。

如果你安裝 glibc 版本，使用以下指令產生語系檔案：

```
[xchroot /mnt] # xbps-reconfigure -f glibc-locales
```

### 設置 Root 密碼

[設置好至少一個超級使用者帳號](../../config/users-and-groups.md)。其他
使用者帳號可以之後再創建，不過你必須設定 root 的密碼或創建一個可以使用
[sudo(8)](https://man.voidlinux.org/sudo.8) 的使用者帳號。

用以下指令來設定 root 的密碼：

```
[xchroot /mnt] # passwd
```

### 設置 fstab

可以透過複製 `/proc/mounts` 檔案來從目前掛載的檔案系統自動產生
[fstab(5)](https://man.voidlinux.org/fstab.5) 檔案。

```
[xchroot /mnt] # cp /proc/mounts /etc/fstab
```

將 `/etc/fstab` 中含有 `proc`、`sys`、`devtmpfs` 和 `pts` 的行刪除。

將 `/dev/sdXX`、`/dev/nvmeXnYpZ` 或其他之類的玩意用它們的 UUID 替換掉。
UUID 可以使用 [blkid(8)](https://man.voidlinux.org/blkid.8) 取得。使用
UUID 來代表檔案系統可以確保系統一定找得到該檔案系統，即使設備名稱有改
變過。例如使用 USB 開機，這時使用 UUID 來代表檔案系統就是必要的。不過
其他時候，硬碟的名稱通常不會有變化，除非物理地增加或移除硬碟。所以這個
步驟並非必要，不過我們仍強烈建議使用 UUID 來代表檔案系統。

將 `/` 那行最後面的 `0` 改成 `1`，其他行最後面的 `0` 改成 `2`。這些數
值會影響 [fsck(8)](https://man.voidlinux.org/fsck.8) 的行為。

舉例來說，之前的分區表會在 `fstab` 中產生如下配置：

```
/dev/sda1       /boot/efi   vfat    rw,relatime,[...]       0 0
/dev/sda2       /           ext4    rw,relatime             0 0
```

使用 `blkid` 提供的 UUID 及其他上述步驟修改 `/etc/fstab` 的結果：

```
UUID=6914[...]  /boot/efi   vfat    rw,relatime,[...]       0 2
UUID=dc1b[...]  /           ext4    rw,relatime             0 1
```

注意：`/proc/mounts` 的結果會在每個欄位中間有個空格，在此我們將每欄對
齊以便閱讀。

加入 `/tmp`：

```
tmpfs           /tmp        tmpfs   defaults,nosuid,nodev   0 0
```

如果你有使用 swap，為它加入一行：

```
UUID=1cb4[...]  swap        swap    rw,noatime,discard      0 0
```

## 安裝 GRUB

使用
[grub-install](https://www.gnu.org/software/grub/manual/grub/html_node/Installing-GRUB-using-grub_002dinstall.html)
來將 GRUB 安裝到你的開機碟上。

**在 BIOS 電腦上**，安裝 `grub` 套件，並執行 `grub-install /dev/sdX`，
`/dev/sdX` 代表你想要安裝 GRUB 的硬碟（不是分區）。例如：

```
[xchroot /mnt] # xbps-install grub
[xchroot /mnt] # grub-install /dev/sda
```

**在 UEFI 電腦上**，根據你的電腦架構，安裝 `grub-x86_64-efi`、
`grub-i386-efi` 或 `grub-arm64-efi`。接著執行 `grub-install`，並可以指
定一個引導程式標籤（這個標籤可能會在你選擇開機裝置時出現）。

```
[xchroot /mnt] # xbps-install grub-x86_64-efi
[xchroot /mnt] # grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id="Void"
```

### 安裝 GRUB 時常見問題

如果 EFI 變數無法使用，在使用 `grub-install` 指令的時候加上
`--no-nvram` 參數。

#### 安裝到可移除的媒介或不相容的 UEFI 系統

並非所有系統都有標準、完整的 UEFI 實現。某些時候，我們需要「騙」韌體以
預設備用的地址啟動引導程式。如果是在這樣的情況下，或你想安裝到一個可移
除的硬碟中（像是一個 USB 隨身碟），在執行 `grub-install` 的時候加上
`--removable` 參數。

你也可以使用 [mkdir(1)](https://man.voidlinux.org/mkdir.1) 來建立
`/boot/efi/EFI/boot` 目錄，並將已經安裝好的 GRUB 可執行檔（通常會在
`/boot/efi/EFI/Void/grubx64.efi`，你可以使用
[efibootmgr(8)](https://man.voidlinux.org/efibootmgr.8) 來找它的位置）
複製進去：

```
[xchroot /mnt] # mkdir -p /boot/efi/EFI/boot
[xchroot /mnt] # cp /boot/efi/EFI/Void/grubx64.efi /boot/efi/EFI/boot/bootx64.efi
```

## 最後的最後

使用
[xbps-reconfigure(1)](https://man.voidlinux.org/xbps-reconfigure.1) 來
確保所有安裝的套件都已被正確的設置：

```
[xchroot /mnt] # xbps-reconfigure -fa
```

這會使 [dracut(8)](https://man.voidlinux.org/dracut.8) 產生 initramfs，
並讓 GRUB 產生一個正確的設置檔。

現在，你已經完成安裝了，離開 chroot，並重新開機：

```
[xchroot /mnt] # exit
# umount -R /mnt
# shutdown -r now
```

在重新開機進到你的 Void 安裝後，[更新你的系
統](../../xbps/index.md#updating)。
