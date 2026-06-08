# Bài 2: Controller, Service, Module trong NestJS

## 1. Mục tiêu bài học

Sau buổi này, bạn cần hiểu và làm được:

1. Controller dùng để làm gì.
2. Service dùng để làm gì.
3. Module dùng để làm gì.
4. Hiểu luồng xử lý request cơ bản trong NestJS.
5. Tạo được `BooksModule`, `BooksController`, `BooksService`.
6. Tạo được API đơn giản:

```txt
GET /books
GET /books/:id
POST /books
```

---

# 2. Ôn lại kiến thức Bài 1

Ở Bài 1, chúng ta đã biết một project NestJS cơ bản có các file chính:

```txt
main.ts
app.module.ts
app.controller.ts
app.service.ts
```

Luồng xử lý cơ bản là:

```txt
Client → Controller → Service → Response
```

Ví dụ:

```txt
Người dùng mở http://localhost:3000
        ↓
AppController nhận request GET /
        ↓
AppController gọi AppService
        ↓
AppService trả về "Hello World!"
        ↓
Trình duyệt hiển thị kết quả
```

Trong Bài 2, chúng ta sẽ không chỉ dùng `AppController` và `AppService` mặc định nữa. Chúng ta sẽ tạo một module riêng tên là `books`.

---

# 3. Vì sao cần Controller, Service, Module?

Khi project nhỏ, bạn có thể viết mọi thứ vào một file.

Ví dụ:

```ts
app.get('/books', (req, res) => {
  res.json([
    { id: 1, title: 'Clean Code' },
    { id: 2, title: 'Pragmatic Programmer' },
  ]);
});
```

Cách này nhanh, nhưng khi project lớn sẽ khó quản lý.

Ví dụ hệ thống thư viện có nhiều chức năng:

```txt
Books
Users
Borrows
Auth
Categories
Payments
Reports
```

Nếu tất cả code để chung một chỗ, project sẽ rất rối.

NestJS giải quyết bằng cách chia code thành các phần rõ ràng:

```txt
Controller → nhận request
Service    → xử lý logic
Module     → gom nhóm controller và service
```

---

# 4. Controller là gì?

Controller là nơi nhận request từ client.

Client có thể là:

```txt
Trình duyệt
Frontend Angular/React/Vue
Mobile app
Postman
Cypress test
```

Ví dụ client gọi:

```txt
GET /books
```

Thì `BooksController` sẽ nhận request này.

Controller thường xử lý các việc:

```txt
Nhận request
Đọc params
Đọc body
Gọi service
Trả response
```

Controller không nên chứa quá nhiều logic phức tạp.

Ví dụ không nên viết toàn bộ logic thêm sách, kiểm tra sách, xử lý dữ liệu trực tiếp trong controller.

Controller nên mỏng, service mới là nơi xử lý chính.

---

# 5. Service là gì?

Service là nơi xử lý business logic.

Business logic là logic nghiệp vụ của ứng dụng.

Ví dụ với hệ thống thư viện:

```txt
Tìm danh sách sách
Tìm sách theo id
Thêm sách mới
Kiểm tra sách có tồn tại không
Kiểm tra số lượng sách
Cập nhật sách
Xóa sách
```

Service giúp code dễ test hơn.

Ví dụ sau này khi viết unit test, ta có thể test riêng `BooksService` mà không cần gọi HTTP thật.

---

# 6. Module là gì?

Module là nơi gom nhóm các thành phần liên quan.

Ví dụ với chức năng quản lý sách, ta sẽ có:

```txt
books.module.ts
books.controller.ts
books.service.ts
```

Có thể hiểu:

```txt
BooksModule quản lý BooksController và BooksService.
```

Khi project lớn, mỗi nhóm chức năng nên có module riêng:

```txt
BooksModule
UsersModule
BorrowsModule
AuthModule
CategoriesModule
```

Cấu trúc này giúp project dễ mở rộng và dễ bảo trì.

---

# 7. Luồng xử lý request trong Bài 2

Khi client gọi:

```txt
GET /books
```

Luồng xử lý sẽ là:

```txt
Client
  ↓
BooksController
  ↓
BooksService
  ↓
BooksController
  ↓
Response
```

Ví dụ cụ thể:

```txt
Client gọi GET /books
        ↓
BooksController.findAll()
        ↓
BooksService.findAll()
        ↓
Trả về danh sách sách
```

---

# 8. Tạo BooksModule, BooksController, BooksService

Mở terminal trong project:

```bash
cd nest-library-api
```

Chạy lệnh tạo module:

```bash
nest g module books
```

Chạy lệnh tạo controller:

```bash
nest g controller books
```

Chạy lệnh tạo service:

```bash
nest g service books
```

Sau khi chạy xong, NestJS sẽ tạo thêm thư mục:

```txt
src/books/
  books.controller.ts
  books.controller.spec.ts
  books.module.ts
  books.service.ts
  books.service.spec.ts
```

Đồng thời, `BooksModule` sẽ được tự động import vào `AppModule`.

---

# 9. Kiểm tra file `app.module.ts`

Mở file:

```txt
src/app.module.ts
```

Bạn sẽ thấy gần giống như sau:

```ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { BooksModule } from './books/books.module';

@Module({
  imports: [BooksModule],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

Phần quan trọng là:

```ts
imports: [BooksModule]
```

Nghĩa là app chính đã biết đến `BooksModule`.

Nếu không import module, các controller/service trong module đó sẽ không được app sử dụng.

---

# 10. Kiểm tra file `books.module.ts`

Mở file:

```txt
src/books/books.module.ts
```

Nội dung thường sẽ như sau:

```ts
import { Module } from '@nestjs/common';
import { BooksService } from './books.service';
import { BooksController } from './books.controller';

@Module({
  controllers: [BooksController],
  providers: [BooksService],
})
export class BooksModule {}
```

Giải thích:

```ts
controllers: [BooksController]
```

Nghĩa là module này dùng `BooksController`.

```ts
providers: [BooksService]
```

Nghĩa là module này dùng `BooksService`.

Nói dễ hiểu:

```txt
BooksModule gom BooksController và BooksService lại với nhau.
```

---

# 11. Tạo dữ liệu sách tạm thời trong Service

Trong Bài 2, ta chưa dùng database.

Ta sẽ lưu dữ liệu tạm bằng array.

Mở file:

```txt
src/books/books.service.ts
```

Sửa thành:

```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class BooksService {
  private books = [
    {
      id: 1,
      title: 'Clean Code',
      author: 'Robert C. Martin',
    },
    {
      id: 2,
      title: 'The Pragmatic Programmer',
      author: 'Andrew Hunt, David Thomas',
    },
  ];

  findAll() {
    return this.books;
  }

  findOne(id: number) {
    return this.books.find((book) => book.id === id);
  }

  create(data: { title: string; author: string }) {
    const newBook = {
      id: this.books.length + 1,
      ...data,
    };

    this.books.push(newBook);

    return newBook;
  }
}
```

Giải thích từng phần.

Đây là dữ liệu tạm:

```ts
private books = [
  {
    id: 1,
    title: 'Clean Code',
    author: 'Robert C. Martin',
  },
  {
    id: 2,
    title: 'The Pragmatic Programmer',
    author: 'Andrew Hunt, David Thomas',
  },
];
```

Từ khóa `private` nghĩa là biến này chỉ dùng bên trong `BooksService`.

Method lấy tất cả sách:

```ts
findAll() {
  return this.books;
}
```

Method tìm sách theo id:

```ts
findOne(id: number) {
  return this.books.find((book) => book.id === id);
}
```

Method thêm sách mới:

```ts
create(data: { title: string; author: string }) {
  const newBook = {
    id: this.books.length + 1,
    ...data,
  };

  this.books.push(newBook);

  return newBook;
}
```

Ở đây:

```ts
...data
```

nghĩa là lấy toàn bộ field từ object `data`.

Ví dụ `data` là:

```ts
{
  title: 'Refactoring',
  author: 'Martin Fowler'
}
```

Thì `newBook` sẽ là:

```ts
{
  id: 3,
  title: 'Refactoring',
  author: 'Martin Fowler'
}
```

---

# 12. Viết BooksController

Mở file:

```txt
src/books/books.controller.ts
```

Sửa thành:

```ts
import { Body, Controller, Get, Param, Post } from '@nestjs/common';
import { BooksService } from './books.service';

@Controller('books')
export class BooksController {
  constructor(private readonly booksService: BooksService) {}

  @Get()
  findAll() {
    return this.booksService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.booksService.findOne(Number(id));
  }

  @Post()
  create(@Body() body: { title: string; author: string }) {
    return this.booksService.create(body);
  }
}
```

---

# 13. Giải thích BooksController

## 13.1. `@Controller('books')`

```ts
@Controller('books')
```

Nghĩa là controller này xử lý các route bắt đầu bằng:

```txt
/books
```

Ví dụ:

```txt
GET /books
GET /books/1
POST /books
```

---

## 13.2. Inject BooksService

```ts
constructor(private readonly booksService: BooksService) {}
```

Dòng này nghĩa là NestJS sẽ tự đưa `BooksService` vào `BooksController`.

Nhờ vậy controller có thể gọi:

```ts
this.booksService.findAll()
this.booksService.findOne()
this.booksService.create()
```

Đây chính là Dependency Injection.

---

## 13.3. API GET /books

```ts
@Get()
findAll() {
  return this.booksService.findAll();
}
```

Vì controller đã có prefix `books`, nên `@Get()` tương ứng với:

```txt
GET /books
```

Khi client gọi `GET /books`, controller sẽ gọi service:

```ts
this.booksService.findAll()
```

Sau đó trả danh sách sách về client.

---

## 13.4. API GET /books/:id

```ts
@Get(':id')
findOne(@Param('id') id: string) {
  return this.booksService.findOne(Number(id));
}
```

Route này xử lý:

```txt
GET /books/1
GET /books/2
GET /books/10
```

`@Param('id')` dùng để lấy giá trị trên URL.

Ví dụ client gọi:

```txt
GET /books/1
```

Thì:

```ts
id = '1'
```

Lưu ý `id` lấy từ URL sẽ là string, nên ta cần đổi sang number:

```ts
Number(id)
```

---

## 13.5. API POST /books

```ts
@Post()
create(@Body() body: { title: string; author: string }) {
  return this.booksService.create(body);
}
```

Route này xử lý:

```txt
POST /books
```

`@Body()` dùng để lấy dữ liệu gửi lên từ request body.

Ví dụ client gửi:

```json
{
  "title": "Refactoring",
  "author": "Martin Fowler"
}
```

Thì controller nhận được object đó trong biến `body`, sau đó truyền vào service:

```ts
this.booksService.create(body)
```

---

# 14. Chạy server

Chạy lệnh:

```bash
npm run start:dev
```

Nếu server đang chạy rồi thì chỉ cần lưu file, NestJS sẽ tự reload.

---

# 15. Test API bằng trình duyệt

Mở trình duyệt:

```txt
http://localhost:3000/books
```

Kết quả mong muốn:

```json
[
  {
    "id": 1,
    "title": "Clean Code",
    "author": "Robert C. Martin"
  },
  {
    "id": 2,
    "title": "The Pragmatic Programmer",
    "author": "Andrew Hunt, David Thomas"
  }
]
```

Mở tiếp:

```txt
http://localhost:3000/books/1
```

Kết quả mong muốn:

```json
{
  "id": 1,
  "title": "Clean Code",
  "author": "Robert C. Martin"
}
```

---

# 16. Test API POST bằng Postman hoặc Thunder Client

Vì trình duyệt chỉ tiện test GET, với POST nên dùng:

```txt
Postman
Thunder Client trong VS Code
Insomnia
curl
```

Gửi request:

```txt
POST http://localhost:3000/books
```

Body dạng JSON:

```json
{
  "title": "Refactoring",
  "author": "Martin Fowler"
}
```

Kết quả mong muốn:

```json
{
  "id": 3,
  "title": "Refactoring",
  "author": "Martin Fowler"
}
```

Sau đó gọi lại:

```txt
GET http://localhost:3000/books
```

Bạn sẽ thấy sách mới đã được thêm vào danh sách.

---

# 17. Test POST bằng curl

Nếu muốn dùng terminal:

```bash
curl -X POST http://localhost:3000/books \
  -H "Content-Type: application/json" \
  -d "{\"title\":\"Refactoring\",\"author\":\"Martin Fowler\"}"
```

Trên Windows CMD, có thể dùng:

```bash
curl -X POST http://localhost:3000/books -H "Content-Type: application/json" -d "{\"title\":\"Refactoring\",\"author\":\"Martin Fowler\"}"
```

---

# 18. Vấn đề hiện tại của API

API hiện tại chạy được, nhưng vẫn còn nhiều điểm chưa tốt:

```txt
Nếu tìm id không tồn tại, API trả undefined
Chưa validate dữ liệu gửi lên
Chưa kiểm tra title có rỗng không
Chưa kiểm tra author có rỗng không
Chưa có DTO
Chưa có database
Chưa có test
```

Ví dụ gọi:

```txt
GET /books/999
```

Hiện tại có thể trả về rỗng hoặc `undefined`.

Ở bài sau, ta sẽ cải thiện dần bằng cách trả lỗi 404 và xây CRUD đầy đủ hơn.

---

# 19. Checklist hoàn thành Bài 2

Bạn đạt yêu cầu Bài 2 nếu làm được:

```txt
[ ] Hiểu Controller dùng để nhận request
[ ] Hiểu Service dùng để xử lý logic
[ ] Hiểu Module dùng để gom nhóm controller và service
[ ] Tạo được BooksModule
[ ] Tạo được BooksController
[ ] Tạo được BooksService
[ ] Tạo được API GET /books
[ ] Tạo được API GET /books/:id
[ ] Tạo được API POST /books
[ ] Test được GET /books trên trình duyệt
[ ] Test được POST /books bằng Postman hoặc curl
```

---

# 20. Bài tập thực hành cuối buổi

## Bài tập 1: Tạo module books

Chạy 3 lệnh:

```bash
nest g module books
nest g controller books
nest g service books
```

Kiểm tra thư mục:

```txt
src/books/
```

Phải có các file:

```txt
books.module.ts
books.controller.ts
books.service.ts
```

---

## Bài tập 2: Tạo dữ liệu sách tạm thời

Trong `BooksService`, tạo array `books` gồm ít nhất 3 quyển sách.

Mỗi sách có:

```txt
id
title
author
```

Ví dụ:

```ts
{
  id: 1,
  title: 'Clean Code',
  author: 'Robert C. Martin'
}
```

---

## Bài tập 3: Viết method trong service

Trong `BooksService`, viết 3 method:

```ts
findAll()
findOne(id: number)
create(data)
```

Yêu cầu:

```txt
findAll trả về toàn bộ danh sách sách
findOne tìm sách theo id
create thêm sách mới vào array
```

---

## Bài tập 4: Viết API trong controller

Trong `BooksController`, viết 3 API:

```txt
GET /books
GET /books/:id
POST /books
```

Yêu cầu:

```txt
Controller không tự xử lý logic phức tạp
Controller phải gọi BooksService
```

---

## Bài tập 5: Test bằng Postman

Test các API sau:

```txt
GET http://localhost:3000/books
GET http://localhost:3000/books/1
POST http://localhost:3000/books
```

Body cho POST:

```json
{
  "title": "Domain-Driven Design",
  "author": "Eric Evans"
}
```

---

# 21. Mini quiz kiểm tra hiểu bài

## Câu 1

Trong NestJS, Controller có nhiệm vụ chính là gì?

A. Xử lý request từ client
B. Lưu dữ liệu vào database
C. Cài package npm
D. Chạy server

<details>
<summary>Show answer</summary>

Đáp án đúng: **A. Xử lý request từ client**

Giải thích: Controller là nơi nhận request như `GET /books`, `POST /books`, sau đó gọi Service để xử lý.

</details>

---

## Câu 2

Service thường dùng để làm gì?

A. Nhận request trực tiếp từ trình duyệt
B. Xử lý business logic
C. Khởi động ứng dụng
D. Cấu hình TypeScript

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Xử lý business logic**

Giải thích: Service xử lý logic chính như tìm sách, thêm sách, cập nhật sách, xóa sách.

</details>

---

## Câu 3

Module trong NestJS dùng để làm gì?

A. Trang trí giao diện
B. Gom nhóm controller, service và các provider liên quan
C. Tạo database tự động
D. Build frontend

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Gom nhóm controller, service và các provider liên quan**

Giải thích: Module giúp tổ chức code theo từng chức năng, ví dụ `BooksModule`, `UsersModule`, `AuthModule`.

</details>

---

## Câu 4

Lệnh nào dùng để tạo module `books`?

A. `nest new books`
B. `nest create module books`
C. `nest g module books`
D. `npm g module books`

<details>
<summary>Show answer</summary>

Đáp án đúng: **C. `nest g module books`**

Giải thích: `g` là viết tắt của `generate`. Lệnh này tạo module mới trong NestJS.

</details>

---

## Câu 5

Nếu controller có decorator `@Controller('books')` và method có `@Get()`, route được tạo ra là gì?

A. `GET /`
B. `GET /books`
C. `POST /books`
D. `GET /app/books`

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. `GET /books`**

Giải thích: `@Controller('books')` tạo prefix `/books`, còn `@Get()` xử lý method GET tại chính prefix đó.

</details>

---

## Câu 6

Đoạn code sau dùng để làm gì?

```ts
@Param('id') id: string
```

A. Lấy dữ liệu từ request body
B. Lấy dữ liệu từ query string
C. Lấy dữ liệu `id` từ URL
D. Tạo id mới

<details>
<summary>Show answer</summary>

Đáp án đúng: **C. Lấy dữ liệu `id` từ URL**

Giải thích: Với route `GET /books/:id`, `@Param('id')` sẽ lấy giá trị `id` từ URL.

</details>

---

## Câu 7

Vì sao cần chuyển `id` từ string sang number?

A. Vì dữ liệu lấy từ URL mặc định là string
B. Vì NestJS không hỗ trợ string
C. Vì service không được nhận string
D. Vì JSON không hỗ trợ string

<details>
<summary>Show answer</summary>

Đáp án đúng: **A. Vì dữ liệu lấy từ URL mặc định là string**

Giải thích: Khi gọi `/books/1`, giá trị `id` nhận được thường là chuỗi `'1'`, nên cần `Number(id)` để so sánh với `book.id`.

</details>

---

## Câu 8

Luồng xử lý request cơ bản trong NestJS là gì?

A. Client → Service → Controller → Response
B. Client → Controller → Service → Response
C. Service → Module → Client
D. Module → Service → Controller

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Client → Controller → Service → Response**

Giải thích: Client gửi request, Controller nhận request, Service xử lý logic, sau đó trả response.

</details>

---

# 22. Ghi nhớ sau Bài 2

Sau bài này, cần nhớ 5 ý chính:

```txt
1. Controller nhận request từ client.
2. Service xử lý business logic.
3. Module gom nhóm controller và service.
4. Controller nên gọi service, không nên chứa logic phức tạp.
5. Luồng cơ bản là Client → Controller → Service → Response.
```

---

# 23. Chuẩn bị cho Bài 3

Ở Bài 2, ta đã có API cơ bản:

```txt
GET /books
GET /books/:id
POST /books
```

Ở Bài 3, ta sẽ hoàn thiện CRUD API đầy đủ:

```txt
GET    /books
GET    /books/:id
POST   /books
PATCH  /books/:id
DELETE /books/:id
```

Đồng thời học thêm:

```txt
@Patch()
@Delete()
@Body()
@Param()
NotFoundException
```

