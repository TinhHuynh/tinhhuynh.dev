---
title: "Flutter 3.13: làm Sticky Header dễ dàng với SliverMainAxisGroup"
excerpt: "Bài viết này hướng dẫn cách thực hiện Sticky Header animation bằng cách sử dụng SliverMainAxisGroup"
seo_title: "Hướng dẫn Sticky Header Animation với Flutter 3.13 và SliverMainAxisGroup"
seo_description: "Học cách tạo hiệu ứng Sticky Header dễ dàng trong ứng dụng Flutter bằng cách sử dụng SliverMainAxisGroup. Thúc đẩy trải nghiệm người dùng và tạo giao diện người dùng tốt hơn với Flutter 3.13."
categories:
  - Programming
  - Flutter
tags:
  - flutter
  - sticky header
  - programming
  - flutter 3.13
  - ui/ux
  - animation
---

### I. Mở đầu

Mới đây [Flutter 3.10](https://medium.com/flutter/whats-new-in-flutter-3-13-479d9b11df4d) mới ra mắt, mang đến nhiều sự nâng cấp cũng như cải thiện đáng kể như render engine `Impeller` cho iOS, support foldable, dialog + Material widgetS,...và cho ra mắt 1 vài widget sliver hữu ích như `SliverMainAxisGroup`, `SliverCrossAxisGroup`,...Đối với `SliverMainAxisGroup`, chắc chắc lập trình viên sẽ tiết kiệm nhiều thời gian trong việc viết sticky header animation để tăng trải nghiệm người dùng.

<figure>
  <img src="{{ site.url }}{{ site.baseurl }}\assets\gifs\2023-08-27-group-main-axis-sliver.gif" style="width:250px;height:450px;">
</figure>

### 2. SliverMainAxisGroup

Cho những ai chưa biết thì, để hiển thị UI dạng scroll list thì ngoài `ListView`, Flutter còn cung cấp `CustomScrollView` - cho phép hiển thị list content cũng như scroll animation phức tạp trên scroll list.

<figure>
  <video autoplay loop>
    <source src="https://flutter.github.io/assets-for-api-docs/assets/widgets/custom_scroll_view.mp4" type="video/mp4" />
  </video>
  <figcaption>Nguồn: https://api.flutter.dev/flutter/widgets/CustomScrollView-class.html</figcaption>
</figure>

Các widget con của `CustomScrollView` được gọi là sliver và có prefix `Sliver` ở tên class. Ví dụ:

```dart
SliverPadding ~ Padding
SliverList ~ ListView
SliverBoxToAdapter ~ Single Widget
SliverAppBar ~ AppBar
SliverGrid ~ GridView
```

Và giờ đây chúng ta có `SliverMainAxisGroup` - sliver này sắp xếp 1 nhóm widget con tuần tự theo 1 chiều dọc, bạn có thể hiểu nó tương tự như `Column`.

### 3. Sticky header animation

Ok, bây giờ chúng ta hãy tiến hành thực hiện sticky header animation. Như ở gif demo, chúng
ta thấy sẽ có nhiều header và các item đi theo từng header, mình sẽ gọi group 1 header và các item như vậy là 1 section. Như vậy, mình sẽ viết 1 class widget cho section, sử dụng `SliverMainAxisGroup` chứa `SliverAppBar` và `SliverList`.

- `SliverAppBar` được sử dụng cho header, và có thuộc tính `pinned: true` nhằm khi scroll các item thì header sẽ pin ở trên đầu `CustomScrollView`, khi các item bị scroll hết đi, header mới đc scroll theo.
- `SliverList` được sử dụng để hiển thị item đi theo header.

```dart
class SectionWidget extends StatelessWidget {
  final String header;
  final Widget Function(BuildContext, int) itemBuilder;
  final int itemCount;

  const SectionWidget(
      {super.key,
      required this.header,
      required this.itemBuilder,
      required this.itemCount});

  @override
  Widget build(BuildContext context) {
    return SliverMainAxisGroup(
      slivers: <Widget>[
        SliverAppBar(
          title: Text(
                header,
                style: const TextStyle(color: Colors.black),
              ),
          expandedHeight: 70.0,
          backgroundColor: Colors.white,
          elevation: 0.0,
          pinned: true,
        ),
        SliverList.builder(
          itemBuilder: itemBuilder,
          itemCount: itemCount,
        ),
      ],
    );
  }
}
```

Tiếp theo, chúng ta sẽ tạo vài section và hiển thị chúng lên `CustomScrollView`.

```dart
class SliverMainAxisGroupExample extends StatelessWidget {
  const SliverMainAxisGroupExample({super.key});

  @override
  Widget build(BuildContext context) {
    return CustomScrollView(
      slivers: <Widget>[
        SectionWidget(
            header: 'Header 1',
            itemBuilder: (context, index) {
              return Container(
                color: index.isEven ? Colors.amber[300] : Colors.blue[300],
                height: 100,
                padding: const EdgeInsets.all(16),
                child:
                    Text('Item $index', style: const TextStyle(fontSize: 20)),
              );
            },
            itemCount: 10),
        SectionWidget(
            header: 'Header 2',
            itemBuilder: (context, index) {
              return Container(
                color: index.isEven ? Colors.amber[300] : Colors.blue[300],
                height: 100,
                padding: const EdgeInsets.all(16),
                child:
                    Text('Item $index', style: const TextStyle(fontSize: 20)),
              );
            },
            itemCount: 10),
      ],
    );
  }
}
```

Kết quả:

<figure>
  <img src="{{ site.url }}{{ site.baseurl }}\assets\gifs\2023-08-27-group-main-axis-sliver.gif" style="width:250px;height:450px;">
</figure>

### 4. Lời Kết
Với phiên bản Flutter 3.13 và class `SliverMainAxisGroup`, việc thực hiện sticky header animation trở nên dễ dàng hơn bao giờ hết. Đây là một trong những ví dụ cho việc đội ngũ và cộng đồng Flutter đang cố gắng như thế nào để thúc đẩy Flutter trở nên trưởng thành và toàn diện hơn; mang đến trải nghiệm lập trình thú vị và tuyệt vời; giúp lập trình viên tạo ra các ứng dụng chất lượng hơn. Hãy cùng mình chờ xem những tính năng thú vị nào sẽ xuất hiện ở những phiên bản Flutter trong tương lai nhé.


### Tham khảo

- [SliverCrossAxisGroup](https://master-api.flutter.dev/flutter/widgets/SliverMainAxisGroup-class.html)
- [CustomScrollView](https://api.flutter.dev/flutter/widgets/CustomScrollView-class.html)
- [Flutter 3.13](https://docs.flutter.dev/release/release-notes/release-notes-3.13.0)
