---
title: Windows 上使用 SSH 签名 Git 提交记录
slug: signing-git-commits-with-ssh-keys
date: "2022-08-30"
updated: "2022-09-15"
description: "GitHub 官宣正式支持显示 SSH 签名，因此你可以使用自己生成的 SSH 密钥在本地签名提交记录以防止他人冒充伪造提交记录"
tags: reactjs, css, python, nodejs
domain: dev.tobychung.com
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1649662225945/7f_c6UxhR.jpg?auto=compress
---

## 前提

### 生成新 SSH 密钥并复制

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

```bash
clip < ~/.ssh/id_ed25519.pub
```

将结果添加到 GitHub 的 SSH and GPG keys 中。

### 将 SSH 密钥添加到 ssh-agent

- 确保 ssh-agent 正在运行

```bash
eval "$(ssh-agent -s)"
```

- 将 SSH 私钥添加到 ssh-agent

```bash
ssh-add ~/.ssh/id_ed25519
```

## 使用 SSH 签名

- 全局使用 SSH 签名

```bash
git config --global gpg.format ssh
```

- 指定 SSH 签名使用文件

```bash
git config --global user.signingKey ~/.ssh/id_ed25519.pub
```

- 在 .ssh 目录新建一个可信公钥列表文件 `allowed_signers`, 内容为 `id_ed25519.pub` 前面加上邮箱

```bash
touch ~/.ssh/allowed_signers && clip < ~/.ssh/id_ed25519.pub
```

```bash
echo "your_email@example.com 粘贴到这" >> ~/.ssh/allowed_signers
```

例如我的列表文件 allowed_signers

```bash
tobychung@duck.com ssh-ed25519 AAAA(...已省略)RuQj tobychung@duck.com
```

- 指定可信公钥列表文件

```bash
git config --global gpg.ssh.allowedSignersFile ~/.ssh/allowed_signers
```

- 开启全局自动签名

```bash
git config --global commit.gpgsign true
git config --global tag.gpgsign true
```

## 测试现有仓库

```bash
git commit --allow-empty --message="Testing SSH signing"
```

## 查看签名

```bash
git log --show-signature -3
```

这个时候我们就可以看到第一条有`Good "git" signatures ...`验证消息了。

## GitHub 认证标识

点击账号设置 SSH and GPG keys 中勾选 Vigilant mode![勾选认证]((2022-08-30) 使用 SSH 签名 Git 提交记录/勾选认证.png)

再次添加相同的公钥重点 `Key type` 要选择 `signing key` 即可。

```bash
clip < ~/.ssh/id_ed25519.pub
```

![Testing SSH signing](https://cdn.hashnode.com/res/hashnode/image/upload/v1663318561458/AKt-iZOfZ.png)

## 参考

- [SSH commit verification now supported | GitHub Changelog](https://github.blog/changelog/2022-08-23-ssh-commit-verification-now-supported/)
- [About commit signature verification - GitHub Docs](https://docs.github.com/en/authentication/managing-commit-signature-verification/about-commit-signature-verification#ssh-commit-verification)
- [使用 SSH 签名 Git 提交记录](https://taoshu.in/git/ssh-sign.html)
- [Signing Git Commits with Your SSH Key](https://calebhearth.com/sign-git-with-ssh)
