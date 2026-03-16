# 🚀 Expo React Native 微信支付集成全攻略

在 Expo 项目中集成微信支付曾是许多开发者的“噩梦”，但有了 **[expo-react-native-wechat-v2](https://github.com/zeeklog/expo-react-native-wechat-v2)**，这一切变得前所未有的简单。本文将手把手教你如何在 Expo 项目中从零开始实现微信支付。

---

## 📦 第一步：安装依赖

首先，在你的 Expo 项目中安装 SDK。推荐使用 `expo install` 以确保版本兼容性：

```bash
npx expo install expo-react-native-wechat-v2
```

---

## ⚙️ 第二步：环境配置

微信支付需要原生环境的支持，因此我们需要在 `app.json` 中进行必要的配置，并使用 Expo 的 `prebuild` 功能。

### 1. 修改 `app.json`
在 `expo` 配置项下，添加 iOS 的 URL Schemes 和白名单：

```json
{
  "expo": {
    "ios": {
      "bundleIdentifier": "com.yourname.app",
      "infoPlist": {
        "LSApplicationQueriesSchemes": ["weixin", "weixinULAPI", "weixinURLParamsAPI"],
        "CFBundleURLTypes": [
          {
            "CFBundleTypeRole": "Editor",
            "CFBundleURLName": "weixin",
            "CFBundleURLSchemes": ["wx你的AppID"] 
          }
        ]
      }
    },
    "android": {
      "package": "com.yourname.app"
    }
  }
}
```

### 2. 生成原生目录
执行以下命令生成 `ios` 和 `android` 目录：

```bash
npx expo prebuild
```

> **注意**：微信支付涉及原生代码，无法在普通的 Expo Go App 中直接测试，你需要构建 **Development Build**。

---

## 🛠️ 第三步：代码实现

### 1. 初始化 SDK
在应用入口（如 `App.js`）中注册微信应用。**注意：iOS 必须配置 Universal Link。**

```javascript
import * as WeChat from 'expo-react-native-wechat-v2';
import { useEffect } from 'react';

export default function App() {
  useEffect(() => {
    WeChat.registerApp('wx你的AppID', 'https://你的域名.com/');
  }, []);
  
  // ...
}
```

### 2. 监听支付回调
微信支付的结果是通过事件返回的，我们需要提前设置监听器：

```javascript
import { DeviceEventEmitter } from 'react-native';

useEffect(() => {
  const subscription = DeviceEventEmitter.addListener('WeChat_Resp', (resp) => {
    if (resp.type === 'PayReq.Resp') {
      if (resp.errCode === 0) {
        console.log('支付成功！');
      } else {
        console.log('支付失败或取消', resp.errStr);
      }
    }
  });

  return () => subscription.remove();
}, []);
```

### 3. 发起支付请求
当用户点击支付按钮时，从你的后端服务器获取支付参数（prepayId 等），然后调用 `pay` 方法：

```javascript
const handlePay = async () => {
  // 1. 检查是否安装微信
  const isInstalled = await WeChat.isWXAppInstalled();
  if (!isInstalled) {
    alert('请先安装微信');
    return;
  }

  // 2. 调用支付（参数由后端返回）
  try {
    await WeChat.pay({
      partnerId: '商家ID',
      prepayId: '预支付订单ID',
      nonceStr: '随机字符串',
      timeStamp: '时间戳',
      package: 'Sign=WXPay',
      sign: '签名'
    });
  } catch (error) {
    console.error('发起支付失败', error);
  }
};
```

---

## ⚠️ 关键注意事项

1.  **签名校验**：`pay` 方法中的所有参数（尤其是 `sign`）必须由后端根据微信支付 API 规则生成，切勿在前端硬编码私钥。
2.  **Universal Link**：iOS 端如果 Universal Link 配置不正确，支付完成后可能无法正确跳回你的 APP。
3.  **包名/Bundle ID**：确保你在微信开放平台填写的包名与 `app.json` 中的一致，否则会报“签名不对”的错误。

---

## 🚀 结语

通过 `expo-react-native-wechat-v2`，你可以在保持 Expo 开发便利性的同时，完美集成微信支付功能。

如果你在集成过程中遇到任何问题，欢迎访问 [GitHub 仓库](https://github.com/zeeklog/expo-react-native-wechat-v2) 提交 Issue。
