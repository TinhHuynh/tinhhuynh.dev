---
title: "Flutter Test: Generate html từ file lcov.ico ở Windows (chạy được trên Android Studio/VS Code terminal)"
excerpt: "Bài viết này hướng dẫn cách generate html từ file lcov.ico được sinh ra từ flutter test --coverage. Có thể chạy được trên terminal của Android Studio và VS code"
seo_title: "Flutter Test: Generate html từ file lcov.ico ở Windows (chạy được trên Android Studio/VS Code terminal)"
seo_description: "Bài viết này hướng dẫn cách generate html từ file lcov.ico được sinh ra từ flutter test --coverage. Có thể chạy được trên terminal của Android Studio và VS code"
categories:
  - Programming
  - Flutter
tags:
  - flutter
  - testing
  - programming
  - windows
  - android studio
  - visual studio code
  - vs code
  - command line
---

## I. Mở đầu

Khi sử dụng command line `flutter test --coverage` để tạo file coverage report, Flutter sẽ sinh ra file lcov.ico cho chúng ta, và việc cần làm là làm sao để đọc được file này. Câu trả lời là chúng ta sẽ covert file sang html để xem trên trình duyệt. Ở Linux và MacOS thì việc này khá dễ dàng, nhưng trên Windows thì hơi chật vật tí, mình tìm kiếm solution trên mạng thì không thành công hoặc solution không hướng dẫn chi tiết, và ưu tiên của mình là chạy được trên terminal của Android Studio hoặc VS để tăng hiệu quả trong công việc, tránh mở nhiều tab hay cửa số.
Sau nhiều giờ vật lộn thì mình cũng tìm được solution đáp ứng được nhu cầu, và mình xin chia sẻ bài viết này  sẽ giúp ích cho những ai đang gặp vấn đề tương tự.

## II. Thực hiện

### 1. Cài đặt Chocolatey

[Chocolatey](https://chocolatey.org/) là package manager hỗ trợ cho hệ điều hành Windows, nhằm quản lý: cài đặt, update, xóa,... phần mềm thông command line, tương tự như Homebrew bên MacOS, hoặc Dpkg bên Linux.

Để cài đặt Chocolatey, bạn truy cập vào https://chocolatey.org/install và làm theo hướng dẫn nha.

### 2. Cài đặt lcov

lcov, viết tắt của "Linux Test Coverage", là một tool dùng để thu thập các thông tin về code coverage trong source code. Chúng ta sẽ dùng tool này để generate file lcov.info sang dạng html view.

Để cài đặt lcov, các bạn sử dụng command line dưới đây trong terminal, Chocolatey sẽ fetch và cài đặt lcov về máy.

```bash
choco install lcov
```

Chắc bạn sẽ cảm thấy khó hiểu khi Windows lại sử dụng tool mang tên Linux, thì lcov package mà chúng ta cài về sẽ gồm 2 phần:

- [Strawberry Perl](https://strawberryperl.com/) - bản distribution của ngôn ngữ lập trình scripting Perl chạy trên Windows.
- Các file Perl script để chạy function của lcov.

Như vậy, lcov sẽ sử dụng Perl để chạy script covert file lcov.ico sang dạng html.

### 3. Cài đặt biến môi trường

Sau khi cài đặt lcov xong, chúng ta sẽ cài đặt biến môi trường trỏ tới thư mục cài đặt lcov trên Windows.

<figure>
  <img src="{{ site.url }}{{ site.baseurl }}\assets\images\2023-10-12-flutter-test-coverage-windows_1.png">
</figure>

```
GENHTML - C:\ProgramData\chocolatey\lib\lcov\tools\bin\genhtml (thay đổi path nếu bạn cài đặt lcov vào thư mục khác)
```

### 4. Sử dụng lcov
#### a. Ở Powershell 

Các bạn chạy lệnh dưới đây trên Powershell:

```bash
perl $GENHTML path-thư-mục-chứa-file-lcov.ico -o path-thư-mục-lưu-html
```

Ví dụ:
```bash
perl $GENHTML .\coverage\lcov.info -o .\coverage\html
```

#### b. Android Studio
Ở Android Studio, chúng ta chạy lệnh tương tự như Powershell nhưng thêm prefix env: ở trước GENHTML

```bash
perl $env:GENHTML path-thư-mục-chứa-file-lcov.ico -o path-thư-mục-lưu-html
```

#### c. Visual Studio Code

Trước khi chạy lệnh, chúng ta sẽ cài đặt thêm để VS Code sử dụng được biến môi trường của Windows

Bạn vào **File -> Preferences -> Profiles (Default) -> Show Profile Contents**, một side bar bên trái sẽ xuất hiện và bạn hãy chọn **Settings/settings.json** nha.

<figure>
  <img src="{{ site.url }}{{ site.baseurl }}\assets\gifs\2023-10-12-flutter-test-coverage-windows.gif" style="width:600px;height:500px;">
</figure>

Trong file `settings.json`, bạn thêm dòng code dưới đây (có thể tham khảo ở Gif trên):
```json
"terminal.integrated.env.window": {
  "PATH": "${env:PATH}",
  "GENHTML" : "${env:GENHTML}"
}

```

Sau đó chạy lệnh tương tự như Android Studio là được nha.
```bash
perl $env:GENHTML path-thư-mục-chứa-file-lcov.ico -o path-thư-mục-lưu-html
```

-------------------------------------

Sau khi chạy lệnh xong, chúng ta sẽ có thư mục lib chứa file html + css + ảnh,..., bạn chỉ cần mở file `index.html` bằng trình duyệt thì có thể xem coverage report được rồi.

<figure>
  <img src="{{ site.url }}{{ site.baseurl }}\assets\images\2023-10-12-flutter-test-coverage-windows_2.png" style="width:250px;height:450px;">
</figure>

<figure>
  <img src="{{ site.url }}{{ site.baseurl }}\assets\images\2023-10-12-flutter-test-coverage-windows_3.png">
</figure>

## III. Kết luận

Hy vọng bài viết này giúp ích cho bạn trong việc generate coverage report sang HTML View từ lcov.ico khi viết test cho ứng dụng Flutter trên Window. Nếu ai có solution tốt hơn thì email cho mình: tinhhuynh.dev@gmail.com để thảo luận nha (khi nào có thời gian, mình sẽ tích hợp Comment plugin cho tiện ^^).

## IV. Tham khảo

- [How to Generate and Analyze a Flutter Test Coverage Report in VSCode](https://codewithandrea.com/articles/flutter-test-coverage/)
- [Lcov On Windows](https://fredgrott.medium.com/lcov-on-windows-7c58dda07080)
