---
title: "Flutter: InheritedWidget và InheritedModel"
excerpt: "Cùng nhau tìm hiểu về InheritedWidget và InheritedModel - 1 trong những phương pháp state management phổ biến."
seo_title: "Flutter: InheritedWidget và InheritedModel - Phương pháp state management phổ biến"
seo_description: "Tìm hiểu về InheritedWidget và InheritedModel - 2 trong những phương pháp state management phổ biến trong lập trình Flutter. Xem cách sử dụng chúng để truyền và quản lý data hiệu quả giữa các widget con và tránh rebuild UI không cần thiết." 
categories:
  - Programming
  - Flutter
tags:
  - flutter
  - programming
  - inherited widget
  - inherited model
  - state management
  - provider
  - bloc
  - cubit
---
### I. Mở đầu

Khi phát triển ứng dụng Flutter, ít nhiều ta đều nghe đến state management, vậy state management là gì? Cơ bản thì đó là quy trình chúng ta tổ chức và quản lý data và cách UI tương tác lên data: user input, data lấy từ server, application setting, theme,... Flutter ngày càng phát triển và các [state management framework](https://docs.flutter.dev/data-and-backend/state-mgmt/options) cũng ngày càng đa dạng hơn, InheritedWidget/InheritedModel là một trong số đó. Điều đặc biệt là 1 số state management framework phổ biến có sử dụng InheritedWidget: [Provider](https://pub.dev/packages/provider), [Bloc](https://pub.dev/packages/bloc) để đơn giản hóa việc quản lý state trong ứng dụng Flutter. Cùng nhau tìm hiểu về InheritedWidget và Inherited Model nhé!


### 2. InheritedWidget

`InheritedWidget` là 1 widget đặc biệt cho phép bạn truyền data xuống các widget con (cháu chắt chút chít gì cũng được) 1 cách hiệu quả và tự động. Các widget con nào muốn truy cập data từ `InheritedWidget` thì sẽ được rebuild UI mỗi khi data được thay đổi (thông qua method `BuildContext.dependOnInheritedWidgetOfExactType`), các widget con khác không truy cập sẽ không bị rebuild UI. Qua đây, chúng ta có thể thấy sự hiệu quả trong việc quản lý data của InheritedWidget khi nó đảm bảo tránh những việc rebuild không cần biết. 

Ngoài ra, việc sử dụng method `BuildContext.dependOnInheritedWidgetOfExactType` còn giúp đơn giản hóa việc truyền data từ cha xuống con, không cần phải pass data theo kiểu qua từng constructor này kia.

Để dễ hiểu hơn thì mình có 1 project demo, project này sẽ có 2 button để update data cho `InheritedWidget`, 2 text widget sẽ show data của `InheritedWidget`, và 1 widget không lắng nghe data từ `InheritedWidget`; data của `InheritedWidget` là `isDarkMode` và `fontSize`.  

1. Đầu tiên ta có class `ThemeModel` kế thừa `InheritedWidget`

```dart
class ThemeModel extends InheritedWidget {
  final bool isDarkMode;
  final double fontSize;

  const ThemeModel({super.key,
    required this.isDarkMode,
    required this.fontSize,
    required Widget child,
  }) : super(child: child);

  // utility method để giúp widget con truy cập ThemeModel widget gần nhất trong widget tree
  static ThemeModel? of(BuildContext context) {
    return context.dependOnInheritedWidgetOfExactType<ThemeModel>();
  }

  // check xem ThemeModel widget có cần rebuild khi data thay đổi hay không
  @override
  bool updateShouldNotify(ThemeModel oldWidget) {
    return isDarkMode != oldWidget.isDarkMode || fontSize != oldWidget.fontSize;
  }
}
```

2. 2 widget để hiển thị data từ `ThemeModel` và 1 widget ko quan tâm đến `ThemeModel`

```dart
// Hiển thị theme mode (dark mode, bright mode)
class ThemeModeText extends StatelessWidget {
  const ThemeModeText({super.key});

  @override
  Widget build(BuildContext context) {
    final themeModel = ThemeModel.of(context)!;
    final themeModeText = themeModel.isDarkMode ? 'Dark Mode' : 'Light Mode';

    final randomColor = getRandomColor();
    final textColor = themeModel.isDarkMode ? Colors.white : Colors.black;

    return Text(
      'Theme Mode: $themeModeText',
      style: TextStyle(fontSize: 18, color: textColor, backgroundColor: randomColor),
    );
  }
}

// Hiển thị font size
class FontSizeText extends StatelessWidget {
  const FontSizeText({super.key});

  @override
  Widget build(BuildContext context) {
    final themeModel = ThemeModel.of(context)!;

    final randomColor = getRandomColor();
    final textColor = themeModel.isDarkMode ? Colors.white : Colors.black;

    return Text(
      'Font Size: ${themeModel.fontSize}',
      style: TextStyle(fontSize: 18, color: textColor, backgroundColor: randomColor),
    );
  }

}

// Cái tên nói lên tất cả :)
class IdontCareWidget extends StatelessWidget {
  const IdontCareWidget({super.key});

  @override
  Widget build(BuildContext context) {
    return Text(
      'I don\'t care',
      style: TextStyle(fontSize: 18, backgroundColor: getRandomColor()),
    );
  }
}

Color getRandomColor() {
  final random = Random();
  return Color.fromRGBO(random.nextInt(256), random.nextInt(256), random.nextInt(256), 1);
}
``` 

Trong đoạn code trên, ta thấy `ThemeModeText`, `FontSizeText` gọi method `ThemeModel.of(context)` cung cấp bởi class `ThemeModel`. Method này sẽ gọi `context.dependOnInheritedWidgetOfExactType<ThemeModel>()` để truy cập `ThemeModel` gần nhất trong widget tree, đồng thời sử dụng data cũng như rebuild UI khi data của `ThemeModel` thay đổi.

Ngoài ra, 3 widget này sẽ update background color 1 cách random mỗi khi rebuild để tiện cho việc hình dung.

3. cuối cùng UI tổng thể màn hình

```dart
class ThemeHomePage extends StatefulWidget {
  const ThemeHomePage({Key? key}) : super(key: key);

  @override
  State<ThemeHomePage> createState() => _ThemeHomePageState();
}

class _ThemeHomePageState extends State<ThemeHomePage> {
  bool isDarkMode = false;
  double fontSize = 16.0;

  @override
  Widget build(BuildContext context) {
    return ThemeModel(
      isDarkMode: isDarkMode,
      fontSize: fontSize,
      child: Scaffold(
        appBar: AppBar(title: const Text('Theme Switcher App')),
        body: Column(
          mainAxisAlignment: MainAxisAlignment.spaceEvenly,
          children: [
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceEvenly,
              children: [
                // Update theme mode
                ElevatedButton(
                  onPressed: () {
                    setState(() {
                      isDarkMode = !isDarkMode;
                    });
                  },
                  child: const Text('Toggle Theme'),
                ),
                // Update font size
                ElevatedButton(
                  onPressed: () {
                    setState(() {
                      fontSize = fontSize == 16.0 ? 20.0 : 16.0;
                    });
                  },
                  child: const Text('Toggle Font Size'),
                ),
              ],
            ),
            const ThemeModeText(),
            const FontSizeText(),
            const IdontCareWidget()
          ],
        ),
      ),
    );
  }
}


```

Ok hãy chạy ứng dụng lên thử nhé!
<figure>
  <img src="{{ site.url }}{{ site.baseurl }}\assets\gifs\2023-08-05-inherited-widget-model-1.gif" style="width:250px;height:450px;">
</figure>

Như bạn thấy trong gif, mỗi lần mình click vào 2 button để update data cho `ThemeModel` thì `ThemeModeText` và `FontSizeText` sẽ được rebuild (background đổi màu), trong khi đó `IdontCareWidget` không thay đổi vì không phụ thuộc vào `ThemeModel` để render UI. Qua demo này, chúng ta thấy rằng việc sử dụng `InheritedWidget` mang lại những lợi ích về performance trong việc render UI của ứng dụng cũng như trải nghiệm người dùng khi mà nó notify data cho những widget con nào thực sự cần và tránh được việc rebuild UI không cần thiết.

### 3. InheritedModel

Mặc dù `InheritedWidget` đã giúp chúng ta tránh việc rebuild dư thừa, nhưng nếu bạn xem lại gif thì sẽ thấy có 1 vấn đề: Khi mình click button để update theme mode thì `FontSizeText` lại bị rebuild UI và ngược lại. Đó chính là nhược điểm của `InheritedWidget`: Khi bất kì 1 phần nào ở data của `InheritedWidget` thay đổi thì những widget con phụ thuộc vào nó sẽ rebuild UI, bất kể là data đó không liên quan tí nào đến widget con.

Vậy có cách nào có thể giúp các widget con phụ thuộc vào `InheritedWidget` có thể rebuild UI khi 1 phần cụ thể nào đó của data thay đổi không? Câu trả lời là có, đó là sử dụng `InheritedModel`.

`InheritedModel` như 1 bản mở rộng của `InheritedWidget`, cho phép widget rebuild UI khi 1 phần data tương ứng của `InheritedWidget` thay đổi, điểu này giúp cải thiện nhược điểm của `InheritedWidget`, cũng như giảm thiểu việc rebuild UI của các widget con phụ thuộc vào nó.

Quay về demo project, mình sẽ sửa lại ThemeModel:

```dart
// các phần data của ThemeModel
enum ThemeAspect { mode, fontSize }

class ThemeModel extends InheritedModel<ThemeAspect> {
  final bool isDarkMode;
  final double fontSize;

  const ThemeModel({super.key,
    required this.isDarkMode,
    required this.fontSize,
    required Widget child,
  }) : super(child: child);

  // utility method để giúp widget con truy cập ThemeModel widget gần nhất trong widget tree
  static ThemeModel? of(BuildContext context, {required ThemeAspect aspect}) {
    return InheritedModel.inheritFrom<ThemeModel>(context, aspect: aspect);
  }

  // check xem ThemeModel widget có cần rebuild khi data thay đổi hay không
  @override
  bool updateShouldNotify(ThemeModel oldWidget) {
    return isDarkMode != oldWidget.isDarkMode || fontSize != oldWidget.fontSize;
  }

  // check xem widget con phụ thuộc có cần được notify khi 1 phần data bị thay đổi hay không
  @override
  bool updateShouldNotifyDependent(ThemeModel oldWidget, Set<ThemeAspect> dependencies) {
    return (isDarkMode != oldWidget.isDarkMode && dependencies.contains(ThemeAspect.mode)) ||
        (fontSize != oldWidget.fontSize && dependencies.contains(ThemeAspect.fontSize));
  }
}
```

Trong class `InheritedModel` đã có sự thay đổi như sau:
* `ThemeAspect:` Ta tạo enum `ThemeAspect` để khai báo các phần (aspect) của data cho `InheritedModel` (`InheritedModel<ThemeAspect>`).
* `InheritedModel.inheritFrom`: để truy cập `InheritedModel`, widget con phải sử dụng method này thay vì `BuildContext.dependOnInheritedWidgetOfExactType` như trước. Ngoài ra, method này có tham số `aspect` - đây là cách để widget con thông báo cho `InheritedModel` biết rằng nó sẽ rebuild UI theo 1 phần data cụ thể nào đó thay đổi.
* `updateShouldNotifyDependent`: ta phải override thêm method cho `InheritedModel`, method này nhằm check xem widget con phụ thuộc có cần được notify khi 1 phần data bị thay đổi hay không.

Tiếp theo, ta sẽ sửa `FontSizeText`, `ThemeModeText` để phụ thuộc vào phần data cụ thể của `InheritedModel`:

```dart
// Hiển thị theme mode (dark mode, bright mode)
class ThemeModeText extends StatelessWidget {
  const ThemeModeText({super.key});

  @override
  Widget build(BuildContext context) {
    final themeModel = ThemeModel.of(context, aspect: ThemeAspect.mode)!;
    final themeModeText = themeModel.isDarkMode ? 'Dark Mode' : 'Light Mode';

    final randomColor = getRandomColor();
    final textColor = themeModel.isDarkMode ? Colors.white : Colors.black;

    return Text(
      'Theme Mode: $themeModeText',
      style: TextStyle(fontSize: 18, color: textColor, backgroundColor: randomColor),
    );
  }
}

// Hiển thị font size
class FontSizeText extends StatelessWidget {
  const FontSizeText({super.key});

  @override
  Widget build(BuildContext context) {
    final themeModel = ThemeModel.of(context, aspect: ThemeAspect.fontSize)!;

    final randomColor = getRandomColor();
    final textColor = themeModel.isDarkMode ? Colors.white : Colors.black;

    return Text(
      'Font Size: ${themeModel.fontSize}',
      style: TextStyle(fontSize: 18, color: textColor, backgroundColor: randomColor),
    );
  }
}
```

Qua đoạn code trên, ta đơn giản chỉ gọi `ThemeModel.of(context, aspect: aspect)!;` và truyền value `ThemeAspect` mà widget cần phụ thuộc vào method.

Ok, hãy chạy lại ứng dụng xem nhé!

<figure>
  <img src="{{ site.url }}{{ site.baseurl }}\assets\gifs\2023-08-05-inherited-widget-model-2.gif" style="width:250px;height:450px;">
</figure>

Như trong gif ta có thể thấy, `FontSizeText` và `ThemeModeText` rebuild UI khi phần data tương ứng thay đổi, không chung đụng gì với nhau. Vậy là ta có thể thấy `InheritedModel` hữu ích như thế nào khi vừa có lợi ích của `InheritedWidget`: pass data tới widget con 1 cách hiệu quả, và đảm bảo các widget con rebuild UI đúng theo phần data mà nó cần.


### Kết bài
Việc sử dụng `InheritedWidget` và `InheritedModel` sẽ giúp lập trình viên rất nhiều trong việc điều phối data xuống các widget nào thực sự cần một cần hiệu quả và đơn giản, ngoài ra giảm thiểu đến mức tối đa việc rebuild UI không cần thiết, cải thiện performance của ứng dụng và mang lại trải nghiệm người user 1 cách tốt nhất. Trong dự án, để đơn giản hóa việc sử dụng `InheritedWidget` và `InheritedModel``, mình khuyến khích các bạn có thể sử dụng package [Provider](https://pub.dev/packages/provider) hoặc [Bloc](https://pub.dev/packages/bloc),...để tránh boilerplate code, tiếp kiệm thời giản công sức,cững như tận dụng những tính năng khác mà các package đó mang lại.

Happy coding !!!

[Full Source Code](https://github.com/TinhHuynh/inherited_demo)

### Tham khảo

* [InheritedWidget class](https://api.flutter.dev/flutter/widgets/InheritedWidget-class.html)
* [InheritedModel class](https://api.flutter.dev/flutter/widgets/InheritedModel-class.html)

