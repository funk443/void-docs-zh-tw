# 準備安裝媒介

在[下載好安裝映像檔後](../index.md#下載安裝映像)，你需要把它寫入到某種
可以用來開機的媒介，如 USB 隨身碟、SD 卡或 CD/DVD。

## 在 Linux 上創造可開機的 USB 隨身碟或 SD 卡

### 確認設備

在寫入映像前，你需要辨識你要寫入的設備。你可以透過
[fdisk(8)](https://man.voidlinux.org/man8/fdisk.8) 來完成這件事。在連
接上你的儲存設備後，透過以下指令取得裝置位置：

```
# fdisk -l
Disk /dev/sda: 7.5 GiB, 8036286464 bytes, 15695872 sectors
Disk model: Your USB Device's Model
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

在上面這範例中，我們可以看到我們所插入的 USB 隨身碟是 `/dev/sda`。在
Linux 上，一個 USB 裝置的位址通常會長得像 `/dev/sdX`（X 是一個英文字
母），SD 卡會長得像 `/dev/mmcblkX`，其他裝置也會有對應的名字。當你不確
定你的裝置是哪個時，你可以使用裝置型號和其空間大小（以上面的範例來說，
此裝置的大小是 `7.5GiB`）來判斷某個裝置是不是你想要的。

當你辨別出你要用的裝置時，使用
[umount(8)](https://man.voidlinux.org/man8/umount.8) 來缷載它，以確保
它 **沒有** 被掛載：

```
# umount /dev/sdX
umount: /dev/sdX: not mounted.
```

### 寫入 live 映像檔

可以使用 [dd(1)](https://man.voidlinux.org/man1/dd.1) 指令來將一個映像
檔寫入其他裝置：

**警告**：這會把該裝置上所有的檔案摧毀。

```
# dd bs=4M if=/path/to/void-live-ARCH-DATE-VARIANT.iso of=/dev/sdX
90+0 records in
90+0 records out
377487360 bytes (377 MB, 360 MiB) copied, 0.461442 s, 818 MB/s
```

`dd` 指令在直到它完成（或失敗）前不會輸出認何東西。而隨著裝置的不同，
這個過程可以花費數分鐘或以上。如果你使用的是 GNU 核心工具組的 `dd` 的
話，你可以加入 `status=progress` 參數來啟用狀態輸出。

最後，在將裝置退出前，確認所有資料都已經完全寫入：

```
$ sync
```

你使用的裝置和所選的 live 映像檔會影響到寫入的數量、複製的次數和速率

## 燒錄至 CD 或 DVD

任何一個光碟燒錄軟體都應該有辦法將 `.iso` 檔寫入 CD 或 DVD 中。以下自
由軟體可以做到這些事（跨平台的支援可能不盡相同）：

- [Brasero](https://wiki.gnome.org/Apps/Brasero/)
- [K3B](https://userbase.kde.org/K3b)
- [Xfburn](https://docs.xfce.org/apps/xfburn/start)

請注意，如果使用 CD 或 DVD，live 互動的速度會比使用一個 USB 隨身碟或硬
碟還慢。
