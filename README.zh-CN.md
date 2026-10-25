# quay-profiles

[Quay](https://github.com/caoxg/quay)（Arduino® UNO Q 应用管理工具）用的配置，和 Quay 分开发布，改配置不用等 Quay 发版。[English](README.md)

- **App 配置**（`apps/`）：告诉 Quay 一个 App 需要什么——一次性系统设置、启动前的步骤、健康和安全检查、策略插槽、快捷按钮。给还没有自带 [`quay.yaml`](https://github.com/caoxg/quay/blob/main/docs/quay-yaml.md) 的 App 用。
- **系统配置**（`systems/`）：认出原版 Arduino 镜像以外的系统（Duck OS 等），并说明它们和原版有哪些跟 Quay 有关的不同。

Quay 自带一份这个仓库的副本（构建时拍下的），每天大约两次从 GitHub 或 jsDelivr（`cdn.jsdelivr.net/gh/caoxg/quay-profiles@main`）拉新版，哪份的 `updated` 日期新就用哪份。`quay profiles` 显示正在用的是哪份，`quay profiles --refresh` 立即拉取。

> Quay 是独立项目，与 Arduino、陆吾智能都没有关联，也未获其认可。这里的配置是为 Quay 写的，不由厂商维护。

## 目录

```text
index.yaml           格式版本、更新日期，以及配置文件列表
apps/xgoduck.yaml    XGO-Duck（陆吾智能）
systems/duck-os.yaml Duck OS（陆吾智能给 XGO-Duck 做的系统镜像）
```

**每次修改都要把 `index.yaml` 里的 `updated` 改成新日期**，否则已经有同一日期副本的 Quay 不会去用新的。

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
