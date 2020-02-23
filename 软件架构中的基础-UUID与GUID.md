# 软件架构中的基础-UUID与GUID

* [软件架构中的基础-UUID与GUID](#软件架构中的基础-uuid与guid)
	* [UUID与GUID介绍](#uuid与guid介绍)
		* [UUID](#uuid)
			* [应用-Linux下UUID的使用举例](#应用-linux下uuid的使用举例)
		* [GUID](#guid)
			* [应用-磁盘分区表方案](#应用-磁盘分区表方案)
			* [GUID生成方式举例](#guid生成方式举例)
	* [参考链接](#参考链接)

## UUID与GUID介绍

### UUID

UUID含义是通用唯一识别码 (Universally Unique Identifier)，这是一个软件建构的标准，也是被开源软件基金会 (Open Software Foundation, OSF) 的组织应用在分布式计算环境 (Distributed Computing Environment, DCE) 领域的一部分。

>  A UUID is 128 bits long, and can guarantee
   uniqueness across space and time.  UUIDs were originally used in the
   Apollo Network Computing System and later in the Open Software
   Foundation's (OSF) Distributed Computing Environment (DCE), and then
   in Microsoft Windows platforms.    
——《A Universally Unique IDentifier (UUID) URN Namespace》

1. 作用
UUID 的目的，是让分布式系统中的所有元素，都能有唯一的辨识资讯，而不需要透过中央控制端来做辨识资讯的指定。如此一来，每个人都可以建立不与其它人冲突的 UUID。在这样的情况下，就不需考虑数据库建立时的名称重复问题。目前重要的应用有 Linux ext2/ext3 档案系统、LUKS 加密分割区、GNOME、KDE、Mac OS X 等等。

2. 组成
UUID是指在一台机器上生成的数字，它保证对在同一时空中的所有机器都是唯一的。通常平台会提供生成的API。按照开放软件基金会(OSF)制定的标准计算，用到了以太网卡地址、纳秒级时间、芯片ID码和许多可能的数字
UUID由以下几部分的组合：
（1）当前日期和时间，UUID的第一个部分与时间有关，如果你在生成一个UUID之后，过几秒又生成一个UUID，则第一个部分不同，其余相同。
（2）时钟序列。
（3）全局唯一的IEEE机器识别号，如果有网卡，从网卡MAC地址获得，没有网卡以其他方式获得。
UUID的唯一缺陷在于生成的结果串会比较长。**标准的UUID格式为：xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxx (8-4-4-4-12)。**

3. 版本变化
* UUID Version 1：基于时间的UUID
基于时间的UUID通过计算当前时间戳、随机数和机器MAC地址得到。由于在算法中使用了MAC地址，这个版本的UUID可以保证在全球范围的唯一性。但与此同时，使用MAC地址会带来安全性问题，这就是这个版本UUID受到批评的地方。如果应用只是在局域网中使用，也可以使用退化的算法，以IP地址来代替MAC地址－－Java的UUID往往是这样实现的（当然也考虑了获取MAC的难度）。

* UUID Version 2：DCE安全的UUID
DCE（Distributed Computing Environment）安全的UUID和基于时间的UUID算法相同，但会把时间戳的前4位置换为POSIX的UID或GID。这个版本的UUID在实际中较少用到。

* UUID Version 3：基于名字的UUID（MD5）
基于名字的UUID通过计算名字和名字空间的MD5散列值得到。这个版本的UUID保证了：相同名字空间中不同名字生成的UUID的唯一性；不同名字空间中的UUID的唯一性；相同名字空间中相同名字的UUID重复生成是相同的。

* UUID Version 4：随机UUID
根据随机数，或者伪随机数生成UUID。这种UUID产生重复的概率是可以计算出来的，但随机的东西就像是买彩票：你指望它发财是不可能的，但狗屎运通常会在不经意中到来。

* UUID Version 5：基于名字的UUID（SHA1）
和版本3的UUID算法类似，只是散列值计算使用SHA1（Secure Hash Algorithm 1）算法。

从UUID的不同版本可以看出，Version 1/2适合应用于分布式计算环境下，具有高度的唯一性；Version 3/5适合于一定范围内名字唯一，且需要或可能会重复生成UUID的环境下。

#### 应用-Linux下UUID的使用举例

Linux磁盘分区UUID的获取方法

``` 
[san@localhost ~]$ ls -l /dev/disk/by-uuid/
total 0
lrwxrwxrwx 1 root root 10 2010-01-18 02:18 0733f5c1-cb85-4f98-9d4f-122cfcee9806 -> ../../sdc1
lrwxrwxrwx 1 root root 10 2010-01-18 01:13 3754-1BDB -> ../../sda5
lrwxrwxrwx 1 root root 10 2010-01-18 01:13 41a18221-6b1f-4ca2-9bc3-dc353c87d932 -> ../../sda9
lrwxrwxrwx 1 root root 10 2010-01-18 01:13 57183ff9-d4a5-4623-a47f-f8f17339be03 -> ../../sda7
lrwxrwxrwx 1 root root 10 2010-01-18 01:13 6bdf487f-cad7-4197-b0d9-4ddc6df1de2d -> ../../sda8
lrwxrwxrwx 1 root root 10 2010-01-18 01:13 ae6dcc02-3f7f-47cc-8a6e-e29218b4d345 -> ../../sda6
lrwxrwxrwx 1 root root 10 2010-01-18 01:13 CC47-2A04 -> ../../sda1
lrwxrwxrwx 1 root root 10 2010-01-18 02:18 d2154d3e-3006-4a05-a134-f721145f1670 -> ../../sdc2
lrwxrwxrwx 1 root root 10 2010-01-18 02:18 df974270-dbba-4f87-8121-427636dab396 -> ../../sdc3
lrwxrwxrwx 1 root root 10 2010-01-18 01:52 f535fef8-f392-4c84-8e7a-85915d9179fb -> ../../sdb1

[san@localhost ~]$ blkid /dev/sdb1
/dev/sdb1: LABEL="SAN " UUID="f535fef8-f392-4c84-8e7a-85915d9179fb " TYPE="ext3 "

[san@localhost ~]$ blkid /dev/sda6
/dev/sda6: LABEL="/" UUID="ae6dcc02-3f7f-47cc-8a6e-e29218b4d345 " TYPE="ext3" SEC_TYPE="ext2 "

```

Linux UUID的作用及意义：
* 它是真正的唯一标识符
UUID为系统中的存储设备提供唯一的标识字符串，不管这个设备是什么类型的。如果你在系统中添加了新的存储设备如硬盘，很可能会造成一些麻烦，比如说启动的时候因为找不到设备而失败，而使用UUID则不会有这样的问题。
* 设备名并非总是不变的
自动分配的设备名称并非总是一致的，它们依赖于启动时内核加载模块的顺序。如果你在插入了USB盘时启动了系统，而下次启动时又把它拔掉了，就有可能导致设备名分配不一致。使用UUID对于挂载移动设备也非常有好处──例如我有一个24合一的读卡器，它支持各种各样的卡，而使用UUID总可以使同一块卡挂载在同一个地方。
* ubuntu中的许多关键功能现在开始依赖于UUID
例如grub──系统引导程序，现在可以识别UUID，打开你的/boot/grub/menu.lst，你可以看到类似如下的语句：
```
title Ubuntu hardy (development branch), kernel 2.6.24-16-generic
root (hd2,0)
kernel /boot/vmlinuz-2.6.24-16-generic root=UUID=c73a37c8-ef7f-40e4-b9de-8b2f81038441 ro quiet splash
initrd /boot/initrd.img-2.6.24-16-generic
quiet
```

### GUID

全局唯一标识符(GUID，Globally Unique Identifier)是一种由算法生成的二进制长度为128位的数字标识符。其意义同UUID，为了区分于UUID，这里将GUID专指微软对UUID标准的实现。
在 Windows 平台上，GUID 广泛应用于微软的产品中，用于标识如注册表项、类及接口标识、数据库、系统目录等对象。

#### 应用-磁盘分区表方案

先说说广泛使用的磁盘分区表方案。传统的分区方案(称为MBR分区方案)是将分区信息保存到磁盘的第一个扇区(MBR扇区)中的64个字节中，每个分区项占用16个字节，这16个字节中存有活动状态标志、文件系统标识、起止柱面号、磁头号、扇区号、隐含扇区数目(4个字节)、分区总扇区数目(4个字节)等内容。由于MBR扇区只有64个字节用于分区表，所以只能记录4个分区的信息。这就是硬盘主分区数目不能超过4个的原因。后来为了支持更多的分区，引入了扩展分区及逻辑分区的概念。但每个分区项仍用16个字节存储。

MBR分区方案不是用得好好的吗？为什么要提出新的方案呢？那就让我们看看MBR分区方案有什么问题。前面已经提到了主分区数目不能超过4个的限制，这是其一，很多时候，4个主分区并不能满足需要。另外最关键的是MBR分区方案无法支持超过2TB容量的磁盘。因为这一方案用4个字节存储分区的总扇区数，最大能表示2的32次方的扇区个数，按每扇区512字节计算，每个分区最大不能超过2TB。磁盘容量超过2TB以后，分区的起始位置也就无法表示了。在硬盘容量的突飞猛进，2TB的限制已经被突破。由此可见，MBR分区方案已经无法满足需要了。下面介绍GUID分区表方案。

GUID分区表(简称GPT。使用GUID分区表的磁盘称为GPT磁盘)是源自EFI标准的一种较新的磁盘分区表结构的标准。与普遍使用的主引导记录(MBR)分区方案相比，GPT提供了更加灵活的磁盘分区机制。它具有如下优点：
1. 支持2TB以上的大硬盘。
2. 每个磁盘的分区个数几乎没有限制。为什么说“几乎”呢？是因为Windows系统最多只允许划分128个分区。不过也完全够用了。
3. 分区大小几乎没有限制。又是一个“几乎”。因为它用64位的二进制数表示扇区大小，即：
2^64 =18,446,744,073,709,551,616字节
=18,014,398,509,481,984 KB
=17,592,186,044,416 MB
=17,179,869,184 GB
=16,777,216 TB
=16,384 PB
=16 EB
（当今世界所有的磁盘容量加在一起也没有超过16EB）
夸张一点说，一个64位二进制数能代表的分区大小已经是个“天文数字”了，若干年内你都无法见到这样大小的硬盘，更不用说分区了。
4. 分区表自带备份。在磁盘的首尾部分分别保存了一份相同的分区表。其中一份被破坏后，可以通过另一份恢复。
5. 每个分区可以有一个名称(不同于卷标)。

既然GUID分区方案具有如此多的优点，在分区时是不是可以全部采用这种方案呢？不是的。并不是所有的Windows系统都支持这种分区方案。 请看下表：

| Windows种类                       | 能否读写GPT磁盘                      | 能否从GPT磁盘启动                      |
| --------------------------------- | ------------------------------------ | -------------------------------------- |
| 32位 Windows XP                   | 不能。只能看到一个Protective MBR分区 | 不支持                                 |
| Windows 2000/NT/9x                | 不能。只能看到一个Protective MBR分区 | 不支持                                 |
| 64位 Windows XP                   | 能                                   | 只有基于Itanium的系统才能从GPT磁盘启动 |
| Windows Server 2003 SP1及以上版本 | 能                                   | 只有基于Itanium的系统才能从GPT磁盘启动 |
| Windows Vista                     | 能                                   | 只有基于 EFI 的系统支持从GPT磁盘启动   |
| Windows Server 2008               | 能                                   | 只有基于 EFI 的系统支持从GPT磁盘启动   |
| Windows 7                         | 能                                   | 只有基于 EFI 的系统支持从GPT磁盘启动   |
| Windows 8/8.1                     | 能                                   | 只有基于 EFI 的系统支持从GPT磁盘启动   |

是不是很失望？多数的个人电脑系统还无法完美支持GPT磁盘。但是这并不意味着我们不需要了解GUID分区方案。别忘了，硬件的发展速度总是令人吃惊的。1.5TB的硬盘已经大量上市，2TB以上容量的硬盘很快就会普及，基于EFI的主板也正在销售。GUID分区方案终将成为主流 。	

做为一款分区软件，DiskGenius从3.1版本开始支持GUID分区表。这是国内第一款支持GUID分区表的分区软件。DiskGenius提供了GUID分区的建立、删除、格式化、已丢失分区恢复、文件恢复、分区表备份、GUID分区表格式与MBR分区表格式之间的相互转换(无损转换)等功能。

[See also: GUID Partition Table - Wikipedia.](http://en.wikipedia.org/wiki/GUID_Partition_Table)

#### GUID生成方式举例

1. 调用Win32API - CoCreateGuid函数
```
#include <objbase.h>  
#include <stdio.h>  
  
#define GUID_LEN 64  
  
int main(int argc, char* argv[])  
{  
    char buffer[GUID_LEN] = { 0 };  
    GUID guid;  
  
    if ( CoCreateGuid(&guid) == S_OK)  
    {  
        fprintf(stderr, "create guid error\n");  
        return -1;  
    }  
    _snprintf(buffer, sizeof(buffer),   
        "%08X-%04X-%04x-%02X%02X-%02X%02X%02X%02X%02X%02X",   
        guid.Data1, guid.Data2, guid.Data3,   
        guid.Data4[0], guid.Data4[1], guid.Data4[2],   
        guid.Data4[3], guid.Data4[4], guid.Data4[5],   
        guid.Data4[6], guid.Data4[7]);  
    printf("guid: %s\n", buffer);  
  
    return 0;  

}
```

2. 调用Boost库
```
#include <boost/uuid/uuid.hpp>
#include <boost/uuid/uuid_generators.hpp>
#include <boost/uuid/uuid_io.hpp>

boost::uuids::uuid uid = boost::uuids::random_generator()();
 const string uid_str = boost::uuids::to_string(uid);
 cout << uid_str << endl;
```

[See also: crossguid, Lightweight cross platform C++ GUID/UUID library.](https://github.com/graeme-hill/crossguid)

## 参考链接
[1. A Universally Unique IDentifier (UUID) URN Namespace - RFC；](https://www.ietf.org/rfc/rfc4122.txt)
[2. UUID GUID - Cnblogs baihuahua；](https://www.cnblogs.com/baiyw/p/3449173.html)
[3. Create GUIDs online；](https://www.guidgen.com/)
[4. UUID Online - 在线生成UUID；](http://www.uuid.online/)
[5. 一个UUID生成算法的C语言实现——WIN32版本；](http://www.cppblog.com/jerryma/archive/2011/04/29/145330.html)
[6. [译] 把 UUID 或者 GUID 作为主键？你得小心啦！- 掘金翻译计划；](https://blog.csdn.net/weixin_34101784/article/details/87964086)
[7. GUID （磁盘分区表方案）- 百度百科；](https://baike.baidu.com/item/GUID/15412374?fr=aladdin)
[8. Linux磁盘分区UUID的获取及其UUID的作用 - CSDN sanlinux；](https://blog.csdn.net/sanlinux/article/details/5203923)
[9. c++ 生成GUID的几种方法 - cnblogs 丢了木剑的温华；](https://www.cnblogs.com/0523jy/p/11399578.html)
[10. C++生成GUID的两种方法 - CSDN semicloud；](https://blog.csdn.net/semicloud/article/details/89332018)