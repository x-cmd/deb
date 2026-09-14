# x-cmd APT 仓库

> `https://www.x-cmd.com/deb/` —— 一个标准 Debian 归档（APT
> 仓库），托管官方 **x-cmd** 包。

本仓库按 Debian 标准布局（`pool/` + `dists/`）原样发布到
`https://www.x-cmd.com/deb/`，所以对任何已列出的架构都能
被 `apt` 直接消费。

包由上游
[x-cmd/release](https://github.com/x-cmd/release) 构建并发布；
本仓库只镜像 `.deb` 产物并维护仓库索引。包是架构无关的
（`Architecture: all`），所以一次索引，所有架构都受益。

---

## 安装（Debian / Ubuntu）

```sh
# 1. 添加仓库（目前未签名 → [trusted=yes]）
sudo install -m 0644 \
  <(curl -fsSL https://www.x-cmd.com/deb/x-cmd.list) \
  /etc/apt/sources.list.d/x-cmd.list

# 2. 安装 x-cmd
sudo apt update
sudo apt install -y x-cmd
```

一行版本：

```sh
echo "deb [trusted=yes] https://www.x-cmd.com/deb/ stable main" \
  | sudo tee /etc/apt/sources.list.d/x-cmd.list
sudo apt update && sudo apt install -y x-cmd
```

验证：

```sh
x --version
```

---

## 仓库布局

```
.
├── pool/
│   └── main/x/x-cmd/x-cmd_0.9.9_all.deb   # 托管的包
├── dists/
│   └── stable/                              # suite = stable, component = main
│       ├── main/
│       │   ├── binary-amd64/Packages(.gz)
│       │   ├── binary-arm64/Packages(.gz)
│       │   ├── binary-armhf/Packages(.gz)
│       │   ├── binary-i386/Packages(.gz)
│       │   └── binary-all/Packages(.gz)
│       └── Release                          # 每个索引文件的 checksum
├── x-cmd.list                               # 开箱即用的 sources.list 片段
├── README.md                                # 英文 README
├── README.cn.md                             # 本文件
└── docs/                                    # 双语文档 + FAQ + llms
    ├── 0-install.{md,cn.md,en.md,llms.md,faq.yml}
    ├── 1-packaging.{md,cn.md,en.md,llms.md,faq.yml}
    ├── 2-security.{md,cn.md,en.md,llms.md,faq.yml}
    └── 3-stats.{md,cn.md,en.md,llms.md,faq.yml}
```

`apt` 先拉 `dists/stable/Release`，再拉匹配本机架构（外加
`binary-all`）的 `Packages.gz`。`Architecture: all` 的包在
每个 per-arch 索引里都列出来，所以同一个仓库给 `amd64`、
`arm64`、`armhf`、`i386` 都用。

---

## 新版本发布

1. 把新的 `.deb` 放进 `pool/main/x/x-cmd/`（可选地删掉
   过期的）。用 `dpkg-scanpackages` 重建索引；用
   `apt-ftparchive` 重建 `Release`（这两个工具都是 Debian
   自带的——macOS 上跑容器）：

   ```sh
   docker run --rm -v "$PWD:/repo" debian:latest bash -lc '
     export DEBIAN_FRONTEND=noninteractive
     apt-get update -qq
     apt-get install -y -qq --no-install-recommends dpkg-dev apt-utils >/dev/null
     cd /repo
     for a in amd64 arm64 armhf i386 all; do
       out="dists/stable/main/binary-$a"; mkdir -p "$out"
       dpkg-scanpackages --multiversion --arch "$a" pool /dev/null > "$out/Packages"
       gzip -9c "$out/Packages" > "$out/Packages.gz"
     done
     rm -f dists/stable/Release
     apt-ftparchive release \
       -o APT::FTPArchive::Release::Origin=x-cmd \
       -o APT::FTPArchive::Release::Label=x-cmd \
       -o APT::FTPArchive::Release::Suite=stable \
       -o APT::FTPArchive::Release::Codename=stable \
       -o APT::FTPArchive::Release::Architectures="amd64 arm64 armhf i386 all" \
       -o APT::FTPArchive::Release::Components=main \
       -o APT::FTPArchive::Release::Description="x-cmd APT repository" \
       dists/stable > /tmp/Release && cp /tmp/Release dists/stable/Release'
   ```

   `Release` 在扫描树外生成，避免带自指的 checksum 条目。

2. 提交新的 `.deb`、`dists/stable/main/binary-*/Packages*`、
   `dists/stable/Release`，push。网站每天从本分支刷新一次。

---

## 说明

- 仓库和包目前**未签名**，所以要 `[trusted=yes]`。等 GPG
  key 发布后，切换成签过名的 `InRelease`（去掉 `[trusted=yes]`）。
- 支持的架构：`amd64`、`arm64`、`armhf`、`i386`、`all`。
  加新架构：新建 `binary-<arch>/` 索引，在 `Release` 的
  `Architectures` 一节加上。
- 构建出处和源码在
  [x-cmd/release](https://github.com/x-cmd/release)。
- 配套 RPM 仓库：[x-cmd/rpm](https://github.com/x-cmd/rpm) →
  `https://www.x-cmd.com/rpm/`。

---

## 文档站

用户文档在 `docs/`，双语 + FAQ + llms 四件套，结构对齐
[x-cmd/cve](https://github.com/x-cmd/cve)：

| 页面 | 内容 |
| --- | --- |
| [`docs/0-install.*`](docs/0-install.md) | 怎么装：sources.list、`apt install`、手动 `.deb` 校验 |
| [`docs/1-packaging.*`](docs/1-packaging.md) | 怎么自己打包 + 托管 APT 仓库 |
| [`docs/2-security.*`](docs/2-security.md) | `[trusted=yes]` 真相 + 签名计划 |
| [`docs/3-stats.*`](docs/3-stats.md) | 仓库对外发布什么（版本、大小、checksum） |
