# WR1200JS 原版固件构建

ImmortalWrt **21.02.7** 官方镜像 + ksmbd 共享 + 中文界面。没有任何内核参数调优，没有 zram，就是官方原版加几个包。

- 目标平台：`ramips/mt7621`
- 设备 Profile：`youhua_wr1200js`
- 硬件：MT7621AT 双核 880MHz / 16MB Flash / 128MB RAM / 5×千兆 / USB2.0

## 当前装的包

```
luci-i18n-base-zh-cn     LuCI 中文界面
ksmbd-server             SMB 服务（内核态）
luci-app-ksmbd           SMB 管理界面
luci-i18n-ksmbd-zh-cn    SMB 界面中文
```

## 怎么出固件

Actions → **Build WR1200JS Firmware** → **Run workflow**

- `version` 默认 `21.02.7`，也可以选 24.10.7 / 23.05.7 / 25.12.2
- `extra_packages` 留空即可

5~8 分钟后在本次运行页底部 **Artifacts** 下载 `immortalwrt-21.02.7-wr1200js.zip`。

## 刷机

```sh
sysupgrade -n /tmp/immortalwrt-21.02.7-ramips-mt7621-youhua_wr1200js-squashfs-sysupgrade.bin
```

网页方式：系统 → 备份/升级 → 刷写新固件，选 `*-squashfs-sysupgrade.bin`。
换版本建议**不勾选保留配置**。

## 加插件

编辑 `configs/packages.txt`，删掉行首 `#` 就是启用，提交后自动触发构建。

16MB Flash 是硬限制，官方镜像本体约 8MB，实际可加约 7MB：

- 轻量（sqm / ddns / wol / upnp / ttyd / argon 主题）随便加
- PassWall + xray 约 10MB，压缩后 4~6MB，能装但要单独加
- OpenClash / Docker / Alist 基本塞不进，硬塞会 `Image too big`

## 关于大文件复制卡顿

21.02 内核 5.4，**没有 ntfs3 驱动**，NTFS 盘只能走 `ntfs-3g`（FUSE 用户态，比较吃内存）。
如果 U 盘是 NTFS，复制大文件时会卡，原因是 page cache 脏页堆积。

两个不改固件的缓解办法：

1. U 盘格式化成 **ext4**（最有效，内存占用和速度都更好）
2. 手动调脏页上限（刷完在 SSH 里执行，重启失效）：

```sh
sysctl -w vm.dirty_ratio=8 vm.dirty_background_ratio=3 vm.min_free_kbytes=12288
```

要永久生效就写进 `/etc/sysctl.conf`。需要写进固件的话告诉我，我加回去。
