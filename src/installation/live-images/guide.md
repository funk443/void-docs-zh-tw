# 安裝指南

當你[下載好](../index.md#downloading-installation-media)一個 Void 映像
檔並[準備好](./prep.md)安裝媒介後，你就可以開始安裝 Void Linux 了。

在你開始安裝前，你應該先確認你要安裝的機器是使用 BIOS 還是 UEFI 方式開
機。這會影響到你如何做磁碟分區，詳情請參考[分區說明](./partitions.md)。

安裝腳本不支援下列功能：

- [LVM](https://en.wikipedia.org/wiki/Logical_volume_management)
- [LUKS](https://en.wikipedia.org/wiki/Linux_Unified_Key_Setup)
- [ZFS](https://en.wikipedia.org/wiki/ZFS)

## 開機

使用你的安裝媒介開機。如果你的 RAM 夠大，在開機選單時，會有一個選項讓
你可以將整個映像檔複製進你的記憶體裡。這會需要點時間，不過會讓後面的安
裝過程變更快。

當以映像檔開機後，以 `root` 登入，密碼是 `voidlinux`，然後執行下列指令：

```
# void-installer
```

以下內容會詳細解釋安裝器中的每個過程。

## 鍵盤（Keyboard）

選擇你想要的鍵盤配置，標準的「qwerty」鍵盤通常使用「us」配置。

## 網路（Network）

選擇你的主要網路介面。如果你選擇不使用 DHCP，你會被要求輸入 IP 地址、
閘和 DNS 伺服器。

如果你選擇了一個無線網路介面，你需要輸入網路的 SSID、加密方法（`wpa`
或 `wep`）還有密碼。如果 `void-installer` 沒辦法連接到網路，你需要先退
出安裝器，並透過
[wpa_supplicant](../../config/network/wpa_supplicant.md) 和
[dhcpcd](../../config/network/index.md#dhcpcd) 手動設定好網路再繼續。

## 來源（Source）

如果要從映像檔中直接安裝套件，請選擇 `Local`。你還可以選擇 `Network`
來從 Void 的套件庫下載最新的套件。

> **警告：**如果你想要安裝 xfce 映像的桌面環境，你**必須**選擇 `Local`。

## 主機名稱（Hostname）

為你的電腦選個主機名稱（需為全部小寫，並無空格）。

## 語系（Locale）

選擇你想要的預設語系。這個選項只有在 glibc 下有用，因為 musl 目前並不
支援語系。

## 時區（Timezone）

選擇你的時區。

## Root 密碼（Root password）

輸入並確認你新系統的 `root` 密碼。這個密碼不會被顯示在螢幕上。

## 使用者帳戶（User account）

選擇一個登入帳戶（預設是 `void`），並為它取個好名字。接著為這個帳戶設
定一個密碼。然後你會需要為這個新使用者選擇它所在之群組。它預設會在
`wheel` 裡，所以可以使用 `sudo` 。有關預設的群組請參閱[這
裡](../../config/users-and-groups.html#default-groups)。

登入帳戶的名稱會有些限制，請參閱
[useradd(8)](https://man.voidlinux.org/useradd.8#CAVEATS)。

## 引導程式（Bootloader）

選擇一個磁碟來安裝引導程式，它會在安裝 Void 時被安裝。你也可以選擇
`none` 來跳過這步，然後在安裝完 Void 後自行安裝引導程式。如果你選擇安
裝，安裝器會詢問你是否要為 GRUB 選單安裝一個圖形終端。

## 磁碟分區（Partition）

接下來，你需要為你的磁碟分區。Void 並沒有提供預設的分區方案，所以你會
需要自行使用 [cfdisk(8)](https://man.voidlinux.org/cfdisk.8) 來分區。
你需要選擇在哪個磁碟上進行分區，然後安裝器會啟動 `cfdisk`。別忘了在你
離開分區編輯器前寫入分割表。

如果你使用 UEFI，我們建議你使用 GPT 作為分割表，並建立一個 `EFI
System` 類型的分區（通常大小在 200MB-1GB），這個分區會被掛載在
`/boot/efi`。

如果你使用 BIOS，我們建議你使用 MBR 作為分割表。進階的使用者也可以使用
GPT，不過你需要[為 GRUB 建立一個特殊的 BIOS 分
區](./partitions.md#bios-system-notes)才能開機。

更多詳情請參閱[分區說明](./partitions.md)。

## 檔案系統（Filesystems）

為你建立的每個分區創立檔案系統。你需要為每個分區選擇一種檔案系統、是否
創建一個新的檔案系統及一個掛載點。完成後選擇 `Done` 回到主選單。

如果你使用 UEFI，你需要建立一個 `vfat` 檔案系統，並將它掛載在
`/boot/efi`。

## 檢查設定（Review settings）

你最好在繼續前再審視一次你的設定。使用右方向鍵選擇設定按鈕並按下
`<enter>`。你剛剛的所有選擇都會顯示在該頁面。

## 安裝（Install）

選擇 `Install` 會開始安裝器。它會建立你所選的所有檔案系統、安裝基本系
統套件、產生 initramfs 並將 GRUB2 安裝到開機分區上。

以上流程應該都會自動執行。它們結束後你就可以重新開機進到你的新 Void
Linux 系統了！

## 在安裝之後

在進到你新安裝的 Void 系統後，你應該[更新系
統](../../xbps/index.md#updating)。
