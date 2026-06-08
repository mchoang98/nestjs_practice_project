# Bài 3: Tạo CRUD API cho Books

## 1. Mục tiêu bài học

Sau buổi này, bạn cần làm được:

1. Hiểu CRUD API là gì.
2. Tạo đầy đủ API quản lý sách.
3. Biết dùng `@Get()`, `@Post()`, `@Patch()`, `@Delete()`.
4. Biết lấy dữ liệu từ URL bằng `@Param()`.
5. Biết lấy dữ liệu từ request body bằng `@Body()`.
6. Biết trả lỗi `404 Not Found` khi không tìm thấy sách.
7. Hiểu vì sao nên để business logic trong Service thay vì Controller.

API cần hoàn thành trong bài này:

```txt id="dc9r3w"
GET    /books
GET    /books/:id
POST   /books
PATCH  /books/:id
DELETE /books/:id
```

---

# 2. Ôn lại Bài 2

Ở Bài 2, chúng ta đã tạo:

```txt id="84lhzu"
BooksModule
BooksController
BooksService
```

Và đã có 3 API cơ bản:

```txt id="nuowdq"
GET /books
GET /books/:id
POST /books
```

Luồng xử lý request là:

```txt id="yqf9vb"
Client → BooksController → BooksService → Response
```

Ví dụ:

```txt id="q38pfn"
Client gọi GET /books
        ↓
BooksController.findAll()
        ↓
BooksService.findAll()
        ↓
Trả về danh sách sách
```

Trong Bài 3, ta sẽ hoàn thiện CRUD đầy đủ.

---

# 3. CRUD là gì?

CRUD là viết tắt của 4 thao tác cơ bản với dữ liệu:

```txt id="hy9mx2"
C - Create
R - Read
U - Update
D - Delete
```

Trong REST API, CRUD thường tương ứng với các HTTP method:

| Chức năng          | HTTP Method | API          |
| ------------------ | ----------- | ------------ |
| Lấy danh sách sách | GET         | `/books`     |
| Lấy chi tiết sách  | GET         | `/books/:id` |
| Thêm sách mới      | POST        | `/books`     |
| Cập nhật sách      | PATCH       | `/books/:id` |
| Xóa sách           | DELETE      | `/books/:id` |

Ví dụ với hệ thống thư viện:

```txt id="kyt8d5"
Create  → thêm sách mới
Read    → xem danh sách hoặc xem chi tiết sách
Update  → sửa thông tin sách
Delete  → xóa sách
```

---

# 4. Model tạm thời của Book

Trong bài này, ta chưa dùng database.

Ta sẽ lưu dữ liệu tạm bằng array trong `BooksService`.

Mỗi quyển sách có cấu trúc:

```ts id="xlpctz"
export interface Book {
  id: number;
  title: string;
  author: string;
  isbn: string;
  quantity: number;
}
```

Giải thích:

| Field      | Kiểu dữ liệu | Ý nghĩa                          |
| ---------- | ------------ | -------------------------------- |
| `id`       | number       | Mã định danh của sách            |
| `title`    | string       | Tên sách                         |
| `author`   | string       | Tác giả                          |
| `isbn`     | string       | Mã ISBN                          |
| `quantity` | number       | Số lượng sách còn trong thư viện |

Ví dụ một book:

```ts id="y457r0"
{
  id: 1,
  title: 'Clean Code',
  author: 'Robert C. Martin',
  isbn: '9780132350884',
  quantity: 5,
}
```

---

# 5. Tạo interface Book

Trong thư mục:

```txt id="qn3z0j"
src/books/
```

Tạo file mới:

```txt id="yfclx8"
book.interface.ts
```

Nội dung:

```ts id="twk3n3"
export interface Book {
  id: number;
  title: string;
  author: string;
  isbn: string;
  quantity: number;
}
```

Interface giúp TypeScript hiểu cấu trúc của một object `Book`.

Lợi ích:

```txt id="bk1ijc"
Code rõ ràng hơn
Dễ bắt lỗi sai kiểu dữ liệu
Dễ đọc source code
Dễ maintain khi project lớn
```

---

# 6. Viết BooksService đầy đủ

Mở file:

```txt id="fkcu56"
src/books/books.service.ts
```

Sửa thành:

```ts id="n6gsud"
import { Injectable, NotFoundException } from '@nestjs/common';
import type { Book } from './book.interface';

@Injectable()
export class BooksService {
  private books: Book[] = [
    {
      id: 1,
      title: 'Clean Code',
      author: 'Robert C. Martin',
      isbn: '9780132350884',
      quantity: 5,
    },
    {
      id: 2,
      title: 'The Pragmatic Programmer',
      author: 'Andrew Hunt, David Thomas',
      isbn: '9780201616224',
      quantity: 3,
    },
  ];

  findAll(): Book[] {
    return this.books;
  }

  findOne(id: number): Book {
    const book = this.books.find((item) => item.id === id);

    if (!book) {
      throw new NotFoundException(`Book with id ${id} not found`);
    }

    return book;
  }

  create(data: Omit<Book, 'id'>): Book {
    const newBook: Book = {
      id: this.books.length + 1,
      ...data,
    };

    this.books.push(newBook);

    return newBook;
  }

  update(id: number, data: Partial<Omit<Book, 'id'>>): Book {
    const book = this.findOne(id);

    Object.assign(book, data);

    return book;
  }

  remove(id: number): Book {
    const book = this.findOne(id);

    this.books = this.books.filter((item) => item.id !== id);

    return book;
  }
}
```

---

# 7. Giải thích BooksService

## 7.1. Import `NotFoundException`

```ts id="ocxjdn"
import { Injectable, NotFoundException } from '@nestjs/common';
```

`NotFoundException` dùng để trả lỗi 404.

Ví dụ khi client gọi:

```txt id="mll2ha"
GET /books/999
```

Nếu sách không tồn tại, API nên trả:

```json id="klre83"
{
  "message": "Book with id 999 not found",
  "error": "Not Found",
  "statusCode": 404
}
```

Không nên trả `undefined`, vì client sẽ không biết chuyện gì xảy ra.

---

## 7.2. Dữ liệu tạm thời

```ts id="k391s0"
private books: Book[] = [
  {
    id: 1,
    title: 'Clean Code',
    author: 'Robert C. Martin',
    isbn: '9780132350884',
    quantity: 5,
  },
  {
    id: 2,
    title: 'The Pragmatic Programmer',
    author: 'Andrew Hunt, David Thomas',
    isbn: '9780201616224',
    quantity: 3,
  },
];
```

Ở đây ta dùng array thay cho database.

Dữ liệu này chỉ tồn tại khi server đang chạy.

Nếu restart server, dữ liệu thêm mới sẽ mất.

Điều này bình thường trong giai đoạn học CRUD cơ bản.

---

## 7.3. Method `findAll()`

```ts id="b5sy1f"
findAll(): Book[] {
  return this.books;
}
```

Method này trả về toàn bộ danh sách sách.

API tương ứng:

```txt id="1c8toc"
GET /books
```

---

## 7.4. Method `findOne(id)`

```ts id="slxz42"
findOne(id: number): Book {
  const book = this.books.find((item) => item.id === id);

  if (!book) {
    throw new NotFoundException(`Book with id ${id} not found`);
  }

  return book;
}
```

Method này tìm một quyển sách theo `id`.

Nếu tìm thấy, trả về book.

Nếu không tìm thấy, throw lỗi:

```ts id="hzugvc"
throw new NotFoundException(`Book with id ${id} not found`);
```

Trong NestJS, khi throw `NotFoundException`, framework sẽ tự động trả response HTTP status code `404`.

---

## 7.5. Method `create(data)`

```ts id="x9t2cc"
create(data: Omit<Book, 'id'>): Book {
  const newBook: Book = {
    id: this.books.length + 1,
    ...data,
  };

  this.books.push(newBook);

  return newBook;
}
```

Method này tạo sách mới.

`Omit<Book, 'id'>` nghĩa là dữ liệu đầu vào giống `Book`, nhưng không có field `id`.

Vì `id` sẽ do server tự tạo.

Ví dụ client gửi:

```json id="h929g5"
{
  "title": "Refactoring",
  "author": "Martin Fowler",
  "isbn": "9780201485677",
  "quantity": 4
}
```

Server sẽ tạo thành:

```json id="ql52hw"
{
  "id": 3,
  "title": "Refactoring",
  "author": "Martin Fowler",
  "isbn": "9780201485677",
  "quantity": 4
}
```

---

## 7.6. Method `update(id, data)`

```ts id="wvekc7"
update(id: number, data: Partial<Omit<Book, 'id'>>): Book {
  const book = this.findOne(id);

  Object.assign(book, data);

  return book;
}
```

Method này cập nhật sách theo `id`.

Dòng này:

```ts id="tgqteu"
const book = this.findOne(id);
```

vừa tìm sách, vừa tự xử lý lỗi 404 nếu không tìm thấy.

Dòng này:

```ts id="u34dss"
Object.assign(book, data);
```

dùng để cập nhật dữ liệu mới vào object cũ.

Ví dụ book ban đầu:

```json id="eq19wd"
{
  "id": 1,
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "9780132350884",
  "quantity": 5
}
```

Client gửi PATCH:

```json id="qajcxa"
{
  "quantity": 10
}
```

Kết quả sau update:

```json id="bzsytk"
{
  "id": 1,
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "9780132350884",
  "quantity": 10
}
```

---

## 7.7. Method `remove(id)`

```ts id="yybk3c"
remove(id: number): Book {
  const book = this.findOne(id);

  this.books = this.books.filter((item) => item.id !== id);

  return book;
}
```

Method này xóa sách theo `id`.

Đầu tiên gọi:

```ts id="9esofk"
const book = this.findOne(id);
```

Nếu sách không tồn tại, `findOne()` sẽ tự throw `NotFoundException`.

Nếu sách tồn tại, ta xóa bằng:

```ts id="drfi8m"
this.books = this.books.filter((item) => item.id !== id);
```

Sau đó trả về book vừa xóa.

---

# 8. Viết BooksController đầy đủ

Mở file:

```txt id="2y25ah"
src/books/books.controller.ts
```

Sửa thành:

```ts id="l6l5by"
import { Body, Controller, Delete, Get, Param, Patch, Post } from '@nestjs/common';
import { BooksService } from './books.service';
import type { Book } from './book.interface';

@Controller('books')
export class BooksController {
  constructor(private readonly booksService: BooksService) {}

  @Get()
  findAll(): Book[] {
    return this.booksService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string): Book {
    return this.booksService.findOne(Number(id));
  }

  @Post()
  create(@Body() body: Omit<Book, 'id'>): Book {
    return this.booksService.create(body);
  }

  @Patch(':id')
  update(
    @Param('id') id: string,
    @Body() body: Partial<Omit<Book, 'id'>>,
  ): Book {
    return this.booksService.update(Number(id), body);
  }

  @Delete(':id')
  remove(@Param('id') id: string): Book {
    return this.booksService.remove(Number(id));
  }
}
```

---

# 9. Giải thích BooksController

## 9.1. Import decorator cần dùng

```ts id="v4ce7u"
import { Body, Controller, Delete, Get, Param, Patch, Post } from '@nestjs/common';
```

Ý nghĩa:

| Decorator       | Công dụng                   |
| --------------- | --------------------------- |
| `@Controller()` | Khai báo controller         |
| `@Get()`        | Xử lý request GET           |
| `@Post()`       | Xử lý request POST          |
| `@Patch()`      | Xử lý request PATCH         |
| `@Delete()`     | Xử lý request DELETE        |
| `@Param()`      | Lấy dữ liệu từ URL          |
| `@Body()`       | Lấy dữ liệu từ request body |

---

## 9.2. Import interface bằng `import type`

```ts id="book-import-type"
import type { Book } from './book.interface';
```

`Book` là interface, chỉ dùng để kiểm tra kiểu dữ liệu khi viết TypeScript.

Khi project đang bật `isolatedModules` và `emitDecoratorMetadata`, các type xuất hiện trong method có decorator như `@Get()`, `@Post()`, `@Patch()`, `@Delete()` nên được import bằng `import type`.

Nếu dùng:

```ts id="book-import-wrong"
import { Book } from './book.interface';
```

Bạn có thể gặp lỗi:

```txt id="book-import-error"
TS1272: A type referenced in a decorated signature must be imported with 'import type'
```

Vì vậy trong controller và service, ta nên dùng:

```ts id="book-import-right"
import type { Book } from './book.interface';
```

---

## 9.3. `@Controller('books')`

```ts id="cslf07"
@Controller('books')
```

Tạo prefix `/books`.

Tất cả route trong controller này đều bắt đầu bằng:

```txt id="wsf0jq"
/books
```

---

## 9.4. API GET /books

```ts id="pejk0d"
@Get()
findAll(): Book[] {
  return this.booksService.findAll();
}
```

API này dùng để lấy toàn bộ danh sách sách.

Request:

```txt id="69ctqt"
GET /books
```

Response:

```json id="zhbsvo"
[
  {
    "id": 1,
    "title": "Clean Code",
    "author": "Robert C. Martin",
    "isbn": "9780132350884",
    "quantity": 5
  },
  {
    "id": 2,
    "title": "The Pragmatic Programmer",
    "author": "Andrew Hunt, David Thomas",
    "isbn": "9780201616224",
    "quantity": 3
  }
]
```

---

## 9.5. API GET /books/:id

```ts id="wzgesx"
@Get(':id')
findOne(@Param('id') id: string): Book {
  return this.booksService.findOne(Number(id));
}
```

API này dùng để lấy chi tiết một quyển sách.

Request:

```txt id="g6p1z8"
GET /books/1
```

Response:

```json id="sl7b9u"
{
  "id": 1,
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "9780132350884",
  "quantity": 5
}
```

Nếu gọi id không tồn tại:

```txt id="8i0hgq"
GET /books/999
```

Response:

```json id="ogaa5v"
{
  "message": "Book with id 999 not found",
  "error": "Not Found",
  "statusCode": 404
}
```

---

## 9.6. API POST /books

```ts id="jt3fyz"
@Post()
create(@Body() body: Omit<Book, 'id'>): Book {
  return this.booksService.create(body);
}
```

API này dùng để thêm sách mới.

Request:

```txt id="c158p6"
POST /books
```

Body:

```json id="wi70zw"
{
  "title": "Refactoring",
  "author": "Martin Fowler",
  "isbn": "9780201485677",
  "quantity": 4
}
```

Response:

```json id="p2lndp"
{
  "id": 3,
  "title": "Refactoring",
  "author": "Martin Fowler",
  "isbn": "9780201485677",
  "quantity": 4
}
```

---

## 9.7. API PATCH /books/:id

```ts id="q072wn"
@Patch(':id')
update(
  @Param('id') id: string,
  @Body() body: Partial<Omit<Book, 'id'>>,
): Book {
  return this.booksService.update(Number(id), body);
}
```

API này dùng để cập nhật một phần thông tin sách.

Request:

```txt id="zhzn6i"
PATCH /books/1
```

Body:

```json id="rlljfb"
{
  "quantity": 10
}
```

Response:

```json id="2mq1wk"
{
  "id": 1,
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "9780132350884",
  "quantity": 10
}
```

Điểm quan trọng của PATCH:

```txt id="l8nn1i"
PATCH không cần gửi toàn bộ object.
PATCH chỉ gửi field cần cập nhật.
```

Ví dụ chỉ đổi title:

```json id="fwv3ye"
{
  "title": "Clean Code - Updated Edition"
}
```

---

## 9.8. API DELETE /books/:id

```ts id="zywqee"
@Delete(':id')
remove(@Param('id') id: string): Book {
  return this.booksService.remove(Number(id));
}
```

API này dùng để xóa sách theo id.

Request:

```txt id="b44qv0"
DELETE /books/1
```

Response:

```json id="poho7l"
{
  "id": 1,
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "9780132350884",
  "quantity": 5
}
```

Sau khi xóa, nếu gọi lại:

```txt id="evuxz3"
GET /books/1
```

Sẽ nhận lỗi:

```json id="rgf3hy"
{
  "message": "Book with id 1 not found",
  "error": "Not Found",
  "statusCode": 404
}
```

---

# 10. Test API bằng Postman hoặc Thunder Client

## 10.1. GET /books

```txt id="q8akmu"
GET http://localhost:3000/books
```

Kết quả mong muốn:

```txt id="di7779"
Status: 200 OK
Response: danh sách books
```

---

## 10.2. GET /books/:id

```txt id="d58j38"
GET http://localhost:3000/books/1
```

Kết quả mong muốn:

```txt id="neyths"
Status: 200 OK
Response: thông tin book id = 1
```

---

## 10.3. GET /books/:id không tồn tại

```txt id="40ml12"
GET http://localhost:3000/books/999
```

Kết quả mong muốn:

```txt id="x7kvst"
Status: 404 Not Found
Response: Book with id 999 not found
```

---

## 10.4. POST /books

```txt id="yd8wjf"
POST http://localhost:3000/books
```

Body:

```json id="9dn9y1"
{
  "title": "Domain-Driven Design",
  "author": "Eric Evans",
  "isbn": "9780321125217",
  "quantity": 2
}
```

Kết quả mong muốn:

```txt id="g3qjig"
Status: 201 Created
Response: book mới được tạo
```

Lưu ý: Trong NestJS, `POST` mặc định thường trả status code `201`.

---

## 10.5. PATCH /books/:id

```txt id="3yxh9l"
PATCH http://localhost:3000/books/2
```

Body:

```json id="rndk3g"
{
  "quantity": 8
}
```

Kết quả mong muốn:

```txt id="ndk6wj"
Status: 200 OK
Response: book id = 2 có quantity mới là 8
```

---

## 10.6. DELETE /books/:id

```txt id="wdj18i"
DELETE http://localhost:3000/books/2
```

Kết quả mong muốn:

```txt id="62rba2"
Status: 200 OK
Response: book id = 2 vừa bị xóa
```

---

# 11. Test API bằng curl

## GET /books

```bash id="m30t3e"
curl http://localhost:3000/books
```

## GET /books/1

```bash id="r9xlir"
curl http://localhost:3000/books/1
```

## POST /books

```bash id="mjaz7p"
curl -X POST http://localhost:3000/books \
  -H "Content-Type: application/json" \
  -d "{\"title\":\"Domain-Driven Design\",\"author\":\"Eric Evans\",\"isbn\":\"9780321125217\",\"quantity\":2}"
```

## PATCH /books/1

```bash id="auzvmc"
curl -X PATCH http://localhost:3000/books/1 \
  -H "Content-Type: application/json" \
  -d "{\"quantity\":10}"
```

## DELETE /books/1

```bash id="zr941v"
curl -X DELETE http://localhost:3000/books/1
```

---

# 12. Vì sao Controller không nên chứa logic phức tạp?

Ta có thể viết logic trong controller như sau:

```ts id="fxa3mm"
@Get(':id')
findOne(@Param('id') id: string) {
  const book = this.books.find((item) => item.id === Number(id));

  if (!book) {
    throw new NotFoundException();
  }

  return book;
}
```

Nhưng cách này không tốt.

Vì controller sẽ bị phình to khi logic nhiều hơn.

Ví dụ sau này khi thêm nghiệp vụ:

```txt id="3eq1nt"
Kiểm tra sách có tồn tại không
Kiểm tra số lượng sách còn không
Kiểm tra user có quyền mượn sách không
Kiểm tra sách có đang bị khóa không
Ghi log lịch sử thay đổi
Gửi notification
```

Nếu viết hết trong controller, code sẽ khó đọc và khó test.

Cách tốt hơn:

```txt id="tfr0z4"
Controller chỉ nhận request và gọi service.
Service xử lý logic chính.
```

---

# 13. Vì sao nên dùng `findOne()` lại trong `update()` và `remove()`?

Trong service, ta viết:

```ts id="k0wj8c"
update(id: number, data: Partial<Omit<Book, 'id'>>): Book {
  const book = this.findOne(id);

  Object.assign(book, data);

  return book;
}
```

Và:

```ts id="emzwk5"
remove(id: number): Book {
  const book = this.findOne(id);

  this.books = this.books.filter((item) => item.id !== id);

  return book;
}
```

Lý do là để tái sử dụng logic kiểm tra sách có tồn tại hay không.

Nếu không dùng lại `findOne()`, ta phải lặp code:

```ts id="xwzgoo"
const book = this.books.find((item) => item.id === id);

if (!book) {
  throw new NotFoundException(`Book with id ${id} not found`);
}
```

Lặp code nhiều sẽ dễ sai và khó sửa.

Nguyên tắc nên nhớ:

```txt id="ca3e1a"
Logic nào dùng nhiều lần thì nên tách ra để tái sử dụng.
```

---

# 14. Vấn đề còn tồn tại sau Bài 3

API hiện tại đã CRUD được, nhưng vẫn chưa hoàn chỉnh.

Một số vấn đề còn tồn tại:

```txt id="2lnjfl"
Chưa validate dữ liệu đầu vào
Có thể gửi title rỗng
Có thể gửi quantity âm
Có thể gửi field dư
Có thể gửi quantity là string
Chưa có DTO
Chưa có database thật
Chưa có unit test
Chưa có e2e test
```

Ví dụ hiện tại client có thể gửi:

```json id="x3wv1q"
{
  "title": "",
  "author": "",
  "isbn": "abc",
  "quantity": -10
}
```

API vẫn có thể nhận.

Đây là vấn đề sẽ được xử lý ở Bài 4 bằng:

```txt id="42g2af"
DTO
class-validator
class-transformer
ValidationPipe
```

---

# 15. Checklist hoàn thành Bài 3

Bạn đạt yêu cầu Bài 3 nếu làm được:

```txt id="vafglv"
[ ] Hiểu CRUD là gì
[ ] Tạo được interface Book
[ ] Viết được BooksService.findAll()
[ ] Viết được BooksService.findOne()
[ ] Viết được BooksService.create()
[ ] Viết được BooksService.update()
[ ] Viết được BooksService.remove()
[ ] Viết được API GET /books
[ ] Viết được API GET /books/:id
[ ] Viết được API POST /books
[ ] Viết được API PATCH /books/:id
[ ] Viết được API DELETE /books/:id
[ ] Biết dùng @Param()
[ ] Biết dùng @Body()
[ ] Biết dùng NotFoundException
[ ] Test được API bằng Postman, Thunder Client hoặc curl
```

---

# 16. Bảng tổng kết decorator đã học

| Decorator              | Ví dụ             | Ý nghĩa             |
| ---------------------- | ----------------- | ------------------- |
| `@Controller('books')` | `/books`          | Prefix route        |
| `@Get()`               | `GET /books`      | Lấy danh sách       |
| `@Get(':id')`          | `GET /books/1`    | Lấy chi tiết        |
| `@Post()`              | `POST /books`     | Tạo mới             |
| `@Patch(':id')`        | `PATCH /books/1`  | Cập nhật            |
| `@Delete(':id')`       | `DELETE /books/1` | Xóa                 |
| `@Param('id')`         | `id = '1'`        | Lấy id từ URL       |
| `@Body()`              | request body      | Lấy dữ liệu gửi lên |

---

# 17. Gợi ý cấu trúc source sau Bài 3

Sau bài này, source code nên có dạng:

```txt id="1m7pso"
src/
  books/
    book.interface.ts
    books.controller.ts
    books.controller.spec.ts
    books.module.ts
    books.service.ts
    books.service.spec.ts

  app.controller.ts
  app.controller.spec.ts
  app.module.ts
  app.service.ts
  main.ts
```

---

# 18. Mini quiz kiểm tra hiểu bài

## Câu 1

CRUD là viết tắt của những thao tác nào?

A. Create, Read, Update, Delete
B. Connect, Run, Upload, Deploy
C. Controller, Route, User, Database
D. Create, Request, Update, Data

<details>
<summary>Show answer</summary>

Đáp án đúng: **A. Create, Read, Update, Delete**

Giải thích: CRUD là 4 thao tác cơ bản khi làm việc với dữ liệu.

</details>

---

## Câu 2

API nào thường dùng để lấy danh sách sách?

A. `POST /books`
B. `GET /books`
C. `PATCH /books/:id`
D. `DELETE /books/:id`

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. `GET /books`**

Giải thích: HTTP GET thường dùng để lấy dữ liệu.

</details>

---

## Câu 3

API nào thường dùng để thêm sách mới?

A. `GET /books`
B. `POST /books`
C. `PATCH /books/:id`
D. `DELETE /books/:id`

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. `POST /books`**

Giải thích: HTTP POST thường dùng để tạo dữ liệu mới.

</details>

---

## Câu 4

API nào thường dùng để cập nhật một phần thông tin sách?

A. `GET /books`
B. `POST /books`
C. `PATCH /books/:id`
D. `DELETE /books`

<details>
<summary>Show answer</summary>

Đáp án đúng: **C. `PATCH /books/:id`**

Giải thích: PATCH thường dùng để cập nhật một phần dữ liệu của resource.

</details>

---

## Câu 5

`@Param('id')` dùng để làm gì?

A. Lấy dữ liệu từ request body
B. Lấy dữ liệu từ URL
C. Lấy dữ liệu từ database
D. Tạo dữ liệu mới

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Lấy dữ liệu từ URL**

Giải thích: Với route `/books/:id`, `@Param('id')` lấy giá trị `id` từ URL.

</details>

---

## Câu 6

`@Body()` dùng để làm gì?

A. Lấy dữ liệu từ request body
B. Lấy dữ liệu từ URL
C. Lấy dữ liệu từ module
D. Lấy dữ liệu từ terminal

<details>
<summary>Show answer</summary>

Đáp án đúng: **A. Lấy dữ liệu từ request body**

Giải thích: Khi client gửi JSON lên bằng POST hoặc PATCH, ta dùng `@Body()` để lấy dữ liệu đó.

</details>

---

## Câu 7

Khi không tìm thấy book theo id, nên trả lỗi gì?

A. 200 OK
B. 201 Created
C. 404 Not Found
D. 500 Internal Server Error

<details>
<summary>Show answer</summary>

Đáp án đúng: **C. 404 Not Found**

Giải thích: Khi resource không tồn tại, API nên trả về 404 để client hiểu dữ liệu cần tìm không có.

</details>

---

## Câu 8

Trong NestJS, class nào thường dùng để trả lỗi 404?

A. `BadRequestException`
B. `NotFoundException`
C. `ForbiddenException`
D. `UnauthorizedException`

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. `NotFoundException`**

Giải thích: `NotFoundException` là exception có sẵn trong NestJS để trả HTTP status code 404.

</details>

---

## Câu 9

Vì sao nên để logic chính trong Service?

A. Để Controller ngắn gọn và dễ test hơn
B. Vì Controller không thể return dữ liệu
C. Vì Service chạy nhanh hơn database
D. Vì Module không cho viết logic

<details>
<summary>Show answer</summary>

Đáp án đúng: **A. Để Controller ngắn gọn và dễ test hơn**

Giải thích: Controller nên nhận request và gọi Service. Service nên chứa business logic chính để dễ test và dễ maintain.

</details>

---

## Câu 10

Dòng code sau có ý nghĩa gì?

```ts id="70iyco"
Object.assign(book, data);
```

A. Xóa book khỏi danh sách
B. Tạo module mới
C. Cập nhật dữ liệu từ `data` vào `book`
D. Chuyển `book` thành string

<details>
<summary>Show answer</summary>

Đáp án đúng: **C. Cập nhật dữ liệu từ `data` vào `book`**

Giải thích: `Object.assign()` copy các field trong `data` vào object `book`.

</details>

---

# 19. Ghi nhớ sau Bài 3

Sau bài này, cần nhớ 6 ý chính:

```txt id="kqbdwf"
1. CRUD gồm Create, Read, Update, Delete.
2. GET dùng để lấy dữ liệu.
3. POST dùng để tạo dữ liệu mới.
4. PATCH dùng để cập nhật dữ liệu.
5. DELETE dùng để xóa dữ liệu.
6. Không tìm thấy dữ liệu thì nên trả 404 Not Found.
```

---

# 20. Bài tập thực hành cuối buổi

Lưu ý: Bài tập dưới đây dùng dữ liệu khác với ví dụ trong bài để bạn tự thực hành lại, không copy y nguyên code mẫu.

## Bài tập 1: Tạo CRUD cho Authors

Tạo module mới tên là:

```txt id="rhop9b"
authors
```

Chạy các lệnh:

```bash id="r4slxo"
nest g module authors
nest g controller authors
nest g service authors
```

Tạo interface:

```txt id="oovsm3"
src/authors/author.interface.ts
```

Author gồm các field:

```ts id="wgu5p4"
export interface Author {
  id: number;
  name: string;
  country: string;
  birthYear: number;
}
```

Yêu cầu tạo đầy đủ API:

```txt id="9f6c61"
GET    /authors
GET    /authors/:id
POST   /authors
PATCH  /authors/:id
DELETE /authors/:id
```

Dữ liệu mẫu ban đầu:

```ts id="vo1pqm"
[
  {
    id: 1,
    name: 'Nguyễn Nhật Ánh',
    country: 'Vietnam',
    birthYear: 1955,
  },
  {
    id: 2,
    name: 'J. K. Rowling',
    country: 'United Kingdom',
    birthYear: 1965,
  },
]
```

Yêu cầu xử lý lỗi:

```txt id="4gzsa8"
Nếu author không tồn tại, trả 404 Not Found
Message lỗi: Author with id <id> not found
```

---

## Bài tập 2: Tạo CRUD cho Categories

Tạo module mới tên là:

```txt id="ka7yxv"
categories
```

Category gồm:

```ts id="d2c3ci"
export interface Category {
  id: number;
  name: string;
  description: string;
}
```

API cần làm:

```txt id="83tvzn"
GET    /categories
GET    /categories/:id
POST   /categories
PATCH  /categories/:id
DELETE /categories/:id
```

Dữ liệu mẫu ban đầu:

```ts id="l2mftv"
[
  {
    id: 1,
    name: 'Programming',
    description: 'Books about software development',
  },
  {
    id: 2,
    name: 'Novel',
    description: 'Fiction and literature books',
  },
]
```

Yêu cầu:

```txt id="st88oy"
Controller chỉ gọi service
Service xử lý logic
Nếu category không tồn tại, trả 404
```

---

## Bài tập 3: Test API bằng Postman hoặc Thunder Client

Sau khi làm xong `authors`, test các API sau:

```txt id="u2oyr2"
GET    http://localhost:3000/authors
GET    http://localhost:3000/authors/1
POST   http://localhost:3000/authors
PATCH  http://localhost:3000/authors/1
DELETE http://localhost:3000/authors/1
```

Body cho `POST /authors`:

```json id="7u0g60"
{
  "name": "George Orwell",
  "country": "United Kingdom",
  "birthYear": 1903
}
```

Body cho `PATCH /authors/1`:

```json id="4iuqvg"
{
  "country": "Vietnam"
}
```

---

## Bài tập 4: Kiểm tra lỗi 404

Gọi API với id không tồn tại:

```txt id="uakxmy"
GET http://localhost:3000/authors/999
```

Kết quả mong muốn:

```txt id="v8j45k"
Status: 404 Not Found
```

Response mong muốn:

```json id="yzjg4k"
{
  "message": "Author with id 999 not found",
  "error": "Not Found",
  "statusCode": 404
}
```

Làm tương tự với:

```txt id="ofxrdp"
PATCH /authors/999
DELETE /authors/999
GET /categories/999
PATCH /categories/999
DELETE /categories/999
```

---

## Bài tập 5: Nâng cao 1 - Không cho cập nhật id

Hiện tại nếu client gửi PATCH như sau:

```json id="inxr3l"
{
  "id": 100,
  "name": "Updated Name"
}
```

Có thể làm thay đổi `id` nếu code không kiểm soát kỹ.

Yêu cầu nâng cao:

```txt id="ctk4xz"
Không cho phép cập nhật field id
Nếu body có id thì bỏ qua id
Chỉ cho cập nhật name, country, birthYear
```

Gợi ý:

```ts id="jjujs1"
const { id: _ignoredId, ...updateData } = data;
Object.assign(author, updateData);
```

Hoặc tự tạo type không chứa `id`:

```ts id="zl01l0"
Partial<Omit<Author, 'id'>>
```

---

## Bài tập 6: Nâng cao 2 - Tạo id an toàn hơn

Trong ví dụ bài học, ta tạo id bằng:

```ts id="4y021z"
id: this.books.length + 1
```

Cách này có vấn đề.

Ví dụ:

```txt id="b2xtlz"
Ban đầu có id 1, 2, 3
Xóa item id 3
Thêm item mới
length + 1 lại tạo id 3
```

Yêu cầu nâng cao:

```txt id="as3k16"
Tạo id mới bằng cách lấy id lớn nhất hiện tại + 1
Nếu danh sách rỗng thì id mới là 1
```

Gợi ý:

```ts id="tqi7x1"
const nextId =
  this.authors.length > 0
    ? Math.max(...this.authors.map((author) => author.id)) + 1
    : 1;
```

Áp dụng logic này cho cả:

```txt id="vq7e2r"
AuthorsService.create()
CategoriesService.create()
```

---

## Bài tập 7: Tự kiểm tra lại code

Sau khi hoàn thành, tự kiểm tra checklist:

```txt id="l245t7"
[ ] AuthorsModule hoạt động
[ ] AuthorsController có đủ 5 API
[ ] AuthorsService có đủ 5 method
[ ] CategoriesModule hoạt động
[ ] CategoriesController có đủ 5 API
[ ] CategoriesService có đủ 5 method
[ ] GET danh sách trả 200
[ ] GET theo id tồn tại trả 200
[ ] GET theo id không tồn tại trả 404
[ ] POST tạo mới thành công
[ ] PATCH cập nhật thành công
[ ] PATCH id không tồn tại trả 404
[ ] DELETE xóa thành công
[ ] DELETE id không tồn tại trả 404
[ ] Không cho cập nhật id
[ ] Tạo id mới không bị trùng sau khi xóa item
```

---

# 21. Chuẩn bị cho Bài 4

Sau Bài 3, API đã CRUD được nhưng vẫn chưa validate dữ liệu đầu vào.

Ví dụ hiện tại client vẫn có thể gửi dữ liệu sai:

```json id="d6zsu9"
{
  "title": "",
  "author": "",
  "isbn": "",
  "quantity": -5
}
```

Hoặc gửi field dư:

```json id="bbg61b"
{
  "title": "Some Book",
  "author": "Someone",
  "isbn": "123",
  "quantity": 1,
  "randomField": "should not be accepted"
}
```

Ở Bài 4, ta sẽ học:

```txt id="1q5bbm"
DTO
class-validator
class-transformer
ValidationPipe
```

Mục tiêu là API phải tự động chặn request sai và trả lỗi:

```txt id="fxc3qw"
400 Bad Request
```
