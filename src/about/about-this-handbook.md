# 關於本指南

本指南並非注重於如何使用和設定一般的 Linux 程式，而是在說明如何安裝、
設定和維護 Void Linux 系統，並點出Void 和其他 Linux Distro 的差異。

要在本指南中搜尋某樣東西，點擊「放大鏡」圖標，或按下 「s」 鍵。

如果你在找的是如何設定一個 Linux 系統，你應該去查找該軟體的文件。另外，
你還可以查閱[Arch Wiki](https://wiki.archlinux.org) ，或其他網路搜尋引
擎。

## 閱讀手冊頁

本指南並沒有大量的複製及貼上的設定指示，但我們有提供到 [man
pages](https://man.voidlinux.org) 的連結。

要學習如何使用 [man(1)](https://man.voidlinux.org/man.1) 手冊頁閱讀器，
使用這個指令 `man man` 。它可以使用 `/etc/man.conf` 來設定。查閱
[man.conf(5)](https://man.voidlinux.org/man.conf.5) 以獲得更多資訊。

Void 使用 [mandoc](https://mandoc.bsd.lv/) 工具組來產生手冊頁。

## 指令範例

在本指南中的範例通常有一些需要在你的終端機中運行的指令。當你看到這些範
例時，以 `$` 開頭的行要以普通使用者身份來執行。而 `#` 開頭的行則應以
`root` 身份來執行。在這些指令之後，可能會有一些該行的輸出範例。

### 佔位符

某些範例中會有佔位符。你應該自行以合適的內容替換掉該佔位符，舉例來說：

```
# ln -s /etc/sv/<service_name> /var/service/
```

這表示你需要用實際的服務名稱來替換掉 `<service_name>` 。
