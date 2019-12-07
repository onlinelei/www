---
title: Poi导出Excel插入文本框（TextBox）
date: 2016-09-22 17:59:44
tags: java
---

项目中需要导出Excel插入TextBox实现如下效果。
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/2007394-3c6faea7ae431e4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<!--more-->

	/**
	 * 
	 * 
	 * 功能描述：excel添加textBox
	 * 
	 * @param workbook
	 *            需要添加备注信息的Excel
	 * 
	 * @param parameter
	 *            备注信息
	 * 
	 * @author qianlei
	 * 
	 * @since 20160926
	 * 
	 * @update:[变更日期YYYY-MM-DD][更改人姓名][变更描述]
	 */
	public static void setExcelTextBoxAtFirstSheet(HSSFWorkbook workbook,String text) {
		HSSFSheet createSheet = workbook.createSheet("文本框");
		// 获得drawingPatriarch
		HSSFPatriarch pat = createSheet.createDrawingPatriarch();
		// 创建锚点
		HSSFClientAnchor anchor = new HSSFClientAnchor(1, 1, 20, 20, (short) 1,1, (short) 20, 20);
		HSSFTextbox createTextbox = pat.createTextbox(anchor);
		createTextbox.setString(new HSSFRichTextString(text));
	}
