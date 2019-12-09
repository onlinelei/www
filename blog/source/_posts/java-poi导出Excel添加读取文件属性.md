---
title: poi导出Excel添加/读取文件属性
date: 2016-09-20 17:59:44
tags: java
---

poi导出Excel添加/读取文件属性
<!--more-->

```java
 	/**
	 * 
	 * 
	 * 功能描述：excel添加备注信息
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
	public static void set_ExcelProperties(HSSFWorkbook workbook, Map<String, ?> parameter) {
		workbook.createInformationProperties();
		SummaryInformation suminfoInformation = workbook.getSummaryInformation();
		DocumentSummaryInformation docmentIfo = workbook.getDocumentSummaryInformation();
		// 添加备注信息
		suminfoInformation.setComments("此文档添加备注信息");
		suminfoInformation.setAuthor("sunlands");
		// 添加自定义属性
		CustomProperties customProperties = new CustomProperties();
		Set<String> parSet = parameter.keySet();
		if (parameter != null) {
			for (String setString : parSet) {
				if (!StringUtils.isBlank(setString) && null != parameter.get(setString)) {
					customProperties.put(setString, parameter.get(setString).toString());
				}
			}
		}
		// 将信息写入workbook
		docmentIfo.setCustomProperties(customProperties);
	}
```

java 获取属性  

```java
	/**
	 *
	 * 功能描述：获取上传报表属性中存入的属性
	 *
	 * @param key 属性key值
	 *
	 * @return key值对应的属性值
	 *
	 * @author 钱磊
	 *
	 * @since 2016年9月29日
	 *
	 * @update:[变更日期YYYY-MM-DD][更改人姓名][变更描述]
	 */
	public String getProperties(String key){
		String uuid = "";
		if (!StringUtils.isBlank(key)) {
			DocumentSummaryInformation docuInfo = wb.getDocumentSummaryInformation();
			CustomProperties customProperties = docuInfo.getCustomProperties();
			Object temp = customProperties.get(key);
			if (null != temp) {
				uuid = temp.toString();
			}
		}
		return uuid;
	}
```
