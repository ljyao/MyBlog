---
title: parcelable to bytes
date: 2016-09-08 10:07:52
tags: tools
---
实现parcelable接口的可序列化对象与byte[]相互转换，从而实现保存对象到文件或数据库。

parcelable to bytes
```
Parcel parcel = Parcel.obtain();
recorderConfiguration.writeToParcel(parcel, 0);
byte[] bundleBytes = parcel.marshall();
parcel.recycle();
```
bytes to parcelable
```
byte[] configBytes ;
Parcel parcel = Parcel.obtain();
parcel.unmarshall(configBytes, 0, configBytes.length);
parcel.setDataPosition(0);
request.recorderConfiguration = new RecorderConfiguration(parcel);
parcel.recycle();
```