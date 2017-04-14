---
title: 断点下载与解压ZIP
date: 2017-01-13 16:05:24
tags:
---
# 初始化
获取文件大小，已下载文件大小
```
private void init() throws Exception {
        URL url = new URL(httpUrl);
        HttpURLConnection connection = (HttpURLConnection) url.openConnection();
        connection.setConnectTimeout(5000);
        connection.setRequestMethod("GET");
        fileSize = connection.getContentLength();
        connection.disconnect();

        Context context = NiceApplication.getApplication().getApplicationContext();
        File fileDir = StorageUtils.getIndividualFileDirectory(context, "nice-lens-resource");
        rootDir = fileDir.getAbsolutePath();
        File file = new File(fileDir, fileName + ".zip");
        if (!file.exists()) {
            file.createNewFile();
            completeSize = 0;
        } else {
            completeSize = file.length();
        }
        zipFilePath = file.getAbsolutePath();
    }
```
# 断点下载
设置http头，指定下载开始和结束点，"Range"->"bytes=startPosi-endPosi"
```
 private String download() throws Exception {
        if (completeSize == fileSize) {
            return zipFilePath;
        }

        HttpURLConnection connection;
        RandomAccessFile randomAccessFile;
        InputStream is;
        URL url = new URL(httpUrl);
        connection = (HttpURLConnection) url.openConnection();
        connection.setConnectTimeout(5000);
        connection.setRequestMethod("GET");
        // 设置范围，格式为Range：bytes x-y;
        connection.setRequestProperty("Range", String.format("bytes=%s-%s", completeSize, fileSize));

        randomAccessFile = new RandomAccessFile(zipFilePath, "rwd");
        randomAccessFile.seek(completeSize);
        is = connection.getInputStream();
        byte[] buffer = new byte[1024];
        int length;
        while ((length = is.read(buffer)) != -1) {
            randomAccessFile.write(buffer, 0, length);
            completeSize += length;
            update();
        }
        return zipFilePath;
    }
```
# 解压zip
主要使用ZipInputStream
```
private static void unZipFolder(String zipFilePath, String outputDir) throws Exception {
        FileUtils.deleteFile(outputDir);

        ZipInputStream inZip = new ZipInputStream(new FileInputStream(zipFilePath));
        ZipEntry zipEntry;
        String szName = "";
        while ((zipEntry = inZip.getNextEntry()) != null) {
            szName = zipEntry.getName();

            if (szName.contains("__MACOSX")) {
                continue;
            }

            if (zipEntry.isDirectory()) {
                // get the folder name of the widget
                szName = szName.substring(0, szName.length() - 1);
                File folder = new File(outputDir + File.separator + szName);
                folder.mkdirs();
            } else {
                File file = new File(outputDir + File.separator + szName);
                file.createNewFile();
                // get the output stream of the file
                FileOutputStream out = new FileOutputStream(file);
                int len;
                byte[] buffer = new byte[1024];
                // read (len) bytes into buffer
                while ((len = inZip.read(buffer)) != -1) {
                    // write (len) byte from buffer at the position 0
                    out.write(buffer, 0, len);
                    out.flush();
                }
                out.close();
            }
        }
        inZip.close();
    }
```
