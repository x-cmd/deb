---
x-title: 自己打包 .deb 并托管为 APT 仓库
x-desc: 如何把 shell 项目组装成 Debian 包、用 dpkg-scanpackages 和 apt-ftparchive 重建索引，并按规范的 pool/ + dists/ 布局托管。
x-sidebar: 打包
x-keywords: deb, dpkg-deb, dpkg-scanpackages, apt-ftparchive, Debian 包, APT 仓库, pool, dists, control 文件
x-json-ld:
  '@context': https://schema.org
  '@graph':
    - '@type': TechArticle
      headline: '自己打包 .deb 并托管为 APT 仓库'
      inLanguage: 'zh-Hans'
      about: '打包 Debian 包并托管 APT 仓库'
---

# 自己打包 .deb 并托管为 APT 仓库

> 一条最短路径：把一个目录里的文件打成（签或不签的）.deb，
> 用 `dpkg-scanpackages` 和 `apt-ftparchive` 重建 APT 索引，
> 然后放到任意静态 CDN 上对外服务。本仓库用的就是这一套。

如果你只想把 shell 脚本、补全、manpage 发给 Debian/Ubuntu
用户、不想走官方 archive 流程，这条路最短。

## `.deb` 到底是什么

`.deb` 本质上是一个 `ar` 归档，里面三个文件：

```
deb_extract/
├── control.tar.gz   # 元数据：包名、依赖、脚本
├── data.tar.gz      # 真正要安装的文件
└── debian-binary    # 字面量 "2.0\n"
```

不用手搓，`dpkg-deb -b` 帮你搞定。

## 1. 摆源码树

```
myapp-1.2.3/
├── DEBIAN/
│   ├── control       # 必填
│   ├── conffiles     # 可选
│   ├── postinst      # 可选
│   ├── prerm         # 可选
│   └── ...
└── usr/
    ├── bin/myapp
    ├── share/man/man1/myapp.1
    └── share/myapp/...
```

`DEBIAN/` 放元数据；其余目录映射装好之后的文件系统布局。

## 2. 写 `DEBIAN/control`

最重要的一个文件。最小可跑版本：

```
Package: myapp
Version: 1.2.3
Architecture: all
Maintainer: 你的名字 <you@example.com>
Description: 一句话简介
 多行更长描述（缩进，可任意行数）
```

按需加字段：`Depends`、`Recommends`、`Section`、`Priority`、
`Homepage`。完整字段表看 `man 5 deb-control`。

## 3. 打 `.deb`

```sh
dpkg-deb --build --root-owner=group myapp-1.2.3
# → myapp-1.2.3.deb
```

`--root-owner=group` 是现代写法，不用 root 也能设属主；纯
shell 包用默认的 `root:root` 也行。

## 4. 放进 `pool/`

APT 标准布局是 `pool/<component>/<首字母>/<源名>/<file>`：

```
repo/
├── pool/main/m/myapp/myapp_1.2.3_all.deb
├── dists/stable/Release
└── dists/stable/main/binary-all/Packages*
```

`pool/` 下路径按**源包名**（不是二进制名），首字母取源包
名首字母。`myapp` 就是 `pool/main/m/myapp/`。

## 5. 重建索引

`dpkg-scanpackages` 扫 `pool/` 生成各架构的 `Packages`，
`apt-ftparchive release` 生成 suite 级的 `Release`（带所有
索引的 checksum）。在 Debian 容器里跑（macOS 没这些工具）：

```sh
docker run --rm -v "$PWD:/repo" debian:latest bash -lc '
  export DEBIAN_FRONTEND=noninteractive
  apt-get update -qq
  apt-get install -y -qq --no-install-recommends dpkg-dev apt-utils >/dev/null
  cd /repo

  for a in amd64 arm64 armhf i386 all; do
    out="dists/stable/main/binary-$a"; mkdir -p "$out"
    dpkg-scanpackages --multiversion --arch "$a" pool /dev/null \
      > "$out/Packages"
    gzip -9c "$out/Packages" > "$out/Packages.gz"
  done

  rm -f dists/stable/Release
  apt-ftparchive release \
    -o APT::FTPArchive::Release::Origin=myorg \
    -o APT::FTPArchive::Release::Label=myorg \
    -o APT::FTPArchive::Release::Suite=stable \
    -o APT::FTPArchive::Release::Codename=stable \
    -o APT::FTPArchive::Release::Architectures="amd64 arm64 armhf i386 all" \
    -o APT::FTPArchive::Release::Components=main \
    -o APT::FTPArchive::Release::Description="My APT repository" \
    dists/stable > /tmp/Release && cp /tmp/Release dists/stable/Release
'
```

三个细节：

- `Release` 在**扫描树外**生成再拷进来，避免带自指的
  checksum 条目。
- `--multiversion` 让 `pool/` 里历史版本都进索引（方便回滚）。
- 每个发布的架构都跑一遍。`all` 是特例——它和每个 per-arch
  索引都列出同一个架构无关的包。

## 6. 服务出去

`pool/` + `dists/` 都是纯静态文件。任意 HTTP server 都行——
GitHub Pages、Cloudflare R2、自建 nginx，看你。用户加一行
`sources.list`：

```
deb [trusted=yes] https://example.com/deb/ stable main
```

……等你上了 GPG 签名（见 [安全](./2-security.cn.md)），就去掉
`[trusted=yes]`，加 `Signed-By`。

## 7. 加 `x-cmd.list` snippet

仓库根放一个可直接复制的 sources.list 片段对 landing 在
GitHub 页面的人类友好：

```
# myapp APT 仓库  (suite: stable, component: main)
#   curl -fsSL https://example.com/deb/myapp.list \
#     | sudo tee /etc/apt/sources.list.d/myapp.list
#   sudo apt update && sudo apt install -y myapp
deb [trusted=yes] https://example.com/deb/ stable main
```

本仓库就是这个套路——见 [`x-cmd.list`](../../x-cmd.list)。

## 什么时候不要走这条

- **想让包进 Debian / Ubuntu 官方 archive**——走
  `ftp-master.debian.org` 或者找 sponsor；自托管适合不想
  走流程的项目。
- **有多架构原生二进制**——用 `sbuild` + 类 Launchpad 的
  pipeline；上面的手工方法只适合纯 shell / `Architecture: all`。
- **要可复现构建 / SBOM**——加 `dpkg-buildpackage` + CI 矩阵；
  这里只给你二进制，没给源码到二进制的可复现路径。

## 下一步

- 想给 `Release` 加 GPG 签名？看 [安全](./2-security.cn.md)。
- 想看本仓库对外的标准布局？看 [统计](./3-stats.cn.md)。

数据源：本仓库的 [`README.cn.md`](../../README.cn.md) 及上游
Debian handbook 的 APT 仓库章节。
