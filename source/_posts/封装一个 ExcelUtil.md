---
title: 封装一个 ExcelUtil
date: 2020-07-26 08:45:11
tags: 代码片段
categories: code
---

```java
package com.hdvon.iomp.visual.utils;
import com.google.common.collect.Lists;
import org.apache.poi.ss.usermodel.Row;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.xssf.streaming.SXSSFWorkbook;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.BeanWrapperImpl;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.List;
import java.util.Map;

/**
 * @author lyle 2019/5/13 14:14.
 */
public class ExcelUtils {
    private static final Logger logger = LoggerFactory.getLogger(ExcelUtils.class);
    private ExcelUtils() {}

    /**
     * @param header   excel 的第一行
     * @param dataList 数据集
     * @param fileName 文件名
     * @param response 响应
     * @description 导出 excel，默认只有一个 sheet
     **/
    public static <T> void export(Map<String, String> header, List<T> dataList, String fileName, HttpServletResponse response) throws IOException {
        try (Workbook workbook = new SXSSFWorkbook()) {
            Sheet sheet = workbook.createSheet("sheet1");
            Row headerRow = sheet.createRow(0);
            List<String> values = Lists.newArrayList(header.values());
            for (int i = 0; i < values.size(); i++) {
                headerRow.createCell(i).setCellValue(values.get(i));
            }

            List<String> keys = Lists.newArrayList(header.keySet());
            int dataStart = 1;
            for (int i = 0; i < dataList.size(); i++) {
                Row row = sheet.createRow(dataStart++);
                T t = dataList.get(i);
                BeanWrapperImpl beanWrapper = new BeanWrapperImpl(t);
                for (int k = 0; k < keys.size(); k++) {
                    Object propertyValue = beanWrapper.getPropertyValue(keys.get(k));
                    row.createCell(k).setCellValue(propertyValue == null ? "" : propertyValue.toString());
                }
            }

            response.reset();
            response.setHeader("Content-Disposition", "attachment; filename=\"" + fileName + "\"");
            response.setContentType("application/octet-stream; charset=utf-8");
            workbook.write(response.getOutputStream());
        } catch (IOException ex) {
            logger.error("导出execl异常，error：{}", ex.getMessage());
            throw ex;
        }
    }
}

```