---
title: GPG forwarding over SSH 的設定心得分享
date: 2022-08-19T17:33:45.761Z
description: 說說我在設定 GPG forwarding over SSH 時遇到的各種疑難雜症，也紀錄下來設定的流程，避免自己以後忘記。
---
# 前言
最近拿到了人生中第一台筆電，遇到的困擾之一就是要處理連回舊有的 Linux 桌機（工作站）。此時會遇到我需要對 git commit 做簽章但 Yubikey 在我手上，遠端電腦無法存取簽章的問題出現，因此需要 GPG forwarding 到遠端電腦上，才能在桌機上繼續完成我的專案。

# 大致參考的資料
