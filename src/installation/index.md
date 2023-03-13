# 安裝

這個段落說明了安裝 Void 的步驟。如果有特定的需求，請參閱「[進階安裝說
明](./guides/index.md)」。

## 系統需求

Void 對硬體資源的要求不高，不過我們還是建議應該滿足下列條件：

| Architecture | CPU              | RAM  | Storage |
|--------------|------------------|------|---------|
| x86_64-glibc | x86_64           | 96MB | 700MB   |
| x86_64-musl  | x86_64           | 96MB | 600MB   |
| i686-glibc   | Pentium 4 (SSE2) | 96MB | 700MB   |

請注意，安裝 xfce 映像需要更多硬體資源。

Void 不支援 i386、i486 和 i586 架構。

在安裝 musl Void 之前，請先閱讀本指南的 [musl 章節](./musl.md)，以便你
了解一些軟體上的相容性問題。

我們強烈建議你在安裝過程中連上網路以下載更新，不過這不是必須的。ISO 檔
中已包含安裝所需的所有資料，無需網路連接即可安裝。

## 下載安裝映像

最新的 live 映像及 rootfs 壓縮檔可以從
<https://repo-default.voidlinux.org/live/current/> 下載。你也可以從[其
他鏡像站](../xbps/repositories/mirrors/index.md)下載。之前的映像檔可以
在 <https://repo-default.voidlinux.org/live/> 找到。

## 驗證映像檔

每個映像檔的發行目錄會包含兩個檔案用來驗證你下載的映像檔。首先，
`sha256sum.txt` 含有映像檔的核對和，它可以被用來驗證映像檔的完整性。再
來是 `sha256sum.sig`，用來確認核對和的真實性。

你有必要要驗證映像檔的完整性及真實性。所以，我們建議你應該下載這兩個檔
案。

### 驗證映像完整性

你可以使用 [sha256sum(1)](https://man.voidlinux.org/sha256sum.1) 及上
一小節中下載的 `sha256sum.txt` 來檢驗檔案的完整性。以下的指令會檢查你
所下載之映像檔的完整性：

```
$ sha256sum -c --ignore-missing sha256sum.txt
void-live-x86_64-musl-20170220.iso: OK
```

以上輸出說明映像檔沒有損壞。

### 驗證數位簽名

在你使用任何映像檔前，我們強烈建議你應該檢查映像檔上的簽名，以確保該檔
案沒有被動過手腳。

目前發佈的映像都有一組特定的 signify 金鑰。如果你已經在使用 Void 了，
你可以通過安裝 `void-release-keys` 來取得這些金鑰。同時你還需要
[signify(1)](https://man.voidlinux.org/signify.1) 或
[minisign(1)](https://man.voidlinux.org/minisign.1)。在 Void 上，它們
分別由 `outils` 和 `minisign` 套件提供。

要在不是 Void Linux 的系統上取得 `signify` ：

- 在 Arch Linux 或基於 Arch 的系統上，安裝 `signify` 套件
- 在 Debian 或基於 Debian 的系統上，安裝 `siginify-openbsd` 套件
- 在其他發行版上，安裝[此列
  表](https://repology.org/project/signify-openbsd/versions)中所列的套
  件
- 在 macOS 上，使用 homebrew 安裝 `signify-osx`

而 `minisign` 執行檔通常由同名的套件提供，即使沒有 WSL 或 MinGW 也可以
在 Windows 上安裝。

如果你現在不是在使用 Void Linux，你還需要從我們的 [Git
repo](https://github.com/void-linux/void-packages/tree/master/srcpkgs/void-release-keys/files/)
中取得合適的金鑰。

當你取得金鑰後，你就可以使用 `sha256sum.sig` 和 `sha256sum.txt` 來驗證
你的映像檔了。首先，你需要驗證 `sha256sum.txt` 的真實性。

以下示範了驗證 20210930 映像檔的 `sha256sum.txt`。首先，使用 `signify`：

```
$ signify -V -p /etc/signify/void-release-20210930.pub -x sha256sum.sig -m sha256sum.txt
Signature Verified
```

再來使用 `minisign`：

```
$ minisign -V -p /etc/signify/void-release-20210930.pub -x sha256sum.sig -m sha256sum.txt
Signature and comment signature verified
Trusted comment: timestamp:1634597366	file:sha256sum.txt
```

最後，你需要驗證你所下載的映像檔的核對和跟 `sha256sum.txt` 裡的相符。
你可以使用 [sha256(1)](https://man.voidlinux.org/md5.1) 工具，此工具也
來自 `outils` 套件。以下示範檢驗 20210930 `x86_64` 映像檔：

```
$ sha256 -C sha256sum.txt void-live-x86_64-20210930.iso
(SHA256) void-live-x86_64-20210930.iso: OK
```

如果你沒有 `sha256` 工具的話，你可以使用如
[sha256sum(1)](https://man.voidlinux.org/sha256sum.1) 等工具自行計算檔
案的 SHA256 哈希值，然後再跟 `sha256sum.txt` 中對應的值進行比較：

```
$ sha256sum void-live-x86_64-20210930.iso
45b75651eb369484e1e63ba803a34e9fe8a13b24695d0bffaf4dfaac44783294  void-live-x86_64-20210930.iso
$ grep void-live-x86_64-20210930.iso sha256sum.txt
SHA256 (void-live-x86_64-20210930.iso) = 45b75651eb369484e1e63ba803a34e9fe8a13b24695d0bffaf4dfaac44783294
```

如果驗證的結果不是 「OK」 的狀態，請不要使用該映像檔！請通知 Void
Linux 團隊，並說明你從何處取得該映像檔，及你如何驗證它，我們會處理此事。
