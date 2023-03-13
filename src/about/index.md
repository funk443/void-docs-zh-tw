# 關於

歡迎來到 Void 指南！請先閱讀「[關於本指南](./about-this-handbook.md)」
以了解如何有效地使用本手冊。本指南的本地存檔可以透過[安
裝](../xbps/index.md) `void-docs` 套件後以
[void-docs(1)](https://man.voidlinux.org/void-docs.1) 來進行存取。

Void 是一個獨立的，[滾動式發
行](https://en.wikipedia.org/wiki/Rolling_release) 的 Linux 發行版。它
並不是某個發行版的分支，而是從頭開發的，並且更注重穩定性，而非[最新技
術](https://en.wikipedia.org/wiki/Bleeding_edge_technology)。此外，還
有一些特點讓 Void 十分獨特：

- [XBPS](https://github.com/void-linux/xbps) 套件管理員，它是一種速度
  飛快，由 Void 內部開發，並在更新套件時檢查更新不會破壞其他套件的相依
  性。
- [musl libc](https://musl.libc.org/)，注重於遵守標準及正確性，擁有一
  流的支援。這使我們能在 musl 系統上建構在 glibc 上無法構建的靜態組件。
- [runit](../config/services/index.md) 用於
  [init(8)](https://man.voidlinux.org/init.8) 及服務監控。runit 讓我們
  有辦法使用 musl 作為 libc，而這在
  [systemd](https://www.freedesktop.org/wiki/Software/systemd/) 上是不
  可能的。使用 runit 還使 Void 的核心系統更簡潔、高效。

Void 的穩定性通常足以應付日常使用。 Void 是由一小群開發者以閒暇時間開發的，我們以此為樂，並希望我們的努力可以幫助到其他人。

「Void」 這個名字是從 C 語言中的 `void` 關鍵字而來，沒有什麼特別的含意。
