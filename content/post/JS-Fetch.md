+++
title = "JS Fetch"
date = "2021-01-31T17:20:38+08:00"
tags = ["js","web"]
+++
Fetch API 提供了一个获取资源的接口（包括跨域请求）。是**XMLHttpRequest**的一种替代方案。

fetch不是ajax的进一步封装，而是原生js。
<!--more-->
# ajax与fetch的使用对比
## ajax
- 使用步骤
  1. 创建XmlHttpRequest对象
  2. 调用open方法设置基本请求信息
  3. 设置发送的数据，发送请求
  4. 注册监听的回调函数
  5. 拿到返回值，对页面进行更新

```javascript
//1.创建Ajax对象
    if(window.XMLHttpRequest){
       var oAjax=new XMLHttpRequest();
    }else{
       var oAjax=new ActiveXObject("Microsoft.XMLHTTP");
    }

    //2.连接服务器（打开和服务器的连接）
    oAjax.open('GET', url, true);

    //3.发送
    oAjax.send();

    //4.接收
    oAjax.onreadystatechange=function (){
       if(oAjax.readyState==4){
           if(oAjax.status==200){
              //alert('成功了：'+oAjax.responseText);
              fnSucc(oAjax.responseText);
           }else{
              //alert('失败了');
              if(fnFaild){
                  fnFaild();
              }
           }
        }
    };
```

## fetch
Fetch 提供了对 [`Request`](https://developer.mozilla.org/zh-CN/docs/Web/API/Request) 和 [`Response`](https://developer.mozilla.org/zh-CN/docs/Web/API/Response) （以及其他与网络请求有关的）对象的通用定义。

它同时还为有关联性的概念，例如CORS和HTTP原生头信息，提供一种新的定义，取代它们原来那种分离的定义。

`fetch()` 必须接受一个参数——资源的路径。无论请求成功与否，它都返回一个 Promise 对象，resolve 对应请求的 [`Response`](https://developer.mozilla.org/zh-CN/docs/Web/API/Response)。你也可以传一个可选的第二个参数 `init`（参见 [`Request`](https://developer.mozilla.org/zh-CN/docs/Web/API/Request)）。
```javascript
let btn1 = document.querySelector('#btn1');
btn1.onclick = ()=>{
    let form_data = new FormData();
    form_data.append('id',233);
    form_data.append('name','qlel');
    form_data.append('msg','this is fetch post test 测试');
    fetch('file2.php',{
        method:'POST',
        body: form_data
    })
    .then((response)=>{
        return response.json()
        // .then(data=>console.log(data))
    })
  	.then(data=>console.log(data))
    .catch((error)=>console.log(error))
}
```

## fetch和ajax 的主要区别
1. `fetch()`返回的promise将不会拒绝http的错误状态，即使响应是一个HTTP 404或者500
2. 在默认情况下 fetch不会接受或者发送cookies

---
# fetch支持的请求参数
`fetch(input[,init])`
`fetch()` 接受第一个参数为URL，也可以是一个[`Request`](https://developer.mozilla.org/zh-CN/docs/Web/API/Request) 对象；
`fetch()` 接受第二个可选参数，一个可以控制不同配置的 `init` 对象：

- `method` :请求使用的方法，如GET,POST;
- `headers`: 请求的头信息，形式为 [`Headers`](https://developer.mozilla.org/zh-CN/docs/Web/API/Headers) 的对象或包含 [`ByteString`](https://developer.mozilla.org/zh-CN/docs/Web/API/ByteString) 值的对象字面量。
- `body`: 请求的 body 信息：可能是一个 [`Blob`](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)、[`BufferSource`](https://developer.mozilla.org/zh-CN/docs/Web/API/BufferSource)、[`FormData`](https://developer.mozilla.org/zh-CN/docs/Web/API/FormData)、[`URLSearchParams`](https://developer.mozilla.org/zh-CN/docs/Web/API/URLSearchParams) 或者 [`USVString`](https://developer.mozilla.org/zh-CN/docs/Web/API/USVString) 对象。注意 GET 或 HEAD 方法的请求不能包含 body 信息。
- `mode`: 请求的模式，如 `cors、` `no-cors 或者` `same-origin。`
- `credentials`: 请求的 credentials，如 `omit`、`same-origin` 或者 `include`。为了在当前域名内自动发送 cookie ， 必须提供这个选项， 从 Chrome 50 开始， 这个属性也可以接受[`FederatedCredential`](https://developer.mozilla.org/zh-CN/docs/Web/API/FederatedCredential) 实例或是一个 [`PasswordCredential`](https://developer.mozilla.org/zh-CN/docs/Web/API/PasswordCredential) 实例。
- `cache`:  请求的 cache 模式: `default `、 `no-store 、` `reload 、` `no-cache 、` `force-cache `或者 `only-if-cached 。`
- `redirect`: 可用的 redirect 模式: `follow` (自动重定向), `error` (如果产生重定向将自动终止并且抛出一个错误), 或者 `manual` (手动处理重定向). 在Chrome中，Chrome 47之前的默认值是 follow，从 Chrome 47开始是 manual。
- `referrer`: 一个 [`USVString`](https://developer.mozilla.org/zh-CN/docs/Web/API/USVString) 可以是 `no-referrer、``client`或一个 URL。默认是 `client。`
- `referrerPolicy`: 指定了HTTP头部referer字段的值。可能为以下值之一： `no-referrer、` `no-referrer-when-downgrade、` `origin、` `origin-when-cross-origin、` `unsafe-url 。`
- `integrity`: 包括请求的  [subresource integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Subresource_Integrity) 值 （ 例如： `sha256-BpfBw7ivV8q2jLiT13fxDYAe2tJllusRSZ273h2nFSE=`）。

Body 类定义了以下方法（这些方法都被 Request 和Response所实现）以获取 body 内容。这些方法都会返回一个被解析后的Promise对象和数据。

- [`arrayBuffer()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Body/arrayBuffer)
- [`blob()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Body/blob)
- [`json()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Body/json)
- [`text()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Body/text)
- [`formData()`](https://developer.mozilla.org/zh-CN/docs/Web/API/Body/formData)

---
# 一些示例
## 上传文件
```javascript
var formData = new FormData();
var fileField = document.querySelector("input[type='file']");

formData.append('username', 'abc123');
formData.append('avatar', fileField.files[0]);

fetch('https://example.com/profile/avatar', {
  method: 'PUT',
  body: formData
})
.then(response => response.json())
.catch(error => console.error('Error:', error))
.then(response => console.log('Success:', response));
```