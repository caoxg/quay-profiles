# quay-profiles

[Quay](https://github.com/caoxg/quay)（Arduino® UNO Q 应用管理工具）用的配置，和 Quay 分开发布，改配置不用等 Quay 发版。[English](README.md)

- **App 配置**（`apps/`）：告诉 Quay 一个 App 需要什么——一次性系统设置、启动前的步骤、健康和安全检查、策略插槽、快捷按钮。给还没有自带 [`quay.yaml`](https://github.com/caoxg/quay/blob/main/docs/quay-yaml.md) 的 App 用。
- **系统配置**（`systems/`）：认出原版 Arduino 镜像以外的系统（Duck OS 等），并说明它们和原版有哪些跟 Quay 有关的不同。

Quay 自带一份这个仓库的副本（构建时拍下的），每天大约两次从 GitHub 或 jsDelivr（`cdn.jsdelivr.net/gh/caoxg/quay-profiles@main`）拉新版，哪份的 `updated` 日期新就用哪份。`quay profiles` 显示正在用的是哪份，`quay profiles --refresh` 立即拉取。

> Quay 是独立项目，与 Arduino、陆吾智能都没有关联，也未获其认可。这里的配置是为 Quay 写的，不由厂商维护。

## 目录

```text
index.yaml                 格式版本、更新日期，以及配置文件列表
apps/xgoduck/<提交>.yaml   XGO-Duck（陆吾智能），每个版本一个文件，以它起始的运行时提交命名
systems/duck-os.yaml       Duck OS（陆吾智能给 XGO-Duck 做的系统镜像）
```

**每次修改都要把 `index.yaml` 里的 `updated` 改成新日期**，否则已经有同一日期副本的 Quay 不会去用新的。

## App 配置的版本

App 会变：XGO-Duck 的运行时一开始只有 3 个策略模型，现在有 8 个。所以 App 配置按 App 仓库的提交分版本：每个版本一个文件，用它起始的提交命名（`apps/xgoduck/5f511b9.yaml`），文件里的 `from` 写完整哈希。一个版本管从这个提交起的所有提交，直到下一个版本；比所有版本都早的提交没有配置。各版本 `id` 相同，`index.yaml` 里从旧到新列出。

App 改了配置里描述的东西时，从改动的那个提交起加一个新版本，不要改旧的：装在旧提交上的 App 继续用适合它的版本。Quay 在安装、更新时按 App 的 git 历史挑版本，并记在板子上（`.quay.json` 的 `profileFrom`）。

只有用 Quay 装的 App 才用配置。用别的方式装的 App 对 Quay 来说是普通 App：它匹配的配置只用来告诉「用 Quay 重装」从哪里装。

格式说明（App 配置和系统配置的字段、内置动作）见 [README.md](README.md)。

## 安全

系统配置只能做描述：Quay 只读取它点名的文件（JSON 文件里也只读它点名的字段，别的一概不读）、匹配已保存的网络配置，并在用户点按钮确认后执行 Quay 内置的动作（让网络配置不再开机自动连接；删除没在用的网络配置）。配置没法让 Quay 执行它自己写的命令，所以这里写错了、或者被人篡改了，最坏也只是显示一条不准确的提示。

## 试改动

```bash
QUAY_PROFILES=~/src/quay-profiles quay profiles
QUAY_PROFILES=~/src/quay-profiles quay system     # 在运行该系统的板子上
```

## 许可证

Apache-2.0，与 Quay 相同。Copyright 2026 Xiaogang Cao。
