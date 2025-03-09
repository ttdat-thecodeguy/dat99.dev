---
title: Hibernate là gì?
date: 2025-03-09 11:22:00 +/-TTTT
categories: [Hibernate]
tags: [hibernate, springboot]   
authors: dat99
---



## Hibernate là gì?

- làm 1 framework
- ánh xạ các đối tượng Java (POJOs) thành các bảng trong cơ sở dữ liệu quan hệ (RDBMS)
- 1 object tương ứng với 1 dòng trong bảng cơ sở dữ liệu

## Hibernate annonation

### Group 1 - annonation thuộc nhóm ánh xạ dòng, cột, primary key..

- @Id - đánh dấu cột là primary key 
- @Column - đánh dấu cột là column
- @Transient - thuộc tính này sẽ không được ánh xạ
- @Lob - thuộc tính này có kiểu dữ liệu LoB
- @Enity -một lớp là một entity để Hibernate quản lý 

### Group 2 - annonation mappings

- @OneToOne, 
- @OneToMany 
- @ManyToOne
- @ManyToMany

### Group 3 - Annonation mô tả chiến lượng tăng dữ liệu trên

@GeneratedValue(strategy = GenerationType.IDENTITY) - chiến lược tăng giá trị trong hibernate

## Hibernate enity life cycle

1. Từ trạng thái Transient → Persistent (thông qua hành động save object đã được hibernate quản lí và mapping với 1 dòng trong cơ sở dữ liệu) => sau trạng thái perisist mọi thay đổi trên object sẽ được hibernate lưu xuống csdl mặc dù không call save

2. Perisist -> Detached Entity đã từng ở Persistent state, nhưng Session đã đóng hoặc entity bị tách khỏi Session, Hibernate không còn theo dõi sự thay đổi của entity.

3. Perisists -> removed , call delete() Entity bị đánh dấu để xóa khỏi database.