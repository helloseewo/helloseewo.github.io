---
title: 对于希沃管家激活码的研究
author: Sirin
date: 2024-04-27 09:24:00 +0800
categories: [希沃管家, 激活码, 解密]
tags: [希沃管家]
render_with_liquid: false
---

~~**本文发布于2024-04-27，当前该方法已被修复！**~~

**抱歉，眼瞎看错了。**

对于希沃管家激活码的研究。

## 锁屏
希沃管家的锁屏有三个解锁方式：扫码解锁、激活码解锁、密码解锁。本文研究的是其中的激活码解锁。

## 激活码
激活码的获取方式是用微信扫描二维码。希沃会自动读取及判断微信登录账号，匹配成功后直接弹出激活码信息。

激活码url格式(offline版，在线版同理)：`https://campus.seewo.com/hugo-mobile/#/offlinelock?_d=设备id&_k=时间戳(大概)&_p=激活码密文base64`

通过[官方文档](https://help.seewo.com/hugo/X2VZHaFbT3)可以得知，激活码不是单向加密。

## 加密方式
经过一番研究，本人发现激活码的url是用MD5和AES来加密的（主要还是AES）。

加密方式示例：

````js
// 导入 crypto-js 库
const CryptoJS = require("crypto-js");

// 定义 md5 函数
function md5(str) {
  return CryptoJS.MD5(str).toString();
}

// 定义 re.a.encrypt 函数
const re = {
  a: {
    encrypt: (str, key) => CryptoJS.AES.encrypt(str, key).toString()
  }
};

// 模拟 deviceId 和 webConfig.activationCodeUnlockTargetUrl
deviceId = '114514191981066666';
webConfig = {
  activationCodeUnlockTargetUrl: 'https://campus.seewo.com/hugo-mobile/#/offlinelock'
};

class ActivationCode {
  constructor() {
    this.password = null;
    this.clearTextKey = new Date().getTime();
    this.ciphertextKey = "";
    this.BOARD_LIST = [1, 2, 3, 4, 5, 6, 7, 8, 9, 0];
  }

  // 生成6位随机激活码
  newPassword() {
    let password = "";
    for (let i = 0; i < 6; i++) {
      password += Math.floor(10 * Math.random());
    }
    this.password = password;
  }

  // 生成密钥
  newCiphertextKey() {
    this.ciphertextKey = md5(this.clearTextKey + "" + deviceId);
  }

  // 生成二维码链接
  newQrcode() {
    const password = re.a.encrypt(this.password, this.ciphertextKey.toString());
    const t = webConfig.activationCodeUnlockTargetUrl;
    this.qrcodeUrl = t + "?_d=" + deviceId + "&_k=" + this.clearTextKey + "&_p=" + encodeURIComponent(password.toString());
    console.log(this.qrcodeUrl); // Log the QR code URL
  }
}

const code = new ActivationCode();

// 生成激活码
code.newPassword();
code.newCiphertextKey();
code.newQrcode();
````

这段代码是用JavaScript编写的，使用了crypto-js库来处理MD5和AES加密。实现了一个生成激活码和相关信息的类。

 - 导入crypto-js库，并定义md5函数。
 - 定义re.a.encrypt函数，用于加密字符串。
 - 模拟deviceId和webConfig.activationCodeUnlockTargetUrl。
 - 定义ActivationCode类，包含生成随机激活码、密钥、二维码链接等方法。
 - 在ActivationCode类的构造函数中，初始化类的属性，如密码、明文密钥、密文密钥等。
 - 定义生成6位随机激活码的方法。
 - 定义生成密钥的方法。
 - 定义生成二维码链接的方法。
 - 创建ActivationCode实例，并调用其方法生成激活码和相关信息。
 - 输出生成的二维码链接。

## 解密方式
知道了加密方式，我们可以尝试解密激活码。

````js
// 导入 crypto-js 库
const CryptoJS = require("crypto-js");

// 定义 md5 函数
function md5(str) {
  return CryptoJS.MD5(str).toString();
}

// 定义 re.a.decrypt 函数
const re = {
  a: {
    decrypt: (str, key) => CryptoJS.AES.decrypt(str, key).toString(CryptoJS.enc.Utf8)
  }
};

// 模拟 deviceId 和 webConfig.activationCodeUnlockTargetUrl
deviceId = '114514191981066666';
webConfig = {
  activationCodeUnlockTargetUrl: 'https://campus.seewo.com/hugo-mobile/#/offlinelock'
};

// 定义函数解析url参数
function parseUrl(url) {
  const params = {};
  const urlParts = url.split("?");
  if (urlParts.length > 1) {
    const queryString = urlParts[1];
    const pairs = queryString.split("&");
    pairs.forEach(pair => {
      const keyValue = pair.split("=");
      params[keyValue[0]] = decodeURIComponent(keyValue[1]);
    });
  }
  return params;
}

// 解析url参数
const url = "https://campus.seewo.com/hugo-mobile/#/offlinelock?_d=114514191981066666&_k=1714180968345&_p=U2FsdGVkX1%2FauFnk1GiKIOBRZnUIRweNFFwj0RrmuFo%3D";
const urlParams = parseUrl(url);

// 解密激活码
const ciphertextKey = md5(urlParams["_k"] + deviceId);
const decryptedPassword = re.a.decrypt(urlParams["_p"], ciphertextKey);

console.log("Decrypted Password:", decryptedPassword); // 打印解密后的激活码
````

通过这个，我们得出了激活码为`137453`。

## 我不知道为什么这个链接会出现在这
https://helloseewo.github.io/dec