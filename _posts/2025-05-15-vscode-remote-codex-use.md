---
layout: post
title: vscode-remote codex配置
date: 2026-05-15
author: CIG
categories:
- tooling
tags:
- codex
---

> vscode-remote codex配置网络问题

## 1.问题

如果你在使用codex的过程中遇到以下问题：

- Token exchange failed: token endpoint returned status 403 Forbidden: Country, region, or territory not supported
- stream disconnected before completion: error sending request for url (https://chatgpt.com/backend-api/codex/responses)

一般都是有网络问题引起。

## 2.原因

就算开了代理，可能会出现插件没走代理而直连auth.openai.com。

## 3.解决办法

改这两个文件：
~/.vscode-server/server-env-setup
~/.bashrc
固定为你的代理端口。

server-env-setup:
```
#!/bin/sh

# Keep VS Code Remote and Codex on a user-writable temp root.
_runtime_root="${XDG_RUNTIME_DIR:-/run/user/$(id -u)}"
_vscode_tmp="$_runtime_root/vscode-tmp"
mkdir -p "$_vscode_tmp" "$_vscode_tmp/codex-ipc"
chmod 700 "$_vscode_tmp" "$_vscode_tmp/codex-ipc" 2>/dev/null || true
export TMPDIR="$_vscode_tmp"

# Keep Codex/OpenAI auth and API traffic on the verified local proxy.
# The clashctl-generated HTTP port ** currently routes auth.openai.com direct,
# which returns unsupported_country_region_territory from this host.
_codex_proxy_url="http://127.0.0.1:7890"
export http_proxy="$_codex_proxy_url"
export HTTP_PROXY="$_codex_proxy_url"
export https_proxy="$_codex_proxy_url"
export HTTPS_PROXY="$_codex_proxy_url"
export all_proxy="$_codex_proxy_url"
export ALL_PROXY="$_codex_proxy_url"
export no_proxy="localhost,127.0.0.1,::1"
export NO_PROXY="$no_proxy"
unset _codex_proxy_url _runtime_root _vscode_tmp
```
.bashrc:
```
# Codex/OpenAI proxy environment for lab.
if [ -f "$HOME/.vscode-server/server-env-setup" ]; then
  . "$HOME/.vscode-server/server-env-setup"
fi

```