# 查找Linux中最大文件的方法

## 查看当前系统存储情况

`df -lh` 查看磁盘信息,可以从整个硬盘层面来看到当前硬盘存储是否已经达到

## 遍历所有文件

初学linux时候，我们就知道**根目录**和**家目录**来个相对重要的目录。根目录是所有目录的起点，我们遍历文件从**根目录**开始。

### 操作流程

1. `cd /`
2. 在根目录执行`du -h --max-depth=1`,能够获取到根目录下哪个**文件或者目录**最大
3. 找到最大目录后，重新执行2来获取最大文件
4. 重复执行步骤2和3即可获取到设备中最大的文件

### ls命令排序

在查找到某个文件夹占用存储较多以后，到相应的目录下查看具体哪个文件占用了较多的储存。

`ls –lhS` 将文件以从大到小顺序展现

### 清空文件

清空文件有两种方法:

- 简单粗暴方法，直接`rm -rf 文件名或者文件全路径`,将整个文件进行删除
- 对当前已经在使用文件的程序毫无影响的做法是，`truncate -s0 文件名或者文件全路径`,清空文件的所有内容，文件大小变为0

## 总结
1. 先行通过`df -h`查看整体硬盘中是否存在已占用很高的分区
2. 在根目录(/)中使用`du -h --max-depth=1`,如果没有du命令，可以替换为其他命令
3. 确定大存储位置后，进入相应位置后，重复执行步骤2，直到找到相应大文件然后进行相应的处理。