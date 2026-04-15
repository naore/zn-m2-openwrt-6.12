# OpenWRT-CI

官方版：

https://github.com/immortalwrt/immortalwrt.git

高通版：

https://github.com/VIKINGYFY/immortalwrt.git

# U-BOOT

高通版：

https://github.com/chenxin527/uboot-ipq60xx-emmc-build

https://github.com/chenxin527/uboot-ipq60xx-nand-build

https://github.com/chenxin527/uboot-ipq60xx-nor-build

联发科版：

https://drive.wrt.moe/uboot/mediatek

# 固件简要说明

固件每天早上4点自动编译。

固件信息里的时间为编译开始的时间，方便核对上游源码提交时间。

MEDIATEK系列、QUALCOMMAX系列、ROCKCHIP系列、X86系列。

# 目录简要说明

workflows——自定义CI配置

Scripts——自定义脚本

Config——自定义配置

#
[![Stargazers over time](https://starchart.cc/VIKINGYFY/OpenWRT-CI.svg?variant=adaptive)](https://starchart.cc/VIKINGYFY/OpenWRT-CI)


要让 OpenWrt 充分利用 zram 交换区，你需要调整 Linux 内核的 Swappiness 参数。目前交换区空闲 99% 说明内核倾向于保留物理内存而尽量不使用交换空间。
## 1. 修改 Swappiness 参数
vm.swappiness 的值决定了系统将内存数据置换到 Swap 的积极程度，范围是 0 到 100。默认通常为 60，对于内存极其有限的路由器，可以提高到 100 以强制利用 zram。

* 临时生效（重启失效）：

sysctl -w vm.swappiness=100

* 永久生效：
在 /etc/sysctl.conf 文件中添加或修改以下行：

vm.swappiness=100

然后执行 sysctl -p 应用配置。

## 2. 检查 zram 优先级 (Priority)
如果系统同时存在 zram 和物理磁盘上的 Swap 分区，必须确保 zram 的优先级最高（正数越大优先级越高），否则系统可能会优先写入慢速的磁盘空间。

* 查看当前状态：cat /proc/swaps
* 如果在 /etc/config/zram 中配置，可以检查 priority 设置。通常 zram 脚本会自动设置一个高优先级（如 100）。

## 3. 其他进阶优化

* 调整压缩算法：如果 CPU 性能尚可，可以使用 zstd 或 lz4 算法。在 /etc/config/zram 中修改（需安装对应内核模块 kmod-lib-zstd 等）：

option comp_algorithm 'zstd'

* 控制内存回收压力：
调整 vm.vfs_cache_pressure。默认值 100，增加到 150 或更高 可以让内核更积极地回收目录项和索引节点缓存，从而腾出物理内存用于运行程序，并将不常用的匿名内存压入 zram。

## 4. 验证效果
调整后，你可以运行一些占用内存较多的程序，或者通过以下命令观察 zram 实际压缩比例：

zramctl

(注：如果命令不存在，可通过 opkg install zram-tools 安装)
虽然 100 会最大化利用 Swap，但过度频繁的压缩/解压会增加 CPU 负载。如果感觉系统响应变慢，可以尝试降回 80-90 寻找平衡点。

