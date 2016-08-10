---
layout: post
title:  "并发操作读写Excel"
date:   2016-08-10 09:17:52
categories: excel concurrent
published: true
comments: true
thread: 20160810091855555
---

程序猿们为了配合市场工作临时上了一个活动，算是一个小功能，需求如下：
1. 有三个页面，参与活动用户信息录入页面；活动页面，有问卷调查；活动参与结果页面；
2. 要求活动信息留存，用户提交后，立即邮件提醒运营。

本来对应方案：
  信息读取录入到excel文件中。

## 问题：
  由于对并发读写excel没有预判和多线程知识短板，造成该方案堵塞, 话不多说上代码。
```java
// 写文件操作
public ResponseEntity saveInfo(@RequestParam(value = "mobileNum") String mobileNum,
                                  @RequestParam(value = "url") String url,
                                  @RequestParam(value = "answer") String answer) {
       ActivityResponseDTO response = new ActivityResponseDTO();
       File file = getActivityXls();
       FileInputStream fis = null;
       FileOutputStream fos = null;
       try {
           fis = new FileInputStream(file);
           POIFSFileSystem ps = new POIFSFileSystem(fis);
           HSSFWorkbook wb = new HSSFWorkbook(ps);
           HSSFSheet sheet = wb.getSheetAt(0);

           fos = new FileOutputStream(file);
           HSSFRow row = sheet.createRow(sheet.getLastRowNum() + 1);

           row.createCell(0).setCellValue(sheet.getLastRowNum() + 1);
           row.createCell(1).setCellValue(mobileNum);
           row.createCell(2).setCellValue(url);

           String code = NumberUtil.format(sheet.getLastRowNum() + 1, 3);
           row.createCell(3).setCellValue(code);

           response.setCode(code);
           String[] answers = answer.split(",");
           for (int i=4; i< 11; i++) {
               row.createCell(i).setCellValue(answers[i-4]);
           }
           fos.flush();
           wb.write(fos);
       } catch (Exception e) {
           LOGGER.error("", e);
       } finally {
           try {
               if (null != fos) {
                   fos.close();
               }
               if (null != fis) {
                   fis.close();
               }
           } catch (IOException e) {
               LOGGER.error("", e);
           }
       }
       LOGGER.info("===========================save response: "+response.toString());
       noticev4Biz.sendCardMail(mobileNum, url);
       return createResponse(response);
   }
```

## 解决
从代码可以看出，读逻辑是有同步的，但也只是对单用户操作的同步，写操作没有同步保护。
谁是竞争的资源？是excel文件。所以要保证所有的线程对该文件实现读写同步。可以实现一个该文件的饱汉单例，对该单例进行写同步。
```Java
// 写文件操作(同步之后的代码)
public ResponseEntity saveInfo(@RequestParam(value = "mobileNum") String mobileNum,
                                  @RequestParam(value = "url") String url,
                                  @RequestParam(value = "answer") String answer) {
       ActivityResponseDTO response = new ActivityResponseDTO();
       response.setCode(ExcelSingleton.write2File(mobileNum, url, answer));
       LOGGER.info("===========================save response: "+response.toString());
       noticev4Biz.sendCardMail(mobileNum, url);
       return createResponse(response);
}


/**
 * 对固定Excel文件写操作同步
 */
public class ExcelSingleton {

    private static final String EXCEL_FILE_PATH = "/tmp/activity/activity.xls";
    private static final Logger LOGGER = LoggerFactory.getLogger(ExcelSingleton.class);
    private static File excel = new File(EXCEL_FILE_PATH);

    public static synchronized String write2File(String mobileNum, String url, String answer) {
        LOGGER.info("当前线程{}, 开始写入.", Thread.currentThread().getName());

        try (
                FileInputStream fis = new FileInputStream(excel);
        ) {
            POIFSFileSystem ps = new POIFSFileSystem(fis);
            HSSFWorkbook wb = new HSSFWorkbook(ps);
            HSSFSheet sheet = wb.getSheetAt(0);

            FileOutputStream fos = new FileOutputStream(excel);
            HSSFRow row = sheet.createRow(sheet.getLastRowNum() + 1);

            row.createCell(0).setCellValue(sheet.getLastRowNum() + 1);
            row.createCell(1).setCellValue(mobileNum);
            row.createCell(2).setCellValue(url);

            String code = NumberUtil.format(sheet.getLastRowNum() + 1, 3);
            row.createCell(3).setCellValue(code);

            String[] answers = answer.split(",");
            for (int i = 4; i < 11; i++) {
                row.createCell(i).setCellValue(answers[i - 4]);
            }

            wb.write(fos);
            fos.close();
            fis.close();
            LOGGER.info("当前线程{}, 写入完毕.", Thread.currentThread().getName());
            return code;
        } catch (Exception e) {
            LOGGER.error("写入文件失败:", e);
            LOGGER.info("当前线程{}, 写入完毕.", Thread.currentThread().getName());
            return null;
        }
    }

    /**
     * 没有该Excel文件时，先运行该方法
     */
    public static File getActivityXls() {
        File targetFile = new File(EXCEL_FILE_PATH);

        if (targetFile.exists()) {
            return targetFile;
        }

        HSSFWorkbook book = new HSSFWorkbook();
        HSSFSheet sheet = book.createSheet("清凉一夏");
        sheet.setDefaultColumnWidth(50);

        HSSFRow firstRow = sheet.createRow(0);
        CellStyle titleStyle = book.createCellStyle();
        titleStyle.setFillForegroundColor(IndexedColors.YELLOW.getIndex());
        titleStyle.setFillPattern(CellStyle.BORDER_THICK);
        titleStyle.setBorderBottom(HSSFCellStyle.BORDER_THICK);
        titleStyle.setBorderTop(HSSFCellStyle.BORDER_THICK);
        titleStyle.setBorderLeft(HSSFCellStyle.BORDER_THICK);
        titleStyle.setBorderRight(HSSFCellStyle.BORDER_THICK);
        HSSFFont titleFont = book.createFont();
        titleFont.setColor(IndexedColors.BLACK.getIndex());
        titleFont.setFontHeightInPoints((short) 16);
        titleFont.setBoldweight(HSSFFont.BOLDWEIGHT_NORMAL);
        titleStyle.setFont(titleFont);
        firstRow.setHeightInPoints(3 * 20);

        HSSFCell cell = firstRow.createCell(0);
        cell.setCellStyle(titleStyle);
        cell.setCellValue("序号");

        cell = firstRow.createCell(1);
        cell.setCellStyle(titleStyle);
        cell.setCellValue("手机号");

        cell = firstRow.createCell(2);
        cell.setCellStyle(titleStyle);
        cell.setCellValue("名片");

        cell = firstRow.createCell(3);
        cell.setCellStyle(titleStyle);
        cell.setCellValue("抽奖码");

        cell = firstRow.createCell(4);
        cell.setCellStyle(titleStyle);
        cell.setCellValue("题目1");

        cell = firstRow.createCell(5);
        cell.setCellStyle(titleStyle);
        cell.setCellValue("题目2");

        cell = firstRow.createCell(6);
        cell.setCellStyle(titleStyle);
        cell.setCellValue("题目3");

        cell = firstRow.createCell(7);
        cell.setCellStyle(titleStyle);
        cell.setCellValue("题目4");

        cell = firstRow.createCell(8);
        cell.setCellStyle(titleStyle);
        cell.setCellValue("题目5");

        cell = firstRow.createCell(9);
        cell.setCellStyle(titleStyle);
        cell.setCellValue("题目6");

        cell = firstRow.createCell(10);
        cell.setCellStyle(titleStyle);
        cell.setCellValue("题目7");

        ByteArrayOutputStream os = new ByteArrayOutputStream();

        targetFile.getParentFile().mkdirs();
        try {
            book.write(os);
            byte[] ba = os.toByteArray();
            FileUtils.writeByteArrayToFile(targetFile, ba);
        } catch (IOException e) {
            LOGGER.error("", e);
        } finally {
            try {
                if (null != os) {
                    os.close();
                }
            } catch (IOException e) {
                LOGGER.error("", e);
            }
        }
        return targetFile;
    }

    public static void main(String[] args) {
        // getActivityXls(); // 运行下面代码时，注释掉该代码
        new Thread(() -> {
            System.out.println(write2File("1", "url1", "A,B,C,D,E,F,G"));
        }, "One").start();
        new Thread(() -> {
            System.out.println(write2File("2", "url1", "A,B,C,D,E,F,G"));
        }, "Two").start();

        new Thread(() -> {
            readFile();
        }, "readOne").start();

        new Thread(() -> {
            System.out.println(write2File("3", "url1", "A,B,C,D,E,F,G"));
        }, "Three").start();
        new Thread(() -> {
            System.out.println(write2File("4", "url1", "A,B,C,D,E,F,G"));
        }, "Four").start();


        new Thread(() -> {
            readFile();
        }, "readTwo").start();

        new Thread(() -> {
            System.out.println(write2File("5", "url1", "A,B,C,D,E,F,G"));
        }, "Five").start();
        new Thread(() -> {
            System.out.println(write2File("6", "url1", "A,B,C,D,E,F,G"));
        }, "Six").start();
    }
}
```
