# Live Installers

Void 提供內含基本工具、安裝程式和套件檔案的 live 安裝映像檔，可用來安
裝新的 Void 系統，或修復無法正常啟動的系統。

我們提供 `x86_86` 架構的 `glibc` 和 `musl` 的安裝映像。也有 `i686` 架
構的，不過只有 `glibc` 版本。其他架構都沒有 live 安裝器，這些架構的使
用者需要使用 rootfs 壓縮檔安裝，或手動安裝。

## 安裝器映像

Void 提供兩種映像：基本映像和 xfce 映像。我們推薦 Linux 初學者使用功能
更完善的 xfce 映像，不過進階使用者可能會想要使用基本映像來安裝系統和他
們想要的套件。

### 基本映像

基本映像只提供了可用來安裝一個 Void 系統的最基礎套件。這些基礎套件只夠
用來設定新的機器、升級系統或從 repo 安裝其他套件。

### Xfce 映像

Xfce 映像提供了完整的桌面環境、網頁瀏覽器和其他基本程式。它跟基本映像
的唯一區別只有額外安裝的套件和服務。

映像中包含以下軟體：

- **視窗管理員:** xfwm4
- **檔案管理員:** Thunar
- **網路瀏覽器:** Firefox ESR
- **終端機:** xfce4-terminal
- **純文字編輯器:** Mousepad
- **圖片檢視器:** Ristretto
- **其他:** Bulk rename, Orage Globaltime, Orage Calendar, Task
    Manager, Parole Media Player, Audio Mixer, MIME type editor,
    Application finder

Xfce 映像上的安裝流程和基本映像上一樣，除了你 **必須** 在安裝時選擇
`Local` 來源。如果你選了 `Network`，安裝器會下載並安裝最新版的基本系統
套件，而不會下載映像檔中其他額外的套件。
