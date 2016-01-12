title: Formatting and Styling
date: 2016-01-11 19:07:12
category: android
tags: format
---
 # string.xml 中的转义
 - ' "需要用\转义或对整个字符串加上 ""
 ```
<string name="good_example">This\'ll work</string>
<string name="good_example_2">"This'll also work"</string>
```

 - " 需要用\转义,写成 \"
 ```
<string name="good_example">This is a \"good string\".</string>
 ```

 # Formatting strings
 -用String.format(String, Object...), 需要对%s标序号，写成 %1$s

```
<string name="welcome_messages">Hello, %1$s! You have %2$d new messages.</string>
In this example, the format string has two arguments: %1$s is a string and %2$d is a decimal number. 
You can format the string with arguments from your application like this:

Resources res = getResources();
String text = String.format(res.getString(R.string.welcome_messages), username, mailCount);
```
[参考文章](http://developer.android.com/intl/zh-cn/guide/topics/resources/string-resource.html#FormattingAndStyling)