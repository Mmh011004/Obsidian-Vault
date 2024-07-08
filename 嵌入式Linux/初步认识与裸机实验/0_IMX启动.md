# 1.启动设备选择

# 2.IMX 启动方式
在烧写文件的时候，是将bin 文件烧写为load.imx 文件，也就是在.bin 文件前面加上了头部信息。这样的 load.imx 文件是满足 I.MX6U 需要最终的可烧写文件的，最终的可烧写文件的组成如下：
1. Image vector table，简称IVT，IVT 包含了一系列的地址信息，这些信息在ROM 中按照固定的地址存放着。
2. Boot data, 启动数据，包含了镜像要拷贝到哪个地址，拷贝大小为多少等信息
3. Device configuration data，简称 DCD，设备配置信息，重点是 DDR3 的初始化配置。
4. 用户代码可执行文件，比如 led.bin。
## 2_1 Boot Rom 做的事情
1. 设置内核时钟为 396M
2. 使能MMU 和caches，使能L1caches 和L2 caches 和MMU，目的是为了加速启动
3. 从BOOT_CFG 设置的外置存储种，读取镜像image，然后做相应的处理

## 2_2 IVT 表和Boot Data 数据
在
