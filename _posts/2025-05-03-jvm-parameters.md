---
title: JVM parameters quan trọng bạn nhất định phải biết trong java?
date: 2025-05-03 11:22:00 +/-TTTT
categories: [JVM, CMD, Java , SpringBoot ]
tags: [SpringBoot]   
authors: dat99
---



### I, Mở đầu

Xin chào các bạn, hôm nay mình xin giới thiệu 1 bài viết liên quan đến các parameter trong java. Hẳn trước giờ các bạn cũng quá quen với lệnh `java -jar hello.jar` 
dùng để chạy 1 jar và các bạn vẫn ngỡ chỉ có thế, mấy thứ như GC, Heap hay stack thật sự quá xa vời với bạn và bạn chẳng thể động tay vào chúng. Thực ra các bạn quyền năng hơn các bạn tưởng nhiều đấy, trong bài viết hôm nay mình sẽ chỉ các bạn trở thành 1 vị thần trong ứng dụng của mình, chỉ bằng với 1 vài tham số đơn giản mà những nhà phát triển tài ba đã cài cắm vào ứng dụng của bạn

### II. Một số parameters quan trọng

#### ❗ Nguyên tắc chính
1)  Các options thuộc kiểu boolean trong jvm có thể bật lên bằng `-XX:+` và tắt đi bằng `-XX:-`.  

2)  Các options thuộc kiểu số học trong jvm có thể set -XX:=, number có đuôi m kí hiệu là Megabytes, k hoặc K là kilobytes và g hoặc G là giga

3)  String JVM option được set bằng `-XX:=`, theo sau là path

#### 1. Truyền tham số vào chương trình

các bạn còn nhớ hàm main trong java chứ, cấu trúc nó như thế này:
```
   public static void main (String []args) { 
       for (String arg : args) {
           System.out.println("Hello " + arg);
       }
   }
```
đã bao giờ các bạn tự hỏi làm thế nào để sài mảng args bên trên chưa? việc đó vô cùng đơn giản, khi các bạn chạy file jar, chúng ta sẽ thêm nó vào phía sau: `java -jar hello.jar dat hai huynh`
=>chương trình sẽ in ra:
```
hello dat
hello hai
hello huynh
```
**mẹo nhỏ**: khi coding trong intellji các bạn có thể tạo nhanh println bằng cách gõ `sout`

#### 2. Các parameters quan trọng liên quan đến hệ thống chính

**a. Cách xem và setting JVM parameters**

- XX:+PrintCommandLineFlags: xem toàn bộ các thông số liên quan, nếu bạn sử dụng parameter này với java cmd bạn có thể xem thông số mặc định
`java -XX:+PrintFlagsFinal -version`

![image.png](https://images.viblo.asia/eb46c2c5-f597-4264-baa6-5957472c2263.png)

ví dụ: mình muốn xem giới hạn tối đa của max heap size, mình sẽ xem biến `MaxHeapSize`
![image.png](https://images.viblo.asia/066a78f4-d124-4b14-8338-f1c5a115e04b.png)

sau đó, mình muốn set lại giá trị này, mình chỉ cần chạy lệnh `java -XX:MinHeapDeltaBytes=<giá trị_byte> -jar hello.jar`

bạn hoàn toàn có thể chạy lệnh này với 1 jar cụ thể, các thông số chính của jar sẽ xuất hiện
![image.png](https://images.viblo.asia/b6398eb5-8f20-4d9b-ba58-ca23ff28462a.png)


**b. Các parameters liên quan đến Heap**

 `-Xms`  vd: `-Xms400M`  => đặt mức java heap size
     
 `-Xmx` vd: `-Xmx200M`   => đặt mức maximum cho java heap size
     
 `-Xss` vd:  `-Xss200M` => đặt kích thước thread stack
  
  trong ví dụ dưới đây, mình đã đặt 2 parameters là `-Xms200M` và `-Xmx400M` tương ứng với mức khởi tạo 200mb và mức cao nhất 400mb, ta có thể quan sát hoạt động của heap thông qua tool
  
  ![image.png](https://images.viblo.asia/61bbed96-395f-42a4-8169-16069a52dc92.png)

**c. Các parameters liên quan đến GC**

`-XX:+UseParallelGC`  => sử dụng thuật toán song song để tăng tốc độ thu gom rác

`-XX:-UseSerialGC` => JVM sẽ thực hiện quá trình thu gom rác bằng một luồng duy nhất, không dùng song song hoặc đồng thời.

`-verbose:gc` => trong rất nhiều trường hợp có nhiều lỗi xảy ra bởi gc. Khi bật, JVM sẽ ghi log chi tiết mỗi lần GC xảy ra, bao gồm thời gian GC mất bao lâu. Với ứng dụng chạy trên Tomcat, các log này sẽ xuất hiện trong file catalina.out

#### 2. Các custom config

khi các bạn làm việc với spring boot, chúng ta thường hay set chúng trong file `application.properties` hoặc `application.yml`. Tuy nhiên, trong nhiều trường hợp, 1 vài tham số sẽ động hoàn toàn, các bạn không sẽ set chúng bên trong file config mà thay vào đó sẽ sử dụng JVM system properties. 1 ví dụ cụ thể là set lại môi trường khi ta muốn chạy trong môi trường khác nhau, các bạn có thể sử dụng lệnh sau
```
java -jar hello.jar -Dspring.profiles.active=stg
```

#### 3. Sử dụng các parameter này trong intellji

nếu các bạn sử dụng intellji, các bạn chỉ cần vào phần edit configuration -> chọn `add VM options` -> thêm các paramter cần thiết như hình dưới

![image.png](https://images.viblo.asia/6c47f6ac-bbf3-4143-8e95-277bdfb22f39.png)

**Mẹo** : các bạn có thể theo dõi cách bộ nhớ hoạt động ngay trong intellji bằng cách bật profier **chỉ có ở phiên bản ultimate**

![image.png](https://images.viblo.asia/bff3c7b5-cce5-4a6f-a9cf-be88c6a575f8.png)

với profiler, bạn có thể monitor cpu và heap trong hệ thống
![image.png](https://images.viblo.asia/8de006d7-dce8-4915-8909-76f01630d74c.png)

### III. Kết luận

Qua bài viết này, mình hy vọng các bạn đã có thêm một chút kiến thức mới liên quan đến Java cũng như các tham số cấu hình. Những kiến thức này mình cũng đang trong quá trình tìm hiểu và thử nghiệm, nên chưa thể chia sẻ sâu hơn được, nếu có bất kì sai sót gì, mong các bạn thông cảm. 

Nếu bạn thấy bài viết hữu ích, và cũng yêu thích mình cũng như những chia sẻ sắp tới, đừng ngần ngại để lại một upvote và follow mình để không bỏ lỡ các bài viết tiếp theo

### IV. Tham khảo
1. Java_OPTs: https://gxsoftware.atlassian.net/wiki/spaces/PD/pages/24719663/JAVA_OPTS+Parameters
2. Ảnh thumbnail : ChatGPT