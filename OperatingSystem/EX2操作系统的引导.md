# 操作系统的引导

## 实验的内容
### 改写 bootsect.s
1. 这一部分完成的主要内容是显示操作系统的名字
2. 接着修改如何从 bootsect.s 引导到 setup.s 中去

### 改写 setup.s
1. 从 bootsect.s 到达 setup.s 之后,首先在屏幕输出一条"Now we are in SETUP"
2. setup.s 获得一些基本的硬件参数(内存参数,显卡参数,硬盘参数),并且打印到屏幕上
3. 先不加载 linux 内核