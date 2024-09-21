# 何为Uboot
1. uboot 是一个裸机程序
2. uboot 就是一个bootloader, 作用于启动Linux 或者其他的系统。Uboot 最主要的工作就是初始化DDR。这是因为Linux 是运行在DDR 里面的。一般Linux 镜像zImage (uImage) 和设备树 (.dtb) 存放到SD、EMMC、NAND、SPI FLASH 等等外置存储区域
我们需要将Linux 镜像从外置的flash 拷贝到DDR 中，再去启动。
* Uboot 的主要目的就是为系统启动做准备。
* Uboot 不仅仅能启动Linux，也可以启动其他系统，比如vxworks。
* Linux 不仅仅能通过uboot 启动。
* Uboot 是个通用的bootloader，支持多种架构。
