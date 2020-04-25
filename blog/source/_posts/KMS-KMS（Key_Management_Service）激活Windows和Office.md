---
title: KMS（Key Management Service）激活Windows和Office
date: 2017-11-19 19:20:54
tags: kms
---



![](http://oxylkiuu1.bkt.clouddn.com/kms.jpg)
<!--more-->
1.windows
以管理员身份打开命令提示符,然执行下列命令：

	cd /d "%SystemRoot%\system32"
	slmgr /skms kms.suroot.win
	slmgr /ato
	slmgr /xpr
2.office
以管理员身份打开命令提示符,进入软件安装目录,然后执行下列命令:
这里以Office 2013为例:
进入32位版本安装目录：

	cd /d "%ProgramFiles(x86)%\Microsoft Office\Office15"
进入64位版本安装目录:

	cd /d "%ProgramFiles%\Microsoft Office\Office15"
然后执行下列命令:

	cscript ospp.vbs /sethst:kms.suroot.win
	cscript ospp.vbs /act
	cscript ospp.vbs /dstatus
注意事项
KMS方式激活的有效期只有180天.
每隔一段时间系统会自动KMS服务器请求续期。
KMS方式只能激活批量授权版本的windows和Office也就是VOL版本

在使用上，零售版和批量授权版并没有区别，只是授权方式方面的区别，相对而言，VOL 版的更容易激活一些，其他并没有什么区别了。  

Windows10下载地址
-------------------------
	文件名：en_windows_10_multiedition_vl_version_1709_updated_sept_2017_x64_dvd_100090741.iso
	SHA1：1BBF886697A485C18D70AD294A09C08CB3C064AC
	文件大小：4.25GB
	发布时间：2017-10-18
	下载地址：ed2k://|file|en_windows_10_multi-edition_vl_version_1709_updated_sept_2017_x64_dvd_100090741.iso|4558182400|35385C7181C441E9A66823657516633E|/

	文件名:en_windows_10_multi-edition_vl_version_1709_updated_sept_2017_x86_dvd_100090759.iso
	SHA1:8C274CE27B49D43216DFEF115B811936880E6E06
	文件大小:3.11GB
	发布时间:2017-10-18
	下载地址：ed2k://|file|en_windows_10_multi-edition_vl_version_1709_updated_sept_2017_x86_dvd_100090759.iso|3341574144|EC97B9F7E7CC924A17CC7FDB02DF721E|/  

Office下载地址
-------------------------

有需要的可以在下面下载：（以下均是 VL 版）

	版本：Office 2016 Pro Plus  64 位
	
	文件名：SW_DVD5_Office_Professional_Plus_2016_64Bit_ChnSimp_MLF_X20-42426.ISO
	
	文件大小： 1123452928 字节
	
	MD5: 60DC8B1892F611E41140DD3631F39793
	
	SHA1: AEB58DE1BC97685F8BC6BFB0A614A8EF6903E318
	
	CRC32: 8D8AC6D1
	
	下载地址：
	ed2k://|file|SW_DVD5_Office_Professional_Plus_2016_64Bit_ChnSimp_MLF_X20-42426.ISO|1123452928|31087A00FF67D4F5B4CBF4AA07C3433B|/

	版本：Office 2016 Pro Plus 32 位
	
	文件名：SW_DVD5_Office_Professional_Plus_2016_W32_ChnSimp_MLF_X20-41351.ISO
	
	文件大小：986441728 字节
	
	MD5: 49D97BD1B4DFEAAA6B45E3DD3803DAC1
	
	SHA1: 0218F50774AAB63AF7755B0986CDB9972B853E44
	
	CRC32: FF96B0B5

	下载地址：ed2k://|file|SW_DVD5_Office_Professional_Plus_2016_W32_ChnSimp_MLF_X20-41351.ISO|986441728|2DE74581C10096137481873B3AD57D43|/

	版本：Office 2016 Project 专业版 64 位版
	
	文件名：SW_DVD5_Project_Pro_2016_64Bit_ChnSimp_MLF_X20-42676.ISO
	
	文件大小：647157760 字节
	
	MD5: B872E55B8F4A8791D65BCF1DB46D1DCB
	
	SHA1: 3C180FDAF91DBD0CB767BD040B42B0599FC53438
	
	CRC32: 6AB6A570

	下载地址：ed2k://|file|SW_DVD5_Project_Pro_2016_64Bit_ChnSimp_MLF_X20-42676.ISO|647157760|0BBBF20CA3A5F61A819586ADCE6E4DCB|/

	版本：Office 2016 Project 专业版 32 位版
	
	文件名：SW_DVD5_Visio_Pro_2016_64Bit_ChnSimp_MLF_X20-42759.ISO
	
	文件大小：714913792 字节
	
	MD5: 93BEB874F5A5870D5854519856047103
	
	SHA1: 71E082174812F748AB1A70CA33E6004E1E1AACA8
	
	CRC32: F813794B

	下载地址：ed2k://|file|SW_DVD5_Project_Pro_2016_W32_ChnSimp_MLF_X20-41511.ISO|555210752|CA3BD5F8C7B3E263105B041DDD4104AB|/

	版本：Office 2016 Visio 专业版 64 位版
	
	文件名：SW_DVD5_Visio_Pro_2016_64Bit_ChnSimp_MLF_X20-42759.ISO
	
	文件大小：714913792 字节
	
	MD5: 93BEB874F5A5870D5854519856047103
	
	SHA1: 71E082174812F748AB1A70CA33E6004E1E1AACA8
	
	CRC32: F813794B

	下载地址：ed2k://|file|SW_DVD5_Visio_Pro_2016_64Bit_ChnSimp_MLF_X20-42759.ISO|714913792|FC930AB97B366B3595FC2F28ABAC2A6F|/

	版本：Office 2016 Visio 专业版 32 位版
	
	文件名：SW_DVD5_Visio_Pro_2016_W32_ChnSimp_MLF_X20-41580.ISO
	
	文件大小：609447936 字节
	
	MD5: 96E008B110F308F1E424D11964D82CE0
	
	SHA1: 780046411EB18874AA2DA7E4A11322557EB00D92
	
	CRC32: 42E1653D

	下载地址：ed2k://|file|SW_DVD5_Visio_Pro_2016_W32_ChnSimp_MLF_X20-41580.ISO|609447936|91EB248558F236AA66D234EA03FAD9A9|/