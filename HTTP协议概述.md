# 一、HTTP协议概述概述



全称**超文本传输协议**(Hyper Text Transfer Protocol)。
定义服务端和客户端之间传输的协议。底层基于TCP/IP协议。
1990年提出。目前使用最多版本是[1.1](https://tools.ietf.org/html/rfc7230),最新版本2.0。



# 三、HTTP1.1协议模型
一次请求(Request),一次应答(Response)
- 一次请求对应一次应答
- 先有请求再有应答
- 同一客户端两次相同请求会认为是不同请求(两次无状态协议)
## (一)、URL



HTTP使用统一资源**标识**符（Uniform Resource **Identifiers**, URI）来标识资源。
URL(Uniform Resource **Locator**, 统一资源**定位**符),是一种特殊类型的URI。除了可唯一标识资源外，还标识如何获取资源的方式。

示例:

```url
ftp://8.8.8.8:21/somefile
https://www.baidu.com:443
http://www.xxoo.com:80/news/index.html?boardID=5&ID=24618&page=1#top
http://www.xxoo.com/news/index.html?boardID=5&ID=24618&page=1
http://www.xxoo.com:8080/news/index.png?type=tiny
```
其中:
- 协议部分(`http:`)
  - 表示需使用HTTP来获取资源
  - 还有https等协议
  - 后面的“//”为分隔符
- 域名部分(`www.xxoo.com`)。
  - 也可以直接使用IP地址
- 端口部分(8080)
  - 域名和端口之间使用“:”作为分隔符。
  - 可省略(http协议默认端口80，https默认443)
- 资源路径部分(`/news/index.html`)
  - 指定资源在目标服务器的路径(不一定是真实路径)
  - 可省略(默认可能是index.html之类)
- 参数部分(`boardID=5&ID=24618&page=1`)
  - `?`到`#`之间的部分为参数部分(如果无`#`，则到URL结束)
  - 参数分为参数名和参数值两部分，`=`隔开
  - 参数与参数之间用“&”作为分隔符
  - 可省略
- 锚部分(`#top`)
  - 用于指定要查看资源的哪部分
  - 可省略
## (二)、一个请求的数据结构
![][request]
分为三部分:

### 1、请求行
**格式:**`请求方法` `资源路径` `协议版本`
如:
```java
GET /news/index.html?boardID=5&ID=24618&page=1 HTTP/1.1
POST /news/index.html HTTP/1.1
```
**9种请求方法:**

- **GET**(用于获取资源)
- **POST**(用于新增)
- DELETE(用于删除资源)
- PUT(用于全部更新资源)
- PATCH(用于局部更新资源)
- HEAD(同GET，但不要求应答体)
- TRACE(用于追踪请求)
- OPTIONS(用于询问资源支持的操作方法)
- CONNECT(转换连接方式)
### 2、请求头们
- 一个请求头占一行
- 分为两部分，头名词和值，冒号分开
**常见请求头:**
|        协议头         | 说明                           | 示例                                         |
| :-------------------: | ------------------------------ | -------------------------------------------- |
|        Accept         | 可接受应答类型(MIME)           | `Accept`: `text/html,text/plain`             |
|    Accept-Charset     | 可接受字符集                   | `Accept-Charset`:`utf-8`                     |
|    Accept-Encoding    | 可接受应答编码                 | `Accept-Encoding`:`gzip, deflate`            |
|    Accept-Language    | 可接受应答语言列表             | `Accept-Language`:`en-US`                    |
|   **Cache-Control**   | **是否使用缓存机制**           | **`Cache-Control`:`no-cache`**               |
|      **Cookie**       | **之前`Set-Cookie`的内容**     | **`Cookie`:` jessionid=23243232980323`**     |
|   ***Content-Type**   | **请求体的MIME类型(POST/PUT)** | **`Content-Type`:` application/x-w...`**     |
|         Date          | 发送该消息的日期和时间         | `Date`:`Dec, 26 Dec 2015 17:30:00 GMT`       |
|         Host          | 服务器的域名:端口              | `Host`:` www.itbilu.com:80`                  |
| **If-Modified-Since** | **本地缓存的时间**             | **`If-Modified-Since`:`Dec,26 Dec 2015...`** |
|      **Origin**       | **第三方请求来源**             | **`Origin: http://www.xx.com`**              |
|        Pragma         | 遗留字段                       | `Pragma`:` no-cache`                         |
|        Referer        | 请求来源                       | `Referer`:` http://xxoo.com/nodejs`          |
|      User-Agent       | 浏览器身份标识                 | `User-Agent: Mozilla/……`                     |
|        Upgrade        | 要求服务器升级协议             | `Upgrade`:` HTTP/2.0, SHTTP...`              |
### 3、请求体
仅有PUT/POST/PATCH请求方式有请求体。
请求体格式取决于请求体`Content-type`设定的值。常见格式由:
- application/x-www-form-urlencoded
- 数据按照key1=value1&key1=value2&key3=value3的方式进行编码
- 允许key相同
- 类似URL参数形式
- key 和 val 都进行了 URL 转码(空格转换为 "+" 加号，特殊符号转换为 ASCII HEX 值)
```
boardID=5&ID=24618&page=1
```
- multipart/form-data
- 用于文件上传
- 格式后面一般有boundary=分隔符字符串
```java
Content-Type: multipart/form-data; boundary=--9051914041544843365972754266
```
- 示例:
```
-----------------------------9051914041544843365972754266
Content-Disposition: form-data; name="userId"
10086
-----------------------------9051914041544843365972754266
Content-Disposition: form-data; name="file1"; filename="a.jpg"
Content-Type: image/png
图片2进制
-----------------------------9051914041544843365972754266
Content-Disposition: form-data; name="file2"; filename="a.html"
Content-Type: text/html
文件内容(2进制)
-----------------------------9051914041544843365972754266--
```
- application/json
- 内容以json格式编码
- 示例
```java
{
"name":"10086",
"age":18
}
```
### 4、示例:
**GET请求**
```java
GET /news/index.html?boardID=5&ID=24618&page=1 HTTP/1.1
Host:http://www.xxoo.com:8080/
Accept:image/webp,image/*,*/*;q=0.8
Accept-Encoding:gzip, deflate, sdch
Accept-Language:zh-CN,zh;q=0.8
User-Agent:Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, lik...
...somedata...
```
**POST请求(application/x-www-form-urlencoded)**
```java
POST /news/index.html HTTP/1.1
Host:http://www.xxoo.com:8080/
Accept:image/webp,image/*,*/*;q=0.8
Accept-Encoding:gzip, deflate, sdch
Accept-Language:zh-CN,zh;q=0.8
Content-Type:application/x-www-form-urlencoded
Content-Length:40
Connection: Keep-Alive
User-Agent:Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, lik...
boardID=5&ID=24618&page=1
```
## (三)、一个应答的数据结构
![][responce]

分为三部分



### 1、应答行

```
HTTP/1.1 200 OK
```

**状态码:**

- 1xx 请求中
- **2xx 请求成功**
  - 200
  - 201
- **3xx 重定向**
  - 301
  - 302
  - 304
- **4xx 请求失败(客户端问题)**
  - 401
  - 403
  - 404
  - 405
- **5xx 请求失败(服务端问题)**
  - 500
### 2、应答头
- 一个请求头占一行
- 分为两部分，头名词和值，冒号分开
常见应答头:
| 响应头                      | 说明                   | 示例                                               |
| --------------------------- | ---------------------- | -------------------------------------------------- |
| Access-Control-Allow-Origin | 允许跨域网站           | `Access-Control-Allow-Origin`: `*`                 |
| Allow                       | 允许方法               | `Allow`: `GET, HEAD`                               |
| Cache-Control               | 缓存控制               | `Cache-Control`:`max-age=3600`                     |
| Content-Disposition         | 设置下载文件名称       | Content-Disposition`:`attachment; filename="1.txt" |
| Content-Encoding            | 应答编码类型           | `Content-Encoding`: `gzip`                         |
| Content-Language            | 应答语言               | `Content-Language`: `zh-cn`                        |
| **Content-Length**          | 应答长度               | `Content-Length`: `348`                            |
| Content-Location            | 候选位置               | `Content-Location`:`/index.htm`                    |
| ***Content-Type**           | 应答的`MIME`类型       | `Content-Type`:`text/html`                         |
| Date                        | 应答返回时间           | `Date`:`Tue, 15 Nov 1994...`                       |
| ETag                        | 资源标识符             | `ETag`:`737060cd8c284d8af7...`                     |
| Expires                     | 缓存过期时间           | `Expires`:`Thu, 01 Dec 1994 16:...`                |
| Last-Modified               | 最后修改时间           | `Last-Modified`:`Dec, 26 Dec...`                   |
| Location                    | 重定向地址或新资源地址 | `Location`:`http://www.xxx.com/123`                |
| Pragma                      | 与具体的实现相关       | `Pragma: no-cache`                                 |
| Refresh                     | 要求客户端刷新         | `Refresh`:`5;url=http://xx.com`                    |
| Server                      | 服务器的名称           | `Server`: `nginx/1.6.3`                            |
| Set-Cookie                  | 设置cookie             | `Set-Cookie`:`jsessionid=1002...`                  |
| Transfer-Encoding           | 传输编码形式           | `Transfer-Encoding`:` chunked`                     |
| Upgrade                     | 要求客户端升级协议。   | `Upgrade`: `HTTP/2.0..`                            |
### 3、应答体
实际的资源内容，可能是网页，可能是图片/音频等2仅在数据
# 四、MIME
多用途互联网邮件扩展类型(**M**ultipurpose **I**nternet **M**ail **E**xtensions)。

请求头和应答头`Content-Type`的值使用此格式。

基本语法是:`大类型/小类型`。

|   对应后缀   | Content-Type                 | 类型描述           |
| :----------: | ---------------------------- | ------------------ |
|    .html     | text/html                    | 超文本标记语言文本 |
|     .xml     | text/xml                     | xml文档            |
|    .xhtml    | application/xhtml+xml        | XHTML文档          |
|     .txt     | text/plain                   | 普通文本           |
|     .rtf     | application/rtf              | RTF文本            |
|     .pdf     | application/pdf              | PDF文档            |
|    .word     | application/msword           | Microsoft          |
|     .png     | image/png                    | PNG图像            |
|     .gif     | image/gif                    | GIF图形            |
|  .jpeg,.jpg  | image/jpeg                   | JPEG图形           |
|     .au      | audio/basic                  | au声音文件         |
|  mid,.midi   | audio/midi,audio/x-midi      | MIDI音乐文件       |
|  .mpg,.mpeg  | video/mpeg                   | MPEG文件           |
|     .avi     | video/x-msvideo              | AVI文件            |
|     .gz      | application/x-gzip           | GZIP文件           |
|     .tar     | application/x-tar            | TAR文件            |
| **用于下载** | **application/octet-stream** | 任意的二进制数据   |
其他参见MIME。
# 五、GET和POST区别



## (一)、GET



## (二)、POST







[request]:data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAcwAAACgCAMAAAC2RKwhAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyJpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENTNiAoV2luZG93cykiIHhtcE1NOkluc3RhbmNlSUQ9InhtcC5paWQ6MDlCMzAyRUE0QUY2MTFFOUI2QzNBQUVDMDk5MjU2NTEiIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6MDlCMzAyRUI0QUY2MTFFOUI2QzNBQUVDMDk5MjU2NTEiPiA8eG1wTU06RGVyaXZlZEZyb20gc3RSZWY6aW5zdGFuY2VJRD0ieG1wLmlpZDowOUIzMDJFODRBRjYxMUU5QjZDM0FBRUMwOTkyNTY1MSIgc3RSZWY6ZG9jdW1lbnRJRD0ieG1wLmRpZDowOUIzMDJFOTRBRjYxMUU5QjZDM0FBRUMwOTkyNTY1MSIvPiA8L3JkZjpEZXNjcmlwdGlvbj4gPC9yZGY6UkRGPiA8L3g6eG1wbWV0YT4gPD94cGFja2V0IGVuZD0iciI/Ph7nBX8AAAAMUExURZmZmWZmZszMzP////wKZ50AAAz8SURBVHja7J2LmqMgDIWT8P7vvCU3AkKrHaej3bjfzky11cJvAujhCCWXr1kgq+CrYMJ5C572pn2H+/Dx3t0VvvsVDsMkPHHZV7TvPt55MPENmHTasutUOu94uKu0iJ893uQbvNmYJcyEmTATZsJMmAnzkzAx/gnT9cwOlzBDtxqHr/UrMAnXxwsHBLAf/Er/696gvSW8nh0vrNoUVyoBFzDreiB/LwUS1O8n7IRGtI9j0GwXA8xHCWQ8hY0fTuhhV+4BJjEy+xYgX4T2wKx7BP4Ks92vKpeswubHG2E6Jd6iBUTgotfhx2uYDQdambzSuQh1l7CC2bY9PubjSV7LX3UD8/EG+6J+6gL2u4AJzFqTQ2QSrwqjZiRfN1Q46HnTnbcgBcRdMFH2ymcT7oUZTrHZ8ZYwOe08NjvM8byOgbuESe1qAkk919VAi8h87NJWdO+ptVdxztIsB2IfmY8P07CLWZrV76b8eMBrxceYn3TZwqwnC9X4l8TBBQ0J5BXMeqLwWYg70yyClI5TzuR4HUwpkq8COyjGdBSG+iuYXC2kNYztcFzNtfgwgcmnD9g2sMaBeI+PtdBfX41gpVxFL9ihBqPvQuq6gwnQ0ixISe1332YiwZM0KzmApAx6qmL5NZj1GCTUpsfDjiUXCaWwQhOlnH7m2s43xetgSmWjpsEQITDG0RCZctpoVUhlk3+CJjDB0yx/DLSHQG0Xso9tm0lDpnlUBYRmRMvaYE4i005ToYm1CiiWZw9MTrN7YVpYcFm3x8OhFUHLtKHJNKItCcf4fQ7TdojWnsm+cQqzvg01AOWch1gtteTbDlDMpXyAxx7ly0A8BSYwISTWWqOtMqTF0fRkwb6FCdokS3ON3LI+jmqt9z6YSLtheouwOF4PE6XurSh+eiKfPt5ubtvQJ5HZtaIoHbJZZNa6LA6TIkyQPtssMhmmQAnllS7XPpitdrELW92Ck5FKN85s5QZrlI7ApAMwXx2vb0d012idO92MgtnCsb6EcezVwZT+3aZLhASN8gizHgMkzdb4LQ0mJ96hqY0wNSla/wck7nUXa5hSnFBCwq6m5YfXxUuY0rkLpdsH80iafXE83Ax9tJkQrn52ohzWGleUGy4LmOSHjTfHtGK9ZzQdmpBn0gbTQn0BE8IAlcvgTctTmNgi04lhjEawnyHpPoHpp+kumHqCQAyeYzAnxxtghnNV8pBlGeT/aAOSI+PM0Hm2Lv0aZkPWtZll1QHi/h3qyIt7A3zCaKvyDKbkFmhFar2D1sBIz6gOsPtrZSNMuxiCtHOcWU8cg9ml+30w58cb20zqSqEnq8DkNfD6osHmluMAcxWZnJkpkqAdMEGSDdmFGNIxip8PC5ia6rwCsBuReZCCj0tmHaC24nBkvnM579XxIszWjQ3fXq6UyIWQVqJdkQkLmDiHCWh1BNZzwgjTm8VtdBvM0vfVGStCXmjPuyYJM2EmzISZMBNmwkyYCTNhJsyE+Sswc7nqchmYTyLlCgX/adBQud6SMBNmwkyYCfPjMGHy1+T1+sU5MGG62zdg0pOXXw2TdRVehXzXUuc4AurNW35ht4zA3ginwyS0u6aHYJJPhtQKB4x3E6P2vYfJ0kWIGju+O+boOu6673cTwocikyUVPrNUgAF1N6EbZST/lIgYT4XpGo9jMKGTaPjtddF6qcRjClNkU9i4t7uQtXRoKqouMvHSaTaEm0Sp3X737bJJdTotgrGPzp/ClJkC+xPtANOlqxWm0wLVjW9hsn6E5Rm6E4iVVNCkRZs0i5eFKTEILcuKsAraDAzZKNNBVCkkohIcQuinMIOA9ShMUTq72twErSYap0WatbkYfCq4FpoYI6sY6F4wsQpOydtP0pknPWyNTFIBuCqlx1pf9jJ2BqbqjAjegEkubS0yG4AiAgxUBpj+MXmTz4Qy2RTdCibnGk2xNpkE3eShdYBUB+0Etz2g7iDHYRJ4+P8IJrp4X6YRiMB7BrMWvH1MxFOuzyGdMXUjmNBdVVZahMOELBDW3AECo8wVf26b+T7MOKmnIVHl8UKfLGJzkFYTOgkjigS9a2tvEZmqPbRujfRYnVlrTUGGJnYOgA4mrgGTSpy7Qzp3kZ7rk5tYdpjGAyKNuyNMG4o0mK1GYyB6blUB+hOY9GGYrZrBp+xZ1oQnMGVqaWAUNa1wQ5jgOuQwd8gUzy0ylWKjDE/S7DsdoHNg9mPG52JzIBVQz2HeL812MajDS2obS4AZqOtFAzg3MnWy0+GLBjj4DQwwe+OrTp+sdgSojKgTKLdXd+kAwWL0Pm4G/xsWl2bPuTb7zoV22vwB3RpajDPdTSD4E3R7gnK/DlDeNTl0oT1hJsyEmTATZsJMmAkzYSbMhHlpmF++XBUm5XJ8psJVYZZcvifNJpqEmTATZsJMmAkzYX4W5sbgWcxvJ5+ChHl1mJWdWVHLimDV3KmPePYDHn9CVML8XJoFjUx/tgE0E0txDdbYFce6ApAwLwUzhptHpjqKkug27X3NQZ9lDXBytr0sTET29yPEi8PsZoJBpzdhaVQJQqJmug46K+s/gkmdgeNFYcZMq9NSbCoZS9LsTzOFV+ASmfRfwLxNmh0ikzA8xrT+Ebc2Q1OfkZQwL9pmMi60Kbokz1uB+BAQjUR5bkXCvPjQpJno88Rt6uyBydyB9YlIAJi92UsPTcSCv6VZE+rqrHzp33pk5kWD68LUjmybJE+tlbToLS7PTphXh/mHS8JMmAkzYSbMhJkwE2bCzOXeMCGXw0sq2lPRfleY6wyVc00SZsJMmAkzYSbMhHk5mMHuBzZWkwvPIPoVHyD4kUe7337q18ACJjvKzjU5E/+gW/jNjh7t7p0HG4tSIOo92k91gjaP9sN2azjMoQy+6/yMZywrv1lmRL7BH/Auq/F2MMUL2e0OWawreMkEgoTBohTjW8/1aAeiI1bQDjN655EJzMWjXa2QaQWTTa+HM6CINJIrgm4Fk62DPTJp4tEuv9WJ32CquffZHu2dcfEBmHIOFsPifrMVGNDaox1c2tp5X7KzacEberTTwqNdPCah82hH92hnE27AM40Q6Rwn6ODRLjM8Bg+83tYbADw0/bodp9+hQb1HmxkeiQDmeL/yaFcjS2lnx1pfdhn3JtmzbL3DoxAML6xg+seosyiVpBsV6LfyaLcuD3Ka7T3awZ+e4B7t8NSj/Y3IhNM82sU82Mj15hIRpjxvwX36I8zqAt106neBqT0cyS/NPFg6tN0mbdEsUH+hA3S+RzvKkxTWHu1qlAyDRzvd26MdMNp6bz3agcwt2179jkc7vvkojNabbV+0SM8Hn9p6t9cR5p092s3f2WCSO0CHQUsYXfrQdAWznx7zSVtvsWaPIUrlhUc7bGCqE/TdYALFR0e1yBy3ovVr3db7qUf7e5FJJ4wzSykbW29c2npLPwfM1rt0tt4F8F69WTjBXvuC12bp3WuztHx1kwe75YX2AxfaL/7IxYR58K5JwkyYCTNhJsyEmTATZsL8RphfvpSLwix5Un/HkjATZsJMmAkzYebyE5g0/C6z1wnzyjDDVFJdEeSGWMrAPd7Xhc7lTxQz4OQGQVfC/ATM7oarOmqKJqbo9GqEhbCN3xF0bbaBb2Pzj4T5aZjqBaZC0yb9rs8OrzbUS/1wnUrMlKizNwYVNmJG5udhoigRB/lwldIIpMWzt1n1L/6bxI9VV0WjeukubtLn8sswS7OfVmEiRQS4mnMj+uHi77EQdjFxptk/hYk66YaUrLSZU5hVmGcw0bKtkRUVa8L8PEwIs2eC0q1K2oie6IdFDE6DfLhM5MMJ85ORCSNMHmq80A9zr5fIe0/2ya18OGF+fmiiU2m5K6NwgF7oh8nHNZ18GBLmX48z+zGjpc01TGyTrAaYmWb/5AqQa8jnMPtHV3VicMFlWnDqtODU+8onzE/ADA+XC4P+uIoW40zSDeQ/oNtTXs77OMyDS15oT5gJM2HmkjATZsJMmAnzAjBz+SIRdDqu/8Sj/WIwKZfDS8JMmAkzYSbMhJlLwvx/YW7u81Iwzax/hpuB4lKXMK8LE4sI32wN+XWYKosC54ztdm/CvGqatbvyPj+HBWx8L59D0dSqqPf8E+a120yZyFFKoaibEqdsNpbUt4gBfsK8KkwQQbFKVNW0X8NVHbLb/AD4Zau9hPnDyLTJFyZN5Olv3Omp83P4n8KUpwcAUMK8Mky0uXKcaKGb3KgwWUUOgxoqYV4PJt+qiHnXnr9QDZALaGcIWf2I2WZeGibY3Dkx7a/T5fxBN9jkqlgEZPZmLz3OLNr1Ea9oTqZoTv3gdtKoPdmEeeHIlC5ql2a1O1T7OvV/nVfFQnBC6HXkCfN648wu4LoLffb4BbF8kGkBCTMvtCfMhJlLwkyYCTNhJsyEmTC/HmZKmr9GBF1yqsH3TE/I5YsmDuXyNcs/AQYAoh6oJn6LE00AAAAASUVORK5CYII=
[responce]:data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAcwAAACgCAMAAAC2RKwhAAAAGXRFWHRTb2Z0d2FyZQBBZG9iZSBJbWFnZVJlYWR5ccllPAAAAyJpVFh0WE1MOmNvbS5hZG9iZS54bXAAAAAAADw/eHBhY2tldCBiZWdpbj0i77u/IiBpZD0iVzVNME1wQ2VoaUh6cmVTek5UY3prYzlkIj8+IDx4OnhtcG1ldGEgeG1sbnM6eD0iYWRvYmU6bnM6bWV0YS8iIHg6eG1wdGs9IkFkb2JlIFhNUCBDb3JlIDUuMy1jMDExIDY2LjE0NTY2MSwgMjAxMi8wMi8wNi0xNDo1NjoyNyAgICAgICAgIj4gPHJkZjpSREYgeG1sbnM6cmRmPSJodHRwOi8vd3d3LnczLm9yZy8xOTk5LzAyLzIyLXJkZi1zeW50YXgtbnMjIj4gPHJkZjpEZXNjcmlwdGlvbiByZGY6YWJvdXQ9IiIgeG1sbnM6eG1wPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvIiB4bWxuczp4bXBNTT0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL21tLyIgeG1sbnM6c3RSZWY9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC9zVHlwZS9SZXNvdXJjZVJlZiMiIHhtcDpDcmVhdG9yVG9vbD0iQWRvYmUgUGhvdG9zaG9wIENTNiAoV2luZG93cykiIHhtcE1NOkluc3RhbmNlSUQ9InhtcC5paWQ6MDlCMzAyRUE0QUY2MTFFOUI2QzNBQUVDMDk5MjU2NTEiIHhtcE1NOkRvY3VtZW50SUQ9InhtcC5kaWQ6MDlCMzAyRUI0QUY2MTFFOUI2QzNBQUVDMDk5MjU2NTEiPiA8eG1wTU06RGVyaXZlZEZyb20gc3RSZWY6aW5zdGFuY2VJRD0ieG1wLmlpZDowOUIzMDJFODRBRjYxMUU5QjZDM0FBRUMwOTkyNTY1MSIgc3RSZWY6ZG9jdW1lbnRJRD0ieG1wLmRpZDowOUIzMDJFOTRBRjYxMUU5QjZDM0FBRUMwOTkyNTY1MSIvPiA8L3JkZjpEZXNjcmlwdGlvbj4gPC9yZGY6UkRGPiA8L3g6eG1wbWV0YT4gPD94cGFja2V0IGVuZD0iciI/Ph7nBX8AAAAMUExURZmZmWZmZszMzP////wKZ50AAAz8SURBVHja7J2LmqMgDIWT8P7vvCU3AkKrHaej3bjfzky11cJvAujhCCWXr1kgq+CrYMJ5C572pn2H+/Dx3t0VvvsVDsMkPHHZV7TvPt55MPENmHTasutUOu94uKu0iJ893uQbvNmYJcyEmTATZsJMmAnzkzAx/gnT9cwOlzBDtxqHr/UrMAnXxwsHBLAf/Er/696gvSW8nh0vrNoUVyoBFzDreiB/LwUS1O8n7IRGtI9j0GwXA8xHCWQ8hY0fTuhhV+4BJjEy+xYgX4T2wKx7BP4Ks92vKpeswubHG2E6Jd6iBUTgotfhx2uYDQdambzSuQh1l7CC2bY9PubjSV7LX3UD8/EG+6J+6gL2u4AJzFqTQ2QSrwqjZiRfN1Q46HnTnbcgBcRdMFH2ymcT7oUZTrHZ8ZYwOe08NjvM8byOgbuESe1qAkk919VAi8h87NJWdO+ptVdxztIsB2IfmY8P07CLWZrV76b8eMBrxceYn3TZwqwnC9X4l8TBBQ0J5BXMeqLwWYg70yyClI5TzuR4HUwpkq8COyjGdBSG+iuYXC2kNYztcFzNtfgwgcmnD9g2sMaBeI+PtdBfX41gpVxFL9ihBqPvQuq6gwnQ0ixISe1332YiwZM0KzmApAx6qmL5NZj1GCTUpsfDjiUXCaWwQhOlnH7m2s43xetgSmWjpsEQITDG0RCZctpoVUhlk3+CJjDB0yx/DLSHQG0Xso9tm0lDpnlUBYRmRMvaYE4i005ToYm1CiiWZw9MTrN7YVpYcFm3x8OhFUHLtKHJNKItCcf4fQ7TdojWnsm+cQqzvg01AOWch1gtteTbDlDMpXyAxx7ly0A8BSYwISTWWqOtMqTF0fRkwb6FCdokS3ON3LI+jmqt9z6YSLtheouwOF4PE6XurSh+eiKfPt5ubtvQJ5HZtaIoHbJZZNa6LA6TIkyQPtssMhmmQAnllS7XPpitdrELW92Ck5FKN85s5QZrlI7ApAMwXx2vb0d012idO92MgtnCsb6EcezVwZT+3aZLhASN8gizHgMkzdb4LQ0mJ96hqY0wNSla/wck7nUXa5hSnFBCwq6m5YfXxUuY0rkLpdsH80iafXE83Ax9tJkQrn52ohzWGleUGy4LmOSHjTfHtGK9ZzQdmpBn0gbTQn0BE8IAlcvgTctTmNgi04lhjEawnyHpPoHpp+kumHqCQAyeYzAnxxtghnNV8pBlGeT/aAOSI+PM0Hm2Lv0aZkPWtZll1QHi/h3qyIt7A3zCaKvyDKbkFmhFar2D1sBIz6gOsPtrZSNMuxiCtHOcWU8cg9ml+30w58cb20zqSqEnq8DkNfD6osHmluMAcxWZnJkpkqAdMEGSDdmFGNIxip8PC5ia6rwCsBuReZCCj0tmHaC24nBkvnM579XxIszWjQ3fXq6UyIWQVqJdkQkLmDiHCWh1BNZzwgjTm8VtdBvM0vfVGStCXmjPuyYJM2EmzISZMBNmwkyYCTNhJsyE+Sswc7nqchmYTyLlCgX/adBQud6SMBNmwkyYCfPjMGHy1+T1+sU5MGG62zdg0pOXXw2TdRVehXzXUuc4AurNW35ht4zA3ginwyS0u6aHYJJPhtQKB4x3E6P2vYfJ0kWIGju+O+boOu6673cTwocikyUVPrNUgAF1N6EbZST/lIgYT4XpGo9jMKGTaPjtddF6qcRjClNkU9i4t7uQtXRoKqouMvHSaTaEm0Sp3X737bJJdTotgrGPzp/ClJkC+xPtANOlqxWm0wLVjW9hsn6E5Rm6E4iVVNCkRZs0i5eFKTEILcuKsAraDAzZKNNBVCkkohIcQuinMIOA9ShMUTq72twErSYap0WatbkYfCq4FpoYI6sY6F4wsQpOydtP0pknPWyNTFIBuCqlx1pf9jJ2BqbqjAjegEkubS0yG4AiAgxUBpj+MXmTz4Qy2RTdCibnGk2xNpkE3eShdYBUB+0Etz2g7iDHYRJ4+P8IJrp4X6YRiMB7BrMWvH1MxFOuzyGdMXUjmNBdVVZahMOELBDW3AECo8wVf26b+T7MOKmnIVHl8UKfLGJzkFYTOgkjigS9a2tvEZmqPbRujfRYnVlrTUGGJnYOgA4mrgGTSpy7Qzp3kZ7rk5tYdpjGAyKNuyNMG4o0mK1GYyB6blUB+hOY9GGYrZrBp+xZ1oQnMGVqaWAUNa1wQ5jgOuQwd8gUzy0ylWKjDE/S7DsdoHNg9mPG52JzIBVQz2HeL812MajDS2obS4AZqOtFAzg3MnWy0+GLBjj4DQwwe+OrTp+sdgSojKgTKLdXd+kAwWL0Pm4G/xsWl2bPuTb7zoV22vwB3RpajDPdTSD4E3R7gnK/DlDeNTl0oT1hJsyEmTATZsJMmAkzYSbMhHlpmF++XBUm5XJ8psJVYZZcvifNJpqEmTATZsJMmAkzYX4W5sbgWcxvJ5+ChHl1mJWdWVHLimDV3KmPePYDHn9CVML8XJoFjUx/tgE0E0txDdbYFce6ApAwLwUzhptHpjqKkug27X3NQZ9lDXBytr0sTET29yPEi8PsZoJBpzdhaVQJQqJmug46K+s/gkmdgeNFYcZMq9NSbCoZS9LsTzOFV+ASmfRfwLxNmh0ikzA8xrT+Ebc2Q1OfkZQwL9pmMi60Kbokz1uB+BAQjUR5bkXCvPjQpJno88Rt6uyBydyB9YlIAJi92UsPTcSCv6VZE+rqrHzp33pk5kWD68LUjmybJE+tlbToLS7PTphXh/mHS8JMmAkzYSbMhJkwE2bCzOXeMCGXw0sq2lPRfleY6wyVc00SZsJMmAkzYSbMhHk5mMHuBzZWkwvPIPoVHyD4kUe7337q18ACJjvKzjU5E/+gW/jNjh7t7p0HG4tSIOo92k91gjaP9sN2azjMoQy+6/yMZywrv1lmRL7BH/Auq/F2MMUL2e0OWawreMkEgoTBohTjW8/1aAeiI1bQDjN655EJzMWjXa2QaQWTTa+HM6CINJIrgm4Fk62DPTJp4tEuv9WJ32CquffZHu2dcfEBmHIOFsPifrMVGNDaox1c2tp5X7KzacEberTTwqNdPCah82hH92hnE27AM40Q6Rwn6ODRLjM8Bg+83tYbADw0/bodp9+hQb1HmxkeiQDmeL/yaFcjS2lnx1pfdhn3JtmzbL3DoxAML6xg+seosyiVpBsV6LfyaLcuD3Ka7T3awZ+e4B7t8NSj/Y3IhNM82sU82Mj15hIRpjxvwX36I8zqAt106neBqT0cyS/NPFg6tN0mbdEsUH+hA3S+RzvKkxTWHu1qlAyDRzvd26MdMNp6bz3agcwt2179jkc7vvkojNabbV+0SM8Hn9p6t9cR5p092s3f2WCSO0CHQUsYXfrQdAWznx7zSVtvsWaPIUrlhUc7bGCqE/TdYALFR0e1yBy3ovVr3db7qUf7e5FJJ4wzSykbW29c2npLPwfM1rt0tt4F8F69WTjBXvuC12bp3WuztHx1kwe75YX2AxfaL/7IxYR58K5JwkyYCTNhJsyEmTATZsL8RphfvpSLwix5Un/HkjATZsJMmAkzYebyE5g0/C6z1wnzyjDDVFJdEeSGWMrAPd7Xhc7lTxQz4OQGQVfC/ATM7oarOmqKJqbo9GqEhbCN3xF0bbaBb2Pzj4T5aZjqBaZC0yb9rs8OrzbUS/1wnUrMlKizNwYVNmJG5udhoigRB/lwldIIpMWzt1n1L/6bxI9VV0WjeukubtLn8sswS7OfVmEiRQS4mnMj+uHi77EQdjFxptk/hYk66YaUrLSZU5hVmGcw0bKtkRUVa8L8PEwIs2eC0q1K2oie6IdFDE6DfLhM5MMJ85ORCSNMHmq80A9zr5fIe0/2ya18OGF+fmiiU2m5K6NwgF7oh8nHNZ18GBLmX48z+zGjpc01TGyTrAaYmWb/5AqQa8jnMPtHV3VicMFlWnDqtODU+8onzE/ADA+XC4P+uIoW40zSDeQ/oNtTXs77OMyDS15oT5gJM2HmkjATZsJMmAnzAjBz+SIRdDqu/8Sj/WIwKZfDS8JMmAkzYSbMhJlLwvx/YW7u81Iwzax/hpuB4lKXMK8LE4sI32wN+XWYKosC54ztdm/CvGqatbvyPj+HBWx8L59D0dSqqPf8E+a120yZyFFKoaibEqdsNpbUt4gBfsK8KkwQQbFKVNW0X8NVHbLb/AD4Zau9hPnDyLTJFyZN5Olv3Omp83P4n8KUpwcAUMK8Mky0uXKcaKGb3KgwWUUOgxoqYV4PJt+qiHnXnr9QDZALaGcIWf2I2WZeGibY3Dkx7a/T5fxBN9jkqlgEZPZmLz3OLNr1Ea9oTqZoTv3gdtKoPdmEeeHIlC5ql2a1O1T7OvV/nVfFQnBC6HXkCfN648wu4LoLffb4BbF8kGkBCTMvtCfMhJlLwkyYCTNhJsyEmTC/HmZKmr9GBF1yqsH3TE/I5YsmDuXyNcs/AQYAoh6oJn6LE00AAAAASUVORK5CYII=



[https://tools.ietf.org/html/rfc7230]: