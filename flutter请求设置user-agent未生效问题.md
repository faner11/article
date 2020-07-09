最近在玩 Flutter，遇到一个 http 请求设置 user-agent 的无效的坑，是 Dart 原生 HttpClient 的 bug，所以几乎所有相关的 http 库都有这个问题，比如 dio，http 等等，今天将解决方案分享出来，希望官方团队早日修复 bug。

## 未生效案例

```dart
import 'dart:io';

void ioHttp(String url, String ua) async {
  final uri = Uri.parse(url);
  final httpClient = HttpClient();
  final request = await httpClient.getUrl(uri);
  // 设置ua
  request.headers.set(HttpHeaders.userAgentHeader, ua);
  final response = await request.close();
  final responseBody = await response.transform(Utf8Decoder()).join();
  print(responseBody.length);
}
```

以上是标准的设置`user-agent`的写法，网上几乎所有的教程都是这样写，这样写请求的`Headers`会有`user-agent`字段，值也是你设置的 ua 信息，一切都会那么正常，可是这种写法有时返回就是错的。

## 生效案例

```dart
import 'dart:io';

void ioHttp(String url, String ua) async {
  final uri = Uri.parse(url);
  final httpClient = HttpClient();
  // add
  httpClient.userAgent = null;
  final request = await httpClient.getUrl(uri);
   // 设置ua
  request.headers.set(HttpHeaders.userAgentHeader, ua);
  final response = await request.close();
  final responseBody = await response.transform(Utf8Decoder()).join();
  print(responseBody.length);
}
```

当我在代码中加上`httpClient.userAgent = null`的时候就一切正常。

## 原因

当请求的链接做了 301 或 302 之类的跳转时,跳转后的链接 UA 信息为`Dart/<version> (dart:io)`,UA 不对就拿不到正常的返回了，但是设置`httpClient.userAgent = null`后，跳转后的链接 UA 信息又是正常的，好坑。

ps： 如果链接没有跳转是不会触发这个 bug 的。

## 在 dio 中使用

如果使用 Dart 原生的`HttpClient`,我们将失去 http 库很多好用的功能，http 库的本质是原生`HttpClient`的一些功能封装，万幸我们可以通过覆盖 http 库的`HttpClient`来比慢这个 bug，以 dio 为例：

```dart
final dio = Dio();
// 核心 dio.httpClientAdapter
(dio.httpClientAdapter as DefaultHttpClientAdapter).onHttpClientCreate =
    (HttpClient client) {
    client.userAgent = null;
};
final req = await dio.get(url,
    options: Options(headers: {HttpHeaders.userAgentHeader: ua}));
print(req.data.toString()); //正常
```

这样我们就可以使用 dio 继续愉快的工作啦。
