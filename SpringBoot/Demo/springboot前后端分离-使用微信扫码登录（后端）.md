# springboot前后端分离-使用微信扫码登录（后端）

最近项目需要，临时加入了微信扫码登录的功能，让身为小白的我很是折磨，在网上搜了很多的教程，也被一些cv博客误导了许久，本篇blog主要从小白（我自己）的角度介绍前后端分离项目中如何接入微信扫码登录功能（仅后端）

# 扫码登录的基本流程

使用微信扫码登录 我们肯定要先来了解一下扫码登录的基本流程啦

1. 前端首先向服务端发送一个请求，用来获取二维码的url和唯一随机数（用UUID即可，这个随机数可以理解为这个二维码的key值，一一对应，所以尽量要用唯一的）
2. 同时服务端要记录这条随机数（存redis和其他数据库均可，找个地方记录下来）
3. 前端自从收到二维码和随机数后，展示二维码，并轮询一个检查二维码状态的接口（用来判断用户是否扫码并确认）**很关键的一步**
4. 客户扫码并确认后，会回调给服务端一个请求，服务端就能拿到对应的二维码的key（之前产生的随机数）
5. 前端轮询中 发现二维码状态值变为用户扫码已确认的值后，向后端发送业务请求（登录。。。。）

## 微信扫码登录的流程

首先你要有微信开放平台的资质和已审核过的应用（这里我就不过多赘述了，实际我也不太懂##）

**官方文档：https://developers.weixin.qq.com/doc/oplatform/Website_App/WeChat_Login/Wechat_Login.html**

![img](https://s2.loli.net/2022/04/18/rmOAwW2fynUx1DK.png)



首先要在微信开放平台网站应用里设置授权回调域

这个地方要注意不要加http:// https://前缀  要用域名，不要用ip地址  可以加端口号（我自己做的时候加了端口号，是可以的，但是我不知道有些博客上说不能加端口）（不要在这加接口哈！至于微信是怎么知道回调的接口的  是在获取二维码url时候拼接一个请求wx的url，在这个url里有一个redirect_uri参数 在这里指定接口）

本机测试的话 建议进行一下内网穿透 我使用的工具（https://suidao.oauth.k9s.run/）

![image-20220418194642151](https://s2.loli.net/2022/04/18/JkA1WVO8dfure72.png)

![image-20220418190205376](https://s2.loli.net/2022/04/18/sZhOU6iSqjDVI4T.png)

### coding的流程

1. 给微信发个请求 获取二维码url

   这部其实就是拼接请求url 具体看下官方文档 我在这里就不cv了

   注：我这里state的作用：把二维码和一个随机数绑定起来（随机数可理解为二维码的key，设为唯一的）

   把state设为这个key 并发送给前端，让前端轮询的时候带上这个key

   ```java
   /**
        * 生成二维码
        * @return
        */
       @GetMapping("/getWechatQtCode")
       public Object getWechatQtCode(){
           JSONObject res = new JSONObject();
           try {
               // 生成随机数 作为二维码的key
               String key = UUID.randomUUID().toString().replace("-","");
   
               String redirectUri2 = URLEncoder.encode(redirectUri,"utf-8");
   
               String oauthUrl = " https://open.weixin.qq.com/connect/qrconnect" +
                       "?appid=" + appId +
                       "&redirect_uri=" + redirectUri2 +
                       "&response_type=code" +
                       "&scope=snsapi_login" +
                       "&state=" + key +
                       "#wechat_redirect";
   
   
               res.put("key",key);
               res.put("url",oauthUrl);
               return res;
           } catch (Exception e) {
               System.out.println("二维码生成失败");
               return null;
           }
       }
   ```

   

2. 微信在扫码用户确认后，回调接口

   扫码用户点击确认后，微信会回调服务端接口，这个回调的接口就是你拼接生成二维码接口中拼接url的**redirect_uri** （注：这个redirect_uri 的前缀要与你在微信开放平台中填写的回调域一致）

   回调接口会带上两个参数（code:二维码的唯一标识，用于获取**access_token**，state:生成二维码接口中拼接url中的参数）

   **思考：此时是后端能通过code获取扫码用户的信息，但是怎么发送给前端呢？**（注：普通的http协议 后端是不能主动给前端发送数据的）

   方案一：使用websocket，后端主动把数据发送给前端

   方案二：把微信回调的参数code和state存储到数据库中（state作为key） 因为前端是能拿到state的，在前端拿到state后（就是获取二维码url的时间），开始轮询一个检查用户扫码状态的接口（state作为参数），后端去数据库中查这个state是否存在，若不存在，代表用户还未扫码，若存在，则通过code把获取的用户信息返回给前端

   我在本篇blog中用的方案二

   ```java
   /**
        * 微信认证/登录的回调方法（微信开放平台填写的回调域）
        * @return
        */
       @RequestMapping("/api/ucenter/wx/callback")
       public void wxCallBack(@RequestParam("code")String code,@RequestParam("state")String state){
           //把二维码url和key存入redis
           redisUtil.set(state,code,300);
   
       }
   ```

   

3. 轮询的接口

   这个接口中，通过code获取了access_token，又通过access_token获取了userInfo（具体看一下官方文档）

   实际上是使用了HttpClient向微信发送了两次请求（代码中使用的HttpClient工具类请看blog末尾的github源码）

   ```java
   @RequestMapping("/wxLogin/checkState")
       public Object checkState(@RequestBody String json){
           JSONObject params = JSONObject.parseObject(json);
           JSONObject res = new JSONObject();
           String key = params.getString("key");
           Object obj = redisUtil.get(key);
           if (obj==null){
               res.put("code",1);
               // 未扫码
               res.put("state",0);
           }else{
               res.put("code",1);
               // 扫码并已确认
               res.put("state",2);
               //获取微信用户信息
               //获取access_token
               String code = (String) obj;
               String url = "https://api.weixin.qq.com/sns/oauth2/access_token" +
                       "?appid=" + appId +
                       "&secret=" + appSecret +
                       "&code=" + code +
                       "&grant_type=authorization_code";
               JSONObject resultObject = HttpClientUtil.httpGet(url);
   
               //请求获取userInfo 并放入返回结果中
               String infoUrl = "https://api.weixin.qq.com/sns/userinfo" +
                       "?access_token=" + resultObject.getString("access_token") +
                       "&openid=" + resultObject.getString("openid") +
                       "&lang=zh_CN";
               JSONObject resultInfo = HttpClientUtil.httpGet(infoUrl);
               res.put("userInfo",resultInfo);
           }
           return res;
       }
   ```

   

源码：https://github.com/xiaoping123456/wxLogin_demo/tree/master