# wechat-clj微信开发全家桶: 微信支付,网络授权,加密库等

- [wechat-clj微信开发全家桶: 微信支付,网络授权,加密库等](#wechat-clj%E5%BE%AE%E4%BF%A1%E5%BC%80%E5%8F%91%E5%85%A8%E5%AE%B6%E6%A1%B6-%E5%BE%AE%E4%BF%A1%E6%94%AF%E4%BB%98%E7%BD%91%E7%BB%9C%E6%8E%88%E6%9D%83%E5%8A%A0%E5%AF%86%E5%BA%93%E7%AD%89)
  - [Features](#features)
  - [Usage](#usage)
    - [小程序`wx.login`登陆使用后端的jscode2session获取微信信息](#%E5%B0%8F%E7%A8%8B%E5%BA%8Fwxlogin%E7%99%BB%E9%99%86%E4%BD%BF%E7%94%A8%E5%90%8E%E7%AB%AF%E7%9A%84jscode2session%E8%8E%B7%E5%8F%96%E5%BE%AE%E4%BF%A1%E4%BF%A1%E6%81%AF)
    - [微信服务号jsapi签名`wx.init` (用于微信网页支付和分享等的初始化)](#%E5%BE%AE%E4%BF%A1%E6%9C%8D%E5%8A%A1%E5%8F%B7jsapi%E7%AD%BE%E5%90%8Dwxinit-%E7%94%A8%E4%BA%8E%E5%BE%AE%E4%BF%A1%E7%BD%91%E9%A1%B5%E6%94%AF%E4%BB%98%E5%92%8C%E5%88%86%E4%BA%AB%E7%AD%89%E7%9A%84%E5%88%9D%E5%A7%8B%E5%8C%96)
  - [License](#license)

## Features

* 微信支付的后端相关加密,包含小程序和微信服务号
* 微信服务号的二次授权和小程序的jscode2session
* 微信服务号获取用户信息的接口map
* shadow-cljs 编写小程序的需要的npm库`mini-program-cljs`, 运用`wx.*`的方法时mock小程序方法(写自己的wx.*的spec解释器)在普通网页上Repl开发出来 (参考: https://github.com/wechat-miniprogram/sm-crypto)

## Usage

```clojure
[wechat "0.1.2-SNAPSHOT"]
```

### 小程序`wx.login`登陆使用后端的jscode2session获取微信信息

* 小程序端

```js
wx.login({
    success: function (r) {
        var code = r.code;//登录凭证
        if (code) {
            //2、调用获取用户信息接口
            wx.getUserInfo({
                success: function (res) {
                    console.log({encryptedData: res.encryptedData, iv: res.iv, code: code})
                    //3.解密用户信息 获取unionId => 传给后端的jscode2session的API
                    //...
                },
                fail: function () {
                    console.log('获取用户信息失败')
                }
            })

        } else {
            console.log('获取用户登录态失败！' + r.errMsg)
        }
    },
    fail: function () {
        callback(false)
    }
})
```
* 后端的jscode2session的API

```clojure
(ns your.ns
  (:require [wechat-clj.jscode2session :refer [get-jscode2session decrypt-wxdata]]))

(defn jscode2session-api [{{:keys [jscode encrypted_data iv]} :params}]
  (let [{:keys [appid secret]} {:appid "小程序的appid" :secret "小程序的secret"}
        {:keys [errcode errmsg hints session_key expires_in openid unionid]}
        (get-jscode2session {:jscode jscode :wxapp-key wxapp-key :op-fn #(identity %)})]
    (let [wx-user-data
          (decrypt-wxdata {:encrypted_data encrypted_data
                           :session_key session_key
                           :iv iv})]
      ;; do
      wx-user-data
      ;;
      )))
```

### 微信服务号jsapi签名`wx.init` (用于微信网页支付和分享等的初始化)

```clojure

(ns your.ns
  (:require [wechat-clj.wxpay.public :as wx-public]))

(defn url-signature [url]
  (wx-public/url-signature
    {:appid {:appid "服务号的appid" :secret "服务号的secret"}
     :url url
     :debug true
     :official-account-jsapi-ticket "服务号的jsapi ticket: 详情见wechat-clj.public.core的文档"}))

(defn get-wx-jsapi-signature [request]
   (let [request-url (str
                       "http://your.domain"
                       (:uri request)
                       (wx-public/hash-params->url
                         (:params req)))]
     {:signature (-> request-url
                      url-signature
                      json/generate-string)}))

```

## License

Copyright © 2020 Steve Chan
