# musl

[musl](https://musl.libc.org/) 是一個輕量、高效和簡潔的C函式庫。

Void 對 musl 提供官方支援，所有平台都有 musl 版本可供使用（不過 i686
平台上不提供 bianry 套件）。此外，官方 repo 中除了 glibc 版本的套件，
還提供與 musl 相容的套件。

目前，我們提供 musl 的 nonfree 跟 debug 的 repo，不過還沒有 multilib。

## 不相容的軟體

musl 只有對標準的最小相容性。許多常用平台的擴展都沒有被實現。因此，許
多軟體需要一些修改才能正確地編譯或（和）執行。Void 的開發者會為這些軟
體打上補丁，並希望這些增強可移植性或正確性的改變會被軟體的上游所接受。

專有軟體通常只支援 glibc 系統，不過有時候，這些軟體通過
[flatpak](../config/external-applications.md#flatpak) 在 musl 上執行。
[專有的 NVIDIA 驅動程
式
](../config/graphical-session/graphics-drivers/nvidia.md#nvidia-proprietary-driver)
並不支援 musl 系統，在評估硬體相容性的應該考量這點。

### glibc chroot

需要 glibc 的軟體可以透過 glibc
[chroot](../config/containers-and-vms/chroot.md) 執行。
