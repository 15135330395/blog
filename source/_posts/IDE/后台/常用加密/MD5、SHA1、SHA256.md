---
title: MD5、SHA1、SHA256
date: 2019-09-21 00:00:00
tags: [加密]
categories: 常用加密
---
以下是对数据加密的几种方法 MD5,sha1,sha256;  
```java
import java.io.UnsupportedEncodingException;  
    import java.security.MessageDigest;  
    import java.security.NoSuchAlgorithmException;  
    import java.util.Scanner;  
    import org.apache.commons.codec.binary.Hex;  
/** 
 * 常見的几种加密方式 
 */  
public class Encrypt {  
    public static void main(String[] args) {  
        Scanner sc = new Scanner(System.in);  
        String next = sc.next(); //123  
        String md5str = string2MD5(next);               //202CB962AC59075B964B07152D234B70  
        String sha1str = string2Sha1(next);             //40bd001563085fc35165329ea1ff5c5ecbdbbeef  
        String sha256str = String2SHA256(next);             //a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3  
        String sha256Javastr = String2SHA256StrJava(next);    //a665a45920422f9d417e4867efdc4fb8a04a1f3fff1fa07e998e86f7f7a27ae3  
        System.out.println("next");  
        System.out.println("md5str:"+md5str);  
        System.out.println("sha1str:"+sha1str);  
        System.out.println("sha256str:"+sha256str);  
        System.out.println("sha256Javastr："+sha256Javastr);  
    }  
    /** 
     * MD5加密 生成32位md5码 
     */  
    public static String string2MD5(String inStr){  
        MessageDigest md5 = null;  
        try{  
            md5 = MessageDigest.getInstance("MD5");  
        }catch (Exception e){  
            System.out.println(e.toString());  
            e.printStackTrace();  
            return "";  
        }  
        char[] charArray = inStr.toCharArray();  
        byte[] byteArray = new byte[charArray.length];  

        for (int i = 0; i < charArray.length; i++)  
            byteArray[i] = (byte) charArray[i];  
        byte[] md5Bytes = md5.digest(byteArray);  
        StringBuffer hexValue = new StringBuffer();  
        for (int i = 0; i < md5Bytes.length; i++){  
            int val = ((int) md5Bytes[i]) & 0xff;  
            if (val < 16)  
                hexValue.append("0");  
            hexValue.append(Integer.toHexString(val));  
        }  
        return hexValue.toString().toUpperCase();  
    }  
    /** 
     * SHA1加密  
     * @param str 
     * @return 
     */  
    public static String string2Sha1(String str){  
        if(str==null||str.length()==0){  
            return null;  
        }  
        char hexDigits[] = {'0','1','2','3','4','5','6','7','8','9',  
                'a','b','c','d','e','f'};  
        try {  
            MessageDigest mdTemp = MessageDigest.getInstance("SHA1");  
            mdTemp.update(str.getBytes("UTF-8"));  
  
            byte[] md = mdTemp.digest();  
            int j = md.length;  
            char buf[] = new char[j*2];  
            int k = 0;  
            for (int i = 0; i < j; i++) {  
                byte byte0 = md[i];  
                buf[k++] = hexDigits[byte0 >>> 4 & 0xf];  
                buf[k++] = hexDigits[byte0 & 0xf];        
            }  
            return new String(buf);  
        } catch (Exception e) {  
            return null;
        }  
    }  
  
    /** 
     *  利用Apache的工具类实现SHA-256加密 
     *  所需jar包下載 http://pan.baidu.com/s/1nuKxYGh 
     * @param str 加密后的报文 
     * @return 
     */  
    public static String String2SHA256(String str){  
        MessageDigest messageDigest;  
        String encdeStr = "";  
        try {  
            messageDigest = MessageDigest.getInstance("SHA-256");  
            byte[] hash = messageDigest.digest(str.getBytes("UTF-8"));  
            encdeStr = Hex.encodeHexString(hash);  
        } catch (NoSuchAlgorithmException e) {  
            e.printStackTrace();  
        } catch (UnsupportedEncodingException e) {  
            e.printStackTrace();  
        }  
        return encdeStr;  
    }  
    //---------  
    /** 
     * 利用java原生的摘要实现SHA256加密 
     * @param str 加密后的报文 
     * @return 
     */  
    public static String String2SHA256StrJava(String str){  
        MessageDigest messageDigest;  
        String encodeStr = "";  
        try {  
            messageDigest = MessageDigest.getInstance("SHA-256");  
            messageDigest.update(str.getBytes("UTF-8"));  
            encodeStr = byte2Hex(messageDigest.digest());  
        } catch (NoSuchAlgorithmException e) {  
            e.printStackTrace();  
        } catch (UnsupportedEncodingException e) {  
            e.printStackTrace();  
        }  
        return encodeStr;  
    }  
  
    /** 
     * 将byte转为16进制 
     * @param bytes 
     * @return 
     */  
    private static String byte2Hex(byte[] bytes){  
        StringBuffer stringBuffer = new StringBuffer();  
        String temp = null;  
        for (int i=0;i<bytes.length;i++){  
            temp = Integer.toHexString(bytes[i] & 0xFF);  
            if (temp.length()==1){  
                //1得到一位的进行补0操作  
                stringBuffer.append("0");  
            }  
            stringBuffer.append(temp);  
        }  
        return stringBuffer.toString();  
    }  
}  
```