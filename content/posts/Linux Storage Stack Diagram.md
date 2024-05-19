+++
author = "Hwa"
title = "Linux Storage Stack Diagram"
tags = [
    "Linux",
    "OS",
    "IO"
]
date = 2022-10-30
summary = "Linux存储栈图解"
+++

Linux存储栈：分为文件系统层，块层和设备层。

![detailed](https://gallery-1259614029.cos.ap-chengdu.myqcloud.com/img/detailed.png)

对以上进行抽象后：

![abstract](https://gallery-1259614029.cos.ap-chengdu.myqcloud.com/img/abstract.png)

BufferIO:

![buffer_IO](https://gallery-1259614029.cos.ap-chengdu.myqcloud.com/img/buffer_IO.png)

经典读写操作IO：从磁盘到网络

![disk_to_network](https://gallery-1259614029.cos.ap-chengdu.myqcloud.com/img/disk_to_network.png)

网络与磁盘读写IO：

![network_and_disk](https://gallery-1259614029.cos.ap-chengdu.myqcloud.com/img/network_and_disk.png)

![diskIO](https://gallery-1259614029.cos.ap-chengdu.myqcloud.com/img/diskIO.png)
