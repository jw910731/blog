---
title: GPG forwarding over SSH 的設定心得分享
date: 2022-08-19T17:33:45.761Z
description: 說說我在設定 GPG forwarding over SSH 時遇到的各種疑難雜症，也紀錄下來設定的流程，避免自己以後忘記。
---
# 前言
最近拿到了人生中第一台筆電，遇到的困擾之一就是要處理連回舊有的 Linux 桌機（工作站）。此時會遇到我需要對 git commit 做簽章但 Yubikey 在我手上，遠端電腦無法存取簽章的問題出現，因此需要 GPG forwarding 到遠端電腦上，才能在桌機上繼續完成我的專案。

# 過程
照著教學做的我會附上連結並擷取重點，有些自己悟出來的就會寫細一點
## [GnuPG (PGP) SmartCard over SSH to a VM with a Yubikey](https://dev.to/leehambley/gnupg-pgp-smartcard-over-ssh-to-a-vm-with-a-yubikey-kio)
主要先要找到需要 forward 的 socket
```bash
# 在本機上執行：
$ gpgconf --list-dirs agent-ssh-socket
/Users/<username>/.gnupg/S.gpg-agent.ssh
$ gpgconf --list-dir agent-socket
/Users/leehambley/.gnupg/S.gpg-agent
$ gpgconf --list-dirs agent-extra-socket
/Users/<username>/.gnupg/S.gpg-agent.extra


# 在遠端執行：
$ gpgconf --list-dirs agent-ssh-socket
/run/user/<uid>/gnupg/S.gpg-agent.ssh
$ gpgconf --list-dirs agent-socket
/run/user/<uid>/gnupg/S.gpg-agent
```
找到之後就可以開始對 `.ssh/config` 開刀了，指示 ssh 幫忙 forward gpg unix socket。
修改本機的 `$HOME/.ssh/config`
```
Host <hostname>
    HostName <hostname>
    # 這裡是 GPG/SSH forwarding 相關的設定 
    RemoteForward /run/user/<uid>/gnupg/S.gpg-agent /Users/<username>/.gnupg/S.gpg-agent.extra
    RemoteForward /run/user/<uid>/gnupg/S.gpg-agent.ssh /Users/<username>/.gnupg/S.gpg-agent.ssh
    ForwardAgent yes
    ExitOnForwardFailure yes # 確定你有 forward 成功，失敗直接退出
```

然後我還莫名其妙地加上了一段我不懂的修改，修改遠端的 `/etc/ssh/sshd_config`，加上
```
StreamLocalBindUnlink yes
```
相關的文獻可以參考[這篇文章](https://superuser.com/questions/161973/how-can-i-forward-a-gpg-key-via-ssh-agent)
## [GnuPG 的官方 wiki](https://wiki.gnupg.org/AgentForwarding)
關鍵就在 `/run/user/<uid>/gnupg/` 的資料夾有可能不存在，這個資料夾是由 systemd 管理，在使用者「登入」（嚴謹的說是使用 PAM 登入）時建立的。如果這個資料夾不存在，我們 forwarding 就會失敗，因為我們沒有放 unix socket 的目錄，所以我們需要在登入時建立這個目錄，具體的做法就是在**遠端**的 `.zshrc` 或 `.bashrc` 塞入 
```
gpgconf --create-socketdir
```
就好，這個指令會想辦法處理目錄不存在的問題。

## 通靈
我發現讓 ssh 透過 PAM 登入很重要，要從遠端機器修改 `/etc/ssh/sshd_config` 來讓他透過 PAM 驗證，這樣就會跑出 `/run/user/<uid>` 的資料夾。
如下修改 `/etc/ssh/sshd_config` ：
```
# 原本這裡是 no
UsePAM yes
```
小提醒一下，隨時靠 `gpgconf --kill gpg-agent` 來重啟 gpg agent ，這樣調整才會生效（當然你如果調了 `/etc/ssh/sshd_config` 就要換重開 sshd service ）
# 結語
這樣差不多就完成了，我有聽說有人即使 forwarding 成功還是沒辦法做加解密和簽章的，只能說太可憐了，我這裡 forwarding 成功後就可以使用了。
終於可以是完善了整個開發環境了，讓我可以利用遠端強大的算力與充沛的硬碟空間來輔助筆電端。