---
title: 微信小程序之PHP发送模板消息
date: 2017-04-01 12:04:42
tags:
  - 小程序
  - PHP
---

小程序开放个人申请已经有个几天了，踩了无数的坑…折腾了半个多小时终于搞定了模板消息的推送，在这儿 share 一下咯

# 名词解释

首先，发送模板消息需要准备几样东西，先解释一下他们都是啥：

- `formId` 或 `prepay_id`
  
  用户必须得提交了表格或进行了支付才能推送模板消息，表格提交后能得到 `formId`，支付完成能得到 `prepay_id`，而且一个 `formId` 或 `prepay_id` 只能推送一条消息。

  > **支付**
  当用户在小程序内完成过支付行为，可允许开发者向用户在 7 天内推送有限条数的模板消息（1 次支付可下发 1 条，多次支付下发条数独立，互相不影响）
  **提交表单**
  当用户在小程序内发生过提交表单行为且该表单声明为要发模板消息的，开发者需要向用户提供服务时，可允许开发者向用户在7天内推送有限条数的模板消息（1次提交表单可下发1条，多次提交下发条数独立，相互不影响）

- `openID`

  “推送给谁”的用户标识符

- `template_id`

  模板消息的模板编号，可在微信小程序的后台申请

- `access_token`

  > access_token 是全局唯一接口调用凭据，开发者调用各接口时都需使用 access_token，请妥善保存。access_token 的存储至少要保留 512 个字符空间。access_token 的有效期目前为 2个小时，需定时刷新，重复获取将导致上次获取的 access_token 失效。

# 获取 `formID`

首先，个人小程序是无法进行支付的，所以只好用提交 form 的办法了：

``` html
<!-- submit.wxml -->

<form bindsubmit="submit" report-submit="true">
  <!-- 这里是表单的各种 <input> -->
  <button formType="submit">提交</button>
</form>
```

然后，在 JS 里接收 form 的 `formID`

``` javascript
// submit.js

submit: function (e) {
  var formID = e.detail.formId;

  // 这里用 wx.request 把数据提交给后端
}
```

# 发送模板消息

## 获取 `access_token`

`access_token` 需要用到自己小程序的 `appId` 和 `appsecret`，并调用微信的 API，PHP 脚本如下：

> **注**：我自己是把关于小程序的方法都封装在了一个 `Utility` 类里，这里就当作单独的函数列出了。

``` php
// getAccessToken.php

function getAccessToken ($appid, $appsecret) {					
  $url='https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid='.$appid.'&secret='.$appsecret;
  $html = file_get_contents($url);
  $output = json_decode($html, true);
  $access_token = $output['access_token'];

  return $access_token;
}
```

## 发送模板消息

```php
// sendNotice.php

require_once('getAccessToken.php');

// 根据你的模板对应的关键字建立数组
// color 属性是可选项目，用来改变对应字段的颜色
$data_arr = array(
  'keyword1' => array( "value" => $value, "color" => $color ) 
);

$post_data = array (
  // 用户的 openID，可用过 wx.getUserInfo 获取
  "touser"           => $openid,
  // 小程序后台申请到的模板编号
  "template_id"      => $templateid,
  // 点击模板消息后跳转到的页面，可以传递参数
  "page"             => "/pages/check/result?orderID=".$orderID,
  // 第一步里获取到的 formID
  "form_id"          => $formid,
  // 数据
  "data"             => $data_arr,
  // 需要强调的关键字，会加大居中显示
  "emphasis_keyword" => "keyword2.DATA"

);
		
// 发送 POST 请求的函数
// 你也可以用 cUrl 或者其他网络库，简单的请求这个函数就够用了			
function send_post( $url, $post_data ) {
  $options = array(
    'http' => array(
      'method'  => 'POST',
      // header 需要设置为 JSON
      'header'  => 'Content-type:application/json',
      'content' => $post_data,
      // 超时时间
      'timeout' => 60
    )
  );

  $context = stream_context_create( $options );
  $result = file_get_contents( $url, false, $context );

  return $result;
}

// 这里替换为你的 appID 和 appSecret
$url = "https://api.weixin.qq.com/cgi-bin/message/wxopen/template/send?access_token=" . getAccessToken ($appid, $appsecret);  
// 将数组编码为 JSON
$data = json_encode($post_data, true);   

// 这里的返回值是一个 JSON，可通过 json_decode() 解码成数组
$return = send_post( $url, $data);
var_dump($return);
```

# 返回值（直接复制疼训爸爸的原话咯）

| 返回码        | 说明                                         |
| :----------: |----------------------------------------------|
| 0            | 一切正常，推送成功                              |
| 40037        | `template_id` 不正确                          |
| 41028        | `form_id` 不正确，或者过期                     |
| 41029        | `form_id` 已被使用                            |
| 41030        | `page`不正确                                  |
| 45009        | 接口调用超过限额（目前默认每个帐号日调用限额为100万）|

最终推送出来的效果如下：
![](http://upload-images.jianshu.io/upload_images/5477931-af139617f0311bd1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)