---
title: 快速搭建簡易 Git Server
published: 2025-12-06
description: ''
image: ''
tags: [Guides, Service, Git]
draft: false 
lang: 'zh-TW'
---

## 為什麼要再寫一篇 Git Server 的架設教學?

在之前的文章教了各位要如何架設 Gitea Git Server，Gitea 的安裝步驟其實已經夠少了，但如果不想額外安裝套件該怎麼麼做呢？

此篇文章將教你搭建更簡易的 Git Server，只要安裝 Git 就夠了！

## 開始設定

首先在伺服器端及客戶端安裝 Git，輸入下方指令進行安裝
```bash
sudo apt update
sudo apt install git
```

接著在伺服器端創建 `git` 使用者，切換到該使用者並建立目錄
```bash
sudo adduser git
su - git
mkdir .ssh && chmod 700 .ssh
touch .ssh/authorized_keys && chmod 600 .ssh/authorized_keys
```

在客戶端產生金鑰，之後複製公鑰到伺服器端 `git` 使用者下的 `.ssh/authorized_keys`
```bash
# 客戶端
ssh-keygen
cat .ssh/id_ed25519.pub     # 複製輸出，然後到伺服器端進行設定

# 伺服器端
su - git
echo "公鑰內容" >> .ssh/authorized-keys
```

使用有 `sudo` 權限的使用者，建立 `/opt/git` 資料夾，並給予 `git` 使用者該資料夾的擁有權限
```bash
sudo mkdir /opt/git
sudo chown git:git /opt/git
```

切回 `git` 使用者，進入 `/opt/git` 為客戶端建立空儲存庫
```bash
su - git
cd /opt/git
mkdir hello.git
cd hello.git
git init --bare
```

接著就可以回到客戶端為其加入遠程儲存庫了
```bash
cd hello
git init
git add .
git commit -m "Initial commit"
git remote add origin git@git-server-ip:/opt/git/hello.git 
git push origin master
```

為了防止客戶端直接訪問伺服器的 `git` 使用者，我們還需要進行額外設定

首先修改 `.ssh/authorized-keys`，在每個公鑰前加入以下設定防止客戶端透過端口轉發訪問 git 伺服器
```bash
su - git
vim .ssh/authorized-keys

# 要加的是這個
no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty
```

接著更改 shell，限制客戶端的操作
```bash
su - git
mkdir $HOME/git-shell-commands
touch $HOME/git-shell-commands/no-interactive-login
vim $HOME/git-shell-commands/no-interactive-login
```

```bash
#!/bin/sh
printf '%s\n' "Hi $USER! You've successfully authenticated, but I do not"
printf '%s\n' "provide interactive shell access."
exit 128
```

```bash
chmod +x $HOME/git-shell-commands/no-interactive-login
chsh git -s $(which git-shell)
```

## 結語

這樣就完成了一個簡易的 Git Server 架設，不用寫系統設定，有 `ssh` 跟 `git` 就可以搭建了，我們下次見！