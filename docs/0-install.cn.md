---
x-title: 通过 APT 安装 x-cmd
x-desc: 在 Debian 或 Ubuntu 上添加官方 x-cmd APT 仓库，安装 x-cmd 包，一行命令即可验证。
x-sidebar: 安装
x-keywords: x-cmd, APT, Debian, Ubuntu, 安装, sources.list, trusted=yes
x-json-ld:
  '@context': https://schema.org
  '@graph':
    - '@type': TechArticle
      headline: '通过 APT 安装 x-cmd'
      inLanguage: 'zh-Hans'
      about: '在 Debian 和 Ubuntu 上通过 APT 安装 x-cmd'
---

# 通过 APT 安装 x-cmd

> 官方 x-cmd APT 仓库，部署在 <https://www.x-cmd.com/deb/>。
> 在任何 Debian 或 Ubuntu 系统上一行命令搞定——目前无需 GPG
> key（仓库未签名，所以要加 `[trusted=yes]`）。

本页面带你走完添加仓库、刷新包索引、安装 `x-cmd` 的全过程。
后面 1..3 页深入讲解 `.deb` 的构建方式、如何校验包、以及仓库
对外发布哪些元数据。

## 快速安装

最快的路径——一行命令，不用编辑器：

```sh
echo "deb [trusted=yes] https://www.x-cmd.com/deb/ stable main" \
  | sudo tee /etc/apt/sources.list.d/x-cmd.list
sudo apt update && sudo apt install -y x-cmd
```

验证：

```sh
x --version
```

应该看到 `0.9.9`（或更新）的版本号。搞定。

## 分步操作

如果你想看清楚每一步：

```sh
# 1. 添加仓库（目前未签名 → [trusted=yes]）
sudo install -m 0644 \
  <(curl -fsSL https://www.x-cmd.com/deb/x-cmd.list) \
  /etc/apt/sources.list.d/x-cmd.list

# 2. 安装 x-cmd
sudo apt update
sudo apt install -y x-cmd
```

`x-cmd.list` 是本仓库和包一起发布的、标准的 sources.list
片段。通过 `install -m 0644` 管道送到标准 APT 路径，权限也
是对的。

## 安装内容

| 项 | 值 |
| --- | --- |
| 包名 | `x-cmd` |
| 版本 | `0.9.9`（或更新） |
| 架构 | `all`（与架构无关的 shell 代码） |
| 维护者 | Li Junhao <l@x-cmd.com> |
| 主页 | <https://www.x-cmd.com> |
| 文件名 | `pool/main/x/x-cmd/x-cmd_0.9.9_all.deb` |
| 大小 | ~4.16 MB |
| SHA-256 | `0e0e24f1d67bf69a3f748f8b1c75e091381292a81ebe87b4f2290859dc40a3a9` |

包声明 `Architecture: all`，所以同一个产物只需索引一次，就
能被 APT 已知的所有架构使用。

## 为什么要 `[trusted=yes]`

仓库目前**未签名**——`etc/apt/trusted.gpg.d/` 下没有 GPG key，
`Release` 文件也没有 `Signed-By` 指令。这是一个已知的缺口
（见 [安全](./2-security.cn.md)，里面写了等 key 发布后加上
`InRelease` 签名的计划）。

`[trusted=yes]` 是 APT 给未签名仓库开的逃生口。它告诉 `apt`
跳过对这个源的签名校验。等签名上线后，把这个选项去掉、加
上 key 就行。

## 手动校验包

如果你不想加仓库，也可以直接抓 `.deb` 装：

```sh
curl -fsSLO https://www.x-cmd.com/deb/pool/main/x/x-cmd/x-cmd_0.9.9_all.deb
echo "0e0e24f1d67bf69a3f748f8b1c75e091381292a81ebe87b4f2290859dc40a3a9  x-cmd_0.9.9_all.deb" | sha256sum -c -
sudo apt install ./x-cmd_0.9.9_all.deb
```

全程不动 sources.list——在严格锁定的 CI runner 或者临时容器
上很方便。

## 下一步

- 想自己打包 `.deb` 通过 APT 分发？看 [打包](./1-packaging.cn.md)。
- 想了解"未签名"实际意味着什么、后续计划是什么？看 [安全](./2-security.cn.md)。
- 想看仓库目前对外发布什么（版本、checksum、架构）？看 [统计](./3-stats.cn.md)。

数据源：[`README.cn.md`](../../README.cn.md)。
