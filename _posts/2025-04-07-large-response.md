---
title: Tớ đã đối mặt với dữ liệu lớn như thế nào?
date: 2025-04-07 11:22:00 +/-TTTT
categories: [Large Response, SpringBoot ]
tags: [hibernate, SpringBoot]   
authors: dat99
---



## Đặt vấn đề

bạn có 1 ông khách hàng ~~(Bên cam)~~ khá là khó tính, bạn được giao cho 1 nhiệm vụ là làm trang user management, vì biz làm ăn nhỏ nên chỉ khoảng 1000 users và ông ta không thích cái tính năng phân trang cho lắm, ô ấy thích cái kiểu mà load 1 lần xem cả giống như mấy application trên máy tính, sau 7749 bước mà bạn không thể làm thỏa mãn ông ấy, bạn mới phát hiện ra nhà ổng sài 3G (~~mạng cam chỉ thế thôi~~), . Tuyệt, ông ta muốn thời gian tải thấp hơn, bạn phát hiện ra là do response quá lớn,  giờ đây bạn phải tìm ra được 1 cách nào đó để giảm thiểu nó tới mức tối đa để thỏa mãn ổng hoặc bạn sẽ bị chích điện⚡️⚡️⚡️

~~Note nhỏ nhẹ: câu chuyện này tôi tự bịa ra đấy, đừng có tin~~ 

## Tóm tắt data và cách test

bạn có thể gen data ảo thông qua https://www.mockaroo.com/
users 
```
{
   "id" : 1, 
   "firstName" : "Test", 
   "lastName" : "test", 
   "street" : "22 Nguyen Trung Truc", 
   "city" : "Ho Chi Minh City", 
   "country" : "VN", 
   "phoneNumber" : "098.222.333", 
   "email" : "test01@gmail.com"
}
```
số lượng : 1000 users

sau khi có data các bạn có thể save lại và đọc nó từ phía server, nếu các bạn lười tạo 1 ví dụ các bạn có thể sử dụng code của tôi theo link sau:
https://github.com/ttdat-thecodeguy/large-response

Đầu tiên ta sẽ kiểm tra các thông số ban đầu, ta sẽ sử dụng network throtting cho việc kiểm tra 3G:  download: 400Kbit/s upload: 400Kbit/s latency:  2000ms  targetLatency:  400ms

![ảnh.png](https://images.viblo.asia/8951a578-e93b-41d6-8471-c5eb1683be2a.png)

các thông số timing:

![ảnh.png](https://images.viblo.asia/b5878c4e-4979-4ef1-b0a0-a2904f95ad21.png)


## Các kỹ thuật để giảm thiểu response data

### 1. gzip

hồi còn đi học tôi nhớ lúc nộp bài lên cái trang ì ạch của trường tôi, thầy hay bảo mấy em phải zip trước rồi mới nộp, 1 phần vì hệ thống trường chỉ cho nộp 1 file duy nhất (tôi nhớ vậy), phần còn lại là file chỉ được tầm đâu đó 1 mb, vậy nên zip là cách tốt nhất.
Quay trở lại vấn đề, để triển khai gzip trong spring boot chúng ta có thể làm đơn giản như sau

```
server.compression.enabled=true
server.compression.mime-types=application/json,application/xml,text/html,text/xml,text/plain,application/javascript,text/css
server.compression.min-response-size=10240 
```

tiếp theo ta sẽ thử kiểm tra các thông số sau khi gzip với lượng dữ liệu trên

![ảnh.png](https://images.viblo.asia/764343f9-ff5d-491b-98da-9f82364cce94.png)

thông số cho thấy thời gian response đã giảm đi 1 nửa, và lượng dữ liệu chỉ còn 23.04% so với ban đầu

### 2. Field name shorter

Well, well well, sau khi zip xong vẫn không thỏa mãn được ông ấy, vẫn là back về cái mớ kỉ niệm hồi còn bập bõm nộp bài trước 1 tiếng deadline, cái hồi ấy hệ thống trường chỉ có 1MB mà project của tôi đến tận 1.1MB, tôi liền lọ mọ check lại đống source, xóa hết những thứ gì có thể xóa và chỉ chừa lại đúng file src, tôi cũng xóa luôn các file gen tests của hệ thống, kết quả là bài nộp của tôi chỉ còn lại vài trăm KB. 

Ta cũng có thể áp cách trên cho response data lớn, bạn có thể xóa hết những gì cần xóa và để lại những thứ nhận biết duy nhất, áp dụng nguyên lí này tôi nhanh chóng sửa lại model cho response ( ***các bạn làm mà làm api cho cả mobile và web phải tính versioning nhé , không thì lên bàn thờ do chích điện cả lũ đấy ***), response mới sẽ trông như thế này:


```
{
   "id" : 1, 
   "fn" : "Test", 
   "ln" : "test", 
   "str" : "22 Nguyen Trung Truc", 
   "ci" : "Ho Chi Minh City", 
   "ct" : "VN", 
   "p" : "098.222.333", 
   "e" : "test01@gmail.com"
}
```

Well, giảm đi số kí tự trả về nghĩa là giảm response data, dưới đây là kết quả sau khi test

![ảnh.png](https://images.viblo.asia/6923902d-8133-41b5-b886-2e9ef050057a.png)

=> lượng dữ liệu giảm 4.8kb so với ban đầu sau khi áp dụng gzip

### 3. Serialize thành array

Vẫn thất bại, cách trên bị 1 hạn chế là sau khi cắt giảm chữ trong field, đầu tôi sẽ bị lú (chắc là giữ lại cũng được),  trong đầu tôi lại lóe lên 1 ý tưởng khác, vẫn trên tư tưởng là bỏ đi những gì thừa thãi, lần này tôi sẽ serialize thành array, toàn bộ các field sẽ được quy ước theo thứ tự và trả trong field riêng "rule", giờ đây response của chúng ta sẽ như sau (***chú ý: để nâng cao tính flexible các bạn có thể tạo ra 1 api v2 cho phép query theo cột***)

```
{
 "fields" : [ "id", "firstName", "lastName", "street", "city", "country", "phone", "email" ],
 "data": [[1,"Test","test","22 Nguyen Trung Truc","Ho Chi Minh City","VN","098.222.333","test01@gmail.com"]]
}
```

đây là kết quả sau khi thực thi:

![ảnh.png](https://images.viblo.asia/d23fbe98-0746-4252-9a23-fdaa0bb8e172.png)

=>dữ liệu đã giảm đi 8kb sau khi áp dụng gzip


tuy nhiên kĩ thuật này vẫn còn 1 ưu điểm, đó là khi ta chỉ lấy 1 ít cột, thời gian lấy dữ liệu sẽ thực sự còn rất ít , trong trường hợp dưới đây tôi chỉ lấy 3 cột

![ảnh.png](https://images.viblo.asia/ecb6ad76-ee3e-4437-ad3a-47a914e21584.png)

=>CHỈ CÒN 17.8KB sau khi áp dụng gzip

### 4. Các kỹ thuật xử lí UI/UX

#### 4.a Kỹ thuật lazy loading

Vẫn fail, lần này người ae Front end (vẫn là tôi) đã giới thiệu cho tôi 1 cách để không phải load dữ liệu lớn, đó là lz loading, tôi sẽ tính toán làm sao khi ông khách hàng scroll xuống gần dưới cùng thì tôi sẽ load tiếp 1-2 trang dữ liệu nữa, có thể nâng cao bằng cách tính toán tốc độ cuộn chuột để xác định lượng pageSize, số lượng page cần load

#### 4.b Caching

làm gì cũng phải có tí mưu mới nên sự được, nên lần này ae chúng tôi đã tính đến 1 con bài tẩy ít dùng để qua con trăng này, cụ thể kế hoạch như sau:

- Bước 1: Xác định các cột thay đổi và không thay đổi trong user ( thường user sẽ ít có khả năng có thể thay đổi hết các trường => trừ trường hợp đặc biệt )
- Bước 2: Với các trường không thay đổi => cache ở UI, trong localStorage (***set TTL để đề phòng trường hợp đặc biệt***)
- Bước 3: Với các trường thay đổi => sử dụng kĩ thuật serialize thành array để get dữ liệu
- Bước 4: Woalla

#### 4.c Column Reducing

đến lúc này mọi thứ đã khá là ngon nghẻ, ô khách cũng khá hài lòng rồi, lúc này tụi tôi (thực chất vẫn là tôi ) vẫn đang suy tính đến 1 trường hợp xa hơn, đó là khi bảng có 30 chục cột thì sao (hơi căng), cá nhân tôi thực tế chưa gặp trường hợp này nhưng tôi nghĩ có thể thêm feature cho phép select từng column có thể view hoặc dùng 1 loại lazy loading dạng ngang, cụ thể nó như sau:

bạn show các cột theo thứ tự -> khi user scroll ngang -> bạn load tiếp các cột tiếp theo, các bạn có thể coding thử xem

## Tổng kết tí

kĩ thuật này tôi đã được 1 anh architechture truyền dạy lúc tôi còn đang chuẩn bị trước tuần bắt đầu dự án, anh ấy đặt ra 1 vấn đề khá là hay, làm sao để tối ưu hóa response trong trường hợp dữ liệu khá lớn hoặc liên tục, có thể kể đến như hệ thống biểu đồ chứng khoán, nơi dữ liệu được cập nhật liên tục, database cập nhật dữ liệu biểu đồ theo từng giây hoặc sẽ có những trường hợp người ta cần query 500 row per pages, lúc này dữ liệu trả ra sẽ khá lớn và để tăng tốc cho api của hệ thống (cũng như tiết kiệm lưu lượng gói 4g của tôi), các bạn có thể linh động áp dụng các phương án trên để tăng tốc hệ thống của mình nhé 

## Tài liệu tham khảo
- Baeldung - https://www.baeldung.com/json-reduce-data-size