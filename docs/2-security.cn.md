---
x-title: 安全、签名与 [trusted=yes] 的真相
x-desc: 这个 APT 仓库今天为什么没签名，`[trusted=yes]` 实际上打开了什么，如何手动校验 checksum，以及加 InRelease / GPG 签名去掉逃生口的计划。
x-sidebar: 安全
x-keywords: APT, 签名, GPG, InRelease, trusted=yes, sha256, 供应链, deb822, sources.list
x-json-ld:
  '@context': https://schema.org
  '@graph':
    - '@type': TechArticle
      headline: '安全、签名与 [trusted=yes] 的真相'
      inLanguage: 'zh-Hans'
      about: 'APT 仓库安全、签名与校验'
---

# 安全、签名与 `[trusted=yes]` 的真相

> 把这个仓库今天承诺什么、不承诺什么，完整、诚实地写出来。
> 你是安全审计员或者谨慎的运维，看这一页。

本仓库目前**未签名**。`x-cmd.list` 里的 `[trusted=yes]` 是
显式的 opt-in，承认这件事。下面有加真实 GPG 签名、撤掉这个
逃生口的计划。

## 今天的现状

| 属性 | 状态 |
| --- | --- |
| CDN TLS | ✅ (https://www.x-cmd.com/deb/) |
| `Packages` 里每个包的 SHA-256 | ✅ |
| Suite 级 `Release` 文件，含所有索引 checksum | ✅ |
| `Release.gpg` detached 签名 | ❌ |
| `InRelease` clear-signed `Release` | ❌ |
| `/etc/apt/trusted.gpg.d/` 下的 GPG key | ❌ |
| 带过期时间的 key + 轮换策略 | ❌（还没有 key） |

前三行是你现在就能依靠的完整性层。后四行就是 `[trusted=yes]`
静默关掉的检查。

## `[trusted=yes]` 实际打开的是什么

APT 的签名校验挡两种风险：

1. **网络 MITM**：TLS-strip 攻击者改写你要下的 `.deb`。
2. **仓库被攻陷**：操作员（或拿到操作员账号的人）推了你
   不想装的 `.deb`。

`[trusted=yes]` 对**这一个源**把两种都关掉。`https://` 仍
然给你 TLS，所以 MITM 这条挡住了。操作员信任这条就是你现
在只能靠"操作员名声"——和从 PyPI 装包不校验 hash、或
`curl | bash` 一个 README 是同一类信任模型。

对 x-cmd 具体说：仓库在 GitHub（你能审计每个 commit），通
过标准只读 pipeline 镜像到 `www.x-cmd.com`，`.deb` 由和
公开 x-cmd release artifact 同一份源确定性地打出。你可以把
SHA-256 跟 GitHub release 上的 checksum 文件对比——这就是
今天实际的安全网。

## 不加仓库也能校验

```sh
URL="https://www.x-cmd.com/deb/pool/main/x/x-cmd/x-cmd_0.9.9_all.deb"
EXPECTED="0e0e24f1d67bf69a3f748f8b1c75e091381292a81ebe87b4f2290859dc40a3a9"

curl -fsSLO "$URL"
echo "$EXPECTED  x-cmd_0.9.9_all.deb" | sha256sum -c -
```

`OK` 表示你抓下来的字节和 `dists/stable/main/binary-all/Packages`
里声称的一致。把期望的 hash 通过另一个渠道（GitHub release
页面、Twitter、朋友）再交叉对比一次，就是完整意义上的可信。

要更严谨的话，先抓 `dists/stable/Release`，看里面对
`pool/main/x/x-cmd/x-cmd_0.9.9_all.deb` 的 SHA-256，再跟实际
下载的文件对比：

```sh
curl -fsSL https://www.x-cmd.com/deb/dists/stable/Release
# SHA256 一节里就有期望的 hash
```

## 签名计划

操作员发布 GPG key 后，会做这些事：

1. Key（长期 + 一个 APT 用 sub-key）发布到稳定 URL，并在
   x-cmd 博客 + GitHub release 同步公告。
2. 重建索引的 CI 同时输出 `dists/stable/Release`、
   `dists/stable/InRelease`（clear-signed）和
   `dists/stable/Release.gpg`（detached）。
3. `sources.list` 去掉 `[trusted=yes]`，加上 `Signed-By:`
   指令（deb822 风格），或者把 key 放到
   `/etc/apt/keyrings/x-cmd.gpg`（传统风格）。
4. `apt update` 一旦签名缺失、无效、或者不是预期的 key，
   就拒绝从这个源安装——和任何其它签名 APT 源同等级别的保护。

这些工作还没发布之前，**不要**在你审计不到的生产集群里盲
信这个仓库。在开发机用、或者把版本 pin 死。

## 今天我们**挡不住**的供应链风险

- **`x-cmd/deb` 账号被拿下**：被黑的 GitHub token 可以推一
  个恶意 `.deb`。缓解：给 `Release` 加 GPG 签名后，任何
  推 push 都需要操作员的签名 key，而不只是 commit 权限。
- **镜像被攻陷**：`www.x-cmd.com/deb/` 是静态镜像，如果
  在 TLS 层之前被替换，你拿到的是攻击者的字节。缓解：同
  样的 GPG 签名能挡住。
- **重放 / 降级**：TLS-MITM 攻击者可以发一个更老、有已知
  漏洞的 `.deb` 给你。缓解：签名上线后，APT 对 `Release`
  的日期检查 + key 签名能挡住。

## 源出处

`.deb` 由上游 x-cmd release 流水线
([`x-cmd/release`](https://github.com/x-cmd/release)) 打出，
逐字镜像到本仓库。构建的可复现性跟上游流水线一致——对
pin 了 commit SHA 的纯 shell 内容来说，一般都是 yes。

## 下一步

- 想看现在对外发布什么？看 [统计](./3-stats.cn.md)。
- 想自己审计包元数据？看 [打包](./1-packaging.cn.md) 里
  的索引布局。

数据源：本仓库的 [`README.cn.md`](../../README.cn.md) 及
上游 APT 签名文档。
