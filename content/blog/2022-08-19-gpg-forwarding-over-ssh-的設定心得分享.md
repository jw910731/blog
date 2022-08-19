---
title: GPG forwarding over SSH 的設定心得分享
date: 2022-08-19T17:33:45.761Z
description: 說說我在設定 GPG forwarding over SSH 時遇到的各種疑難雜症，也紀錄下來設定的流程，避免自己以後忘記。
---
# 前言
最近拿到了人生中第一台筆電，遇到的困擾之一就是要處理連回舊有的 Linux 桌機（工作站）。此時會遇到我需要對 git commit 做簽章但 Yubikey 在我手上，遠端電腦無法存取簽章的問題出現，因此需要 GPG forwarding 到遠端電腦上，才能在桌機上繼續完成我的專案。

# 大致參考的資料
## 1. [GnuPG (PGP) SmartCard over SSH to a VM with a Yubikey](https://dev.to/leehambley/gnupg-pgp-smartcard-over-ssh-to-a-vm-with-a-yubikey-kio)
## 2. [GnuPG 的官方 wiki](https://wiki.gnupg.org/AgentForwarding)
關鍵就在 `/run/user/<uid>/gnupg/` 的資料夾有可能不存在，這個資料夾是由 systemd 管理，在使用者「登入」（嚴謹的說是使用 PAM 登入）時建立的。如果這個資料夾不存在，我們 forwarding 就會失敗，因為我們沒有放 unix socket 的目錄，所以我們需要在登入時建立這個目錄，具體的做法就是在 `.zshrc` 或 `.bashrc` 塞入 `gpgconf --create-socketdir` 就好，這個指令會想辦法處理目錄不存在的問題。
