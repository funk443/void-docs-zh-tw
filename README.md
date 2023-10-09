# Void 手冊

**本翻譯僅供參考，如對翻譯的內容有疑義，請自行參閱原文。我不會為翻譯內容錯誤所造成的損失負責。**

歡迎來到 Void Linux 的手冊。這個 Repo 存放了
<https://docs.voidlinux.org/> 手冊的繁體中文版。如果想為這個 Repo 出一
份心力，請參考 [CONTRIBUTING](./CONTRIBUTING.md) 。

## Building

[res/build.sh](./res/build.sh) 腳本會產生 HTML 、 roff 和 PDF版的 Void
文件以及 `void-docs.1` 手冊頁。執行它需要下列程式：

- `mdBook`
- `findutils`
- `lowdown` （0.8.1 或更高的版本）
- `texlive`
- `perl`
- `perl-File-Which`
- `perl-JSON`
- `librsvg-utils`
- `python3-md2gemini`

要產生並安裝這些檔案，設定好 `PREFIX` 和 `DESTDIR` 這兩個環境變數，並
依序執行 `res/build.sh`及 `res/install.sh` 。
