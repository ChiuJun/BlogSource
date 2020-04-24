---
title: PhoneNet 工具类实现
top: false
cover: false
img: 'http://q9apk9x38.bkt.clouddn.com/medias/featureimages/ssm.jpg'
toc: true
mathjax: true
date: 2020-04-06 15:45:34
password:
summary: package com.qfedu.common.utils
tags:
    - PhoneNet
    - J2EE
    - SSM
categories:
    - J2EE课程
---

# PhoneNet 工具类实现

## package com.qfedu.common.utils

> 在锋迷网的开发中，需要用到一些工具类  
例如  使用工具类解决邮箱中的编码格式问题、密码加密问题、订单号的生成需要的签名问题等。

## Base64Utils

- 实现字符串的Base64编码解码的工具类

### 前置知识

- Class Base64  
    - Since:JDK1.8   
    - Package java.util   
    - Method Summary 
        ```java
        // Returns a Base64.Decoder that decodes using the Basic type base64 encoding scheme.
        static Base64.Decoder getDecoder();
        // Returns a Base64.Encoder that encodes using the Basic type base64 encoding scheme.
        static Base64.Encoder getEncoder();
        ```
- Class Base64.Decoder
    - Since:JDK1.8   
    - Package java.util  
    - Method Summary 
        ```java
        // Decodes a Base64 encoded String into a newly-allocated byte array using the Base64 encoding scheme.
        byte[] decode​(String src);
        ```
- Class Base64.Encoder
    - Since:JDK1.8   
    - Package java.util  
    - Method Summary 
        ```java
        // Encodes the specified byte array into a String using the Base64 encoding scheme.
        String encodeToString​(byte[] src);
        ```
- Class String
    - Since:JDK1.0   
    - Package java.lang
    - Constructor Summary
        ```java
        // Constructs a new String by decoding the specified array of bytes using the platform's default charset.
        String​(byte[] bytes);
        ```

### 编码实现

```java
package com.qfedu.common.utils;

import java.util.Base64;

public class Base64Utils{

	public static String decode(String msg){

		return new String(Base64.getDecoder().decode(msg));
	}

	public static String encode(String msg){

		return Base64.getEncoder().encodeToString(msg.getBytes());
	}

}
```

## MessageDigestUtils

- 使用MessageDigest实现加密字符串的工具类

### 前置知识

- Class MessageDigest  
    - Since:JDK1.1   
    - Package java.security   
    - Method Summary 
        ```java
        // Returns a MessageDigest object that implements the specified digest algorithm.
        static MessageDigest getInstance​(String algorithm);
        // Updates the digest using the specified array of bytes.
        void update​(byte[] input);	
        // Completes the hash computation by performing final operations such as padding.
        byte[] digest();
        ```
- Class String
    - Since:JDK1.0   
    - Package java.lang
    - Method Summary 
        ```java
        // Encodes this String into a sequence of bytes using the platform's default charset, storing the result into a new byte array.
        byte[] getBytes();
        ```
- Class BigInteger
    - Since:JDK1.1   
    - Package java.math
    - Constructor Summary
        ```java
        // Translates the sign-magnitude representation of a BigInteger into a BigInteger.
        // The sign is represented as an integer signum value: -1 for negative, 0 for zero, or 1 for positive.
        BigInteger​(int signum, byte[] magnitude);
        ```
    - Method Summary 
        ```java
        // Returns the String representation of this BigInteger in the given radix.
        String toString(int radix);
        ```

### 编码实现

```java
package com.qfedu.common.utils;

import java.math.BigInteger;
import java.security.MessageDigest;
import java.security.NoSuchAlgorithmException;

public class MessageDigestUtils {

	public static String md5(String password){

		try {
			MessageDigest digest = MessageDigest.getInstance("MD5");
			digest.update(password.getBytes());

			return new BigInteger(1,digest.digest()).toString(16);
		} catch (NoSuchAlgorithmException e) {
			e.printStackTrace();
		}

		return null;
	}

}
```
## RandomUtils

- 本系统时常会用到随机数的生成  
    例如  激活码、订单编号等
- 实现生成随机数的工具类

### 前置知识

- Class Random  
    - Since:JDK1.0  
    - Package java.util 
    - Constructor Summary
        ```java
        // Creates a new random number generator.
        Random();
        ```
    - Method Summary 
        ```java
        // Returns a pseudorandom, uniformly distributed int value between 0 (inclusive) and the specified value (exclusive), drawn from this random number generator's sequence.
        // [0,bound)
        int	nextInt​(int bound);
        ```
- Class SimpleDateFormat
    - Since:JDK1.1  
    - Package java.text
    - Constructor Summary
        ```java
        // Constructs a SimpleDateFormat using the given pattern and the default date format symbols for the default FORMAT locale.
        SimpleDateFormat​(String pattern);	
        ```
    - Method Summary 
        ```java
        // Formats the given Date into a date/time string and appends the result to the given StringBuffer.
        StringBuffer format​(Date date, StringBuffer toAppendTo, FieldPosition pos);
        ```
- Class Calendar
    - Since:JDK1.1   
    - Package java.util
    - Method Summary 
        ```java
        // Gets a calendar using the default time zone and locale.
        static Calendar	getInstance();
        // Returns a Date object representing this Calendar's time value (millisecond offset from the Epoch").
        // Epoch:January 1, 1970 00:00:00.000 GMT (Gregorian).
        Date getTime();
        ```
- Class Integer
    - Since:JDK1.0
    - Package java.lang
    - Method Summary 
        ```java
        // Returns a string representation of the integer argument as an unsigned integer in base 16.
        static String	toHexString​(int i);
        ```
- Class UUID
    - Since:JDK1.5
    - Package java.util
    - Method Summary 
        ```java
        // UUID:Universally Unique Identifier
        // Static factory to retrieve a type 4 (pseudo randomly generated) UUID.
        static UUID	randomUUID();
        // Returns a String object representing this UUID.
        String	toString();
        ```

### 编码实现

```java
package com.qfedu.common.utils;

import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Random;
import java.util.UUID;

public class RandomUtils{

	public static String getTime(){

		return new SimpleDateFormat("yyyyMMddHHmmssSSS").format(Calendar.getInstance().getTime());
	}

	public static String createActive(){

		return getTime()+Integer.toHexString(new Random().nextInt(900)+100);
	}

	public static String createOrderId(){

		return getTime()+UUID.randomUUID().toString();
	}

}
```

## FileUtils

- 文件工具类

### 前置知识

- Class File
    - Since:JDK1.0  
    - Package java.io
    - Constructor Summary
        ```java
        // Creates a new File instance from a parent abstract pathname and a child pathname string.
        File​(File parent, String child);	
        // Creates a new File instance by converting the given pathname string into an abstract pathname.
        File​(String pathname);
        ```
    - Method Summary 
        ```java
        // Tests whether the file or directory denoted by this abstract pathname exists.
        boolean	exists();	
        // Returns the pathname string of this abstract pathname's parent, or null if this pathname does not name a parent directory.
        String	getParent();
        // Creates the directory named by this abstract pathname, including any necessary but nonexistent parent directories.
        boolean	mkdirs();
        ```

### 编码实现

```java
package com.qfedu.common.utils;

import java.io.File;
import java.text.SimpleDateFormat;
import java.util.Date;
import java.util.UUID;

public class FileUtils {
	
	//创建文件夹 一个月一个文件夹
	public static File createDir(String dir) {
		//子文件名称 yyyyMM
		String month=new SimpleDateFormat("yyyyMM").format(new Date());

		File dir1=new File(new File(dir).getParent(),"fmwimages");
		File dir2=new File(dir1,month) ;
		if(!dir2.exists()) {
			dir2.mkdirs();
		}

		return dir2;
	}

	//创建唯一名称 
	public static String createFileName(String fileName) {
		if(fileName.length()>30) {
			fileName=fileName.substring(fileName.length()-30);
		}
		return UUID.randomUUID().toString()+"_"+fileName;
	}

}
```

## StrUtils

- 字符串校验工具类

### 前置知识

- 略

### 编码实现

```java
package com.qfedu.common.utils;

public class StrUtils {

	/**
	 * 是否为空的校验
	 * @return true 空 false 非空*/
	public static boolean isEmpty(String...msg){
		boolean res = false;
		for(String s:msg){
			res = (s==null|| s.length()==0);
			if(res) {
				break;
			}
		}
		return res;
	}

}
```

## EmailUtils
- 当用户需要注册会员账号时将会用到验证码验证  
    本系统是采用邮箱发送验证码的方式来实现的，另外邮箱还具备推送日常活动信息的功能等
- 实现自动发送激活邮件的工具类

### 前置知识

- 略

### 编码实现

```java
package com.qfedu.common.utils;

import java.io.UnsupportedEncodingException;
import java.net.Inet4Address;
import java.net.UnknownHostException;
import java.util.Date;
import java.util.Properties;
import javax.mail.MessagingException;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeMessage;
import com.qfedu.domain.User;

/*
 * 基于JDK实现邮件发送
 * 主要是实现激活码的发送
 * */
public class EmailUtils {
	public static void sendEmail(User user){
		//邮箱
		String myAccount = "";
		//授权码
		String myPass = "";
		//邮箱服务器
		String SMTPHost = "smtp.163.com";
		//设置属性信息
		Properties prop = new Properties();
		//设置协议
		prop.setProperty("mail.transport.protocol", "smtp");
		//邮件服务器
		prop.setProperty("mail.smtp.host", SMTPHost);
		//认证
		prop.setProperty("mail.smtp.auth", "true");
		//1、创建会话
		Session session = Session.getDefaultInstance(prop);
		//设置是否需要调试
		session.setDebug(false);
		//2、创建发送信息
		MimeMessage message = createMsg(session,myAccount,user);
		//4发送信息操作
		try {
			
			Transport tran = session.getTransport();
			//连接
			tran.connect(myAccount, myPass);
			//发送消息 
			tran.sendMessage(message, message.getAllRecipients());
			//关闭
			tran.close();
		} catch (MessagingException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	//生成邮件消息
	private static MimeMessage createMsg(Session session, String myAccount, User user) {
		//创建消息对象
		MimeMessage message = new MimeMessage(session);
		//设置
		try {
			//3.1发送方
			message.setFrom(new InternetAddress(myAccount, "锋迷网官方邮件", "utf-8"));
			//3.2设置接收方
			/*
			 * MimeMessage.RecipientType.TO 
			 * MimeMessage.RecipientType.CC 
			 * MimeMessage.RecipientType.BCC 
			 * */
			message.setRecipient(MimeMessage.RecipientType.TO, new InternetAddress(user.getEmail(), user.getUsername(), "utf-8"));
			//3.3 设置主题
			message.setSubject("锋迷网激活码","utf-8");
			//获取本机的ip地址
			String ip = Inet4Address.getLocalHost().getHostAddress();
			String url = "http://"+ip+":8086/PhoneNet/activate?e="+ Base64Utils.encode(user.getEmail())+"&c="+Base64Utils.encode(user.getActivatecode());
			//设置正文信息
			message.setContent(user.getUsername()+",欢迎你加入我们<br>为了更好体验我们的产品，请<a href='"+url+"'>点击激活 "+url+"</a>","text/html;charset=utf-8");
			//设置日期
			message.setSentDate(new Date());
			//保存
			message.saveChanges();
		} catch (UnsupportedEncodingException | MessagingException | UnknownHostException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return message;
	}
}
```
