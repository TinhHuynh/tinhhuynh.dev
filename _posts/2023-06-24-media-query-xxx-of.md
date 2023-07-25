---
title: "Flutter 3.10: Sử dụng MediaQuery 1 cách hiệu quả"
excerpt: "Update hữu ích từ MediaQuery trong Flutter 3.10. Bài viết này sẽ hướng dẫn bạn cách sử dụng MediaQuery một cách thông minh và giảm thiểu việc rebuild không cần thiết, giúp ứng dụng của bạn chạy mượt mà hơn bao giờ hết."
categories:
  - Flutter
  - Progamming
tags:
  - flutter
  - programming
  - ui responsive
---

Khi phát triển ứng dụng trên Flutter, sẽ không tránh khỏi những trường hợp mà ta cần biết thông tin về màn hình của thiết bị như: kích thước, mật độ điểm ảnh, orientation, dark mode,...Và Flutter cung cấp class `MediaQuery` để hỗ trợ lập trình viên trong việc này. 

```dart
 // kích thước màn hình
 MediaQuery.of(context).size
 // Mật độ điểm ảnh
 MediaQuery.of(context).devicePixelRatio
 // Orientation
 MediaQuery.of(context).orientation
 ...
```

Tuy nhiên `MediaQuery` có một hạn chế, khi widget gọi hàm `.of(context).xxx`, nghĩa là widget đang lắng nghe sự thay đổi các data tróng `MediaQuery`, và nó sẽ rebuild UI khi data bất kì của `MediaQuery` thay đổi. Bạn đã hình dung ra được vấn đề chưa? Nếu giả sử muốn một widget render UI dựa theo `size`, nhưng nếu `platformBrightness`, `devicePixelRatio`, hay bất kì data từ `MediaQuery` thay đổi thì widget của bạn cũng bị rebuild. 

Đây là example tái hiện:
```dart
class MyHomePage extends StatelessWidget {
  const MyHomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
        body: Center(
            child: SizeText()));
  }
}

class SizeText extends StatelessWidget {
  const SizeText({
    super.key,
  });

  @override
  Widget build(BuildContext context) {
    // Notice here :)
    print('build SizeText');
    return Text(MediaQuery.of(context).size.height.toString());
  }
}

```

<figure>
  <img src="{{ site.url }}{{ site.baseurl }}\assets\gifs\2023-06-24-media-query-xxx-of-gif-1.gif">
</figure>

Như bạn thấy trong gif, mình thay đổi light mode/dark mode (`platformBrightness`) thì `SizeText` bị rebuild mặc dù nó chỉ sử dụng thuộc tính `size` thôi. Từ ví dụ này chúng ta thấy rằng `MediaQuery` đang gây ra việc rebuild UI ko cần thiết như thế nào, đặc biệt tệ hơn nếu đó chúng ta widget với widget tree đồ sộ và phức tạp.

Để giải quyết vấn đề này, `MediaQuery` ơ Flutter phiên bản **3.10** đã cho ra mắt những method mới:
```dart
// kích thước màn hình
 MediaQuery.sizeOf(context)
 // Mật độ điểm ảnh
 MediaQuery.devicePixelRatioOf(context)
 // Orientation
 MediaQuery.orientationOf(context)
 ...
```

Với những method này, khi widget lắng nghe 1 data cụ thể nào từ `MediaQuery` thì khi data đó update thì widget sẽ được rebuild, các data khác có update hay không thì widget mặc kệ. Điều này giảm thiểu được việc rebuild UI không cần thiết. Hãy quay trở lại example.

```dart
class MyHomePage extends StatelessWidget {
  const MyHomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return const Scaffold(
        body: Center(
            child: SizeText()));
  }
}

class SizeText extends StatelessWidget {
  const SizeText({
    super.key,
  });

  @override
  Widget build(BuildContext context) {
    // Notice here :)
    print('build SizeText');
    return Text(MediaQuery.sizeOf(context).height.toString());
  }
}

```
<figure>
  <img src="{{ site.url }}{{ site.baseurl }}\assets\gifs\2023-06-24-media-query-xxx-of-gif-2.gif">
</figure>


Okay! widget không bị rebuild sau khi sử dụng `MediaQuyery.sizeOf(context)` 😄. À nếu bạn replace method cũ sang method mới ở trong code thì nên **hot restart** thay vì hot load nhé, vì hot load thì cái đăng kí lắng nghe `MediaQuery` cũ của widget vẫn còn, và widget sẽ vẫn bị rebuild. Mình đã tốn gần 1-2 tiếng để nhận ra điều này 😢

### Cheatsheet
<figure>
  <img src="{{ site.url }}{{ site.baseurl }}\assets\images\2023-06-24-media-query-xxx-of-cheat-sheet.jpg">
  <figcaption>Nguồn: Facebook/Learnfluttertogether</figcaption>
</figure>

### Bonus
Khi mình tìm hiểu về `MediaQuery` thì nhận ra là class này được implement bằng `InheritedWidget` - một chủ đề yêu thích của mình. Các thư viện state management phổ biến như `Bloc`, `Provider` đều base từ `InheritedWidget`. Chưa kể trong cheatsheet trên có đề cập `InheritedModel` - giúp các widget có thể lắng nghe một phần data cụ thể từ `InheritedWidget`. Chắc chắc mình sẽ có bài viết về 2 class này. Hãy chờ nhé 😄

### Kết luận

Well, đó là tất cả những gì mình muốn truyền tải trong bài viết này. Việc sử dụng những method mới của `MediaQuery` trong các dự án Flutter của bạn có thẻ giúp việc render UI hiệu quả hơn, performance sẽ được cải thiện hơn, mang lại trải nghiệm người dụng mượt mà và nhanh nhạy hơn. 

Happy coding !!! 


### Tham khảo

* [Learnfluttertogether](https://www.facebook.com/flutterwithrehan/posts/pfbid0A9ArxYJeT682fAyoXLe8VY22ezDE63YRRFtzbgMUP7zz9UUeDH7YFW3UnkurXqsfl)
* [MediaQuery class](https://api.flutter.dev/flutter/widgets/MediaQuery-class.html)
* [The most important Flutter 3.10 feature that nobody talks about](https://medium.com/itnext/the-most-important-flutter-3-10-feature-that-nobody-talks-about-1cc575a6063f)