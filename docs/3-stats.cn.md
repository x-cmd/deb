---
x-title: 仓库统计与对外发布清单
x-desc: 这个 APT 仓库此刻对外发布的文件、大小、checksum 的精确快照，外加 suite/component/architecture 矩阵和 GitHub stars 的人气代理指标。
x-sidebar: 统计
x-keywords: APT, 统计, 包版本, 架构, checksum, Release, Packages, binary-all, binary-amd64, binary-arm64, binary-armhf, binary-i386
x-json-ld:
  '@context': https://schema.org
  '@graph':
    - '@type': TechArticle
      headline: '仓库统计与对外发布清单'
      inLanguage: 'zh-Hans'
      about: '仓库元数据、大小、checksum'
---

# 仓库统计与对外发布清单

> 此刻这个仓库里到底有什么——规范化、精确 pin 死的快照。大小
> 和 checksum 直接从提交的文件读出来，它们变了这一页也跟着变。

## Suite / component / architecture 矩阵

| Suite | Component | Architectures | Source letter | Origin / Label |
| --- | --- | --- | --- | --- |
| `stable` | `main` | `amd64`、`arm64`、`armhf`、`i386`、`all` | `x` | x-cmd / x-cmd |

只有一个包（`x-cmd`），因为 `Architecture: all`，每个架构
索引里都列一次。同一个 payload 在所有 `binary-<arch>/Packages*`
索引里出现。

## 包快照

| 字段 | 值 |
| --- | --- |
| 包名 | `x-cmd` |
| 版本 | `0.9.9` |
| 架构 | `all` |
| 维护者 | Li Junhao <l@x-cmd.com> |
| 文件名 | `pool/main/x/x-cmd/x-cmd_0.9.9_all.deb` |
| 大小 | 4,360,540 字节（~4.16 MB） |
| MD5 | `ad1fc276deb13675071ab29ac1f8e3d6` |
| SHA-1 | `d8fedbd604ce48ac24dae3acea9987cc8081cee1` |
| SHA-256 | `0e0e24f1d67bf69a3f748f8b1c75e091381292a81ebe87b4f2290859dc40a3a9` |
| 主页 | <https://www.x-cmd.com> |
| 简介 | Ultimate Integration of POSIX SHELL AWK. A super weapon built for the super engineer |

## 索引文件（`dists/stable/`）

这里的每个文件在 `dists/stable/Release` 里都有 MD5 / SHA-1 /
SHA-256 / SHA-512 几份 checksum。单位是字节。

| 文件 | 大小 |
| --- | --- |
| `dists/stable/Release` | 4,735 |
| `dists/stable/main/binary-all/Packages` | 438 |
| `dists/stable/main/binary-all/Packages.gz` | 354 |
| `dists/stable/main/binary-amd64/Packages` | 438 |
| `dists/stable/main/binary-amd64/Packages.gz` | 354 |
| `dists/stable/main/binary-arm64/Packages` | 438 |
| `dists/stable/main/binary-arm64/Packages.gz` | 354 |
| `dists/stable/main/binary-armhf/Packages` | 438 |
| `dists/stable/main/binary-armhf/Packages.gz` | 354 |
| `dists/stable/main/binary-i386/Packages` | 438 |
| `dists/stable/main/binary-i386/Packages.gz` | 354 |

五个架构的 `Packages` 文件**字节完全相同**——`x-cmd` 是
`Architecture: all`。APT 只拉匹配本机架构的那一份，但所有
五份都在，镜像方愿意全拉也行。

## 产物（`pool/`）

| 文件 | 大小 |
| --- | --- |
| `pool/main/x/x-cmd/x-cmd_0.9.9_all.deb` | 4,360,540 |

一个二进制，对所有架构镜像。

## 仓库元数据（`dists/stable/Release` 节选）

```
Origin: x-cmd
Label: x-cmd
Suite: stable
Codename: stable
Components: main
Architectures: amd64 arm64 armhf i386 all
Description: x-cmd APT repository
Date: Mon, 22 Jun 2026 20:52:23 +0000
```

## 人气（代理指标）

本仓库本身不统计下载数（纯静态镜像，没分析层）。想要一个人
气代理，看上游 x-cmd 项目：

| 渠道 | URL |
| --- | --- |
| GitHub stars | <https://github.com/x-cmd/x-cmd> |
| GitHub releases | <https://github.com/x-cmd/release> |
| 配套 RPM 仓库 | <https://github.com/x-cmd/rpm> |
| 官网 | <https://www.x-cmd.com> |

## 更新节奏

- 仓库在 x-cmd 上游每次发版时发布。
- CDN 镜像每天从本 GitHub 仓库同步一次。
- 没有固定周期的 CI rebuild——没有 release 就没必要重建。

## 怎么直接拉原始数据

```sh
# Suite 元数据 + 所有 checksum
curl -fsSL https://www.x-cmd.com/deb/dists/stable/Release

# 每个架构的包索引
curl -fsSL https://www.x-cmd.com/deb/dists/stable/main/binary-amd64/Packages

# 包本身
curl -fsSLO https://www.x-cmd.com/deb/pool/main/x/x-cmd/x-cmd_0.9.9_all.deb
```

三个文件都从和首页同一个 CDN 服务出来。

## 下一步

- 想看这些文件怎么生成？看 [打包](./1-packaging.cn.md)。
- 想看安全模型今天承诺什么？看 [安全](./2-security.cn.md)。

数据源：
[`dists/stable/Release`](../../dists/stable/Release)、
[`dists/stable/main/binary-amd64/Packages`](../../dists/stable/main/binary-amd64/Packages)、
[`pool/main/x/x-cmd/x-cmd_0.9.9_all.deb`](../../pool/main/x/x-cmd/x-cmd_0.9.9_all.deb)
截至 `git log` 锁定的提交。
