# Bài 4: DTO và ValidationPipe

## 1. Mục tiêu bài học

Sau buổi này, bạn cần làm được:

1. Hiểu DTO là gì.
2. Biết vì sao cần validate request body.
3. Cài được `class-validator` và `class-transformer`.
4. Cấu hình được `ValidationPipe` global trong `main.ts`.
5. Tạo được `CreateBookDto`.
6. Áp dụng DTO cho API `POST /books`.
7. Biết chặn request sai trước khi vào Service.
8. Biết reject field dư bằng `forbidNonWhitelisted`.

Sau bài này, API `POST /books` phải xử lý được các case:

```txt id="z7gfd4"
Thiếu title       → 400 Bad Request
quantity < 0      → 400 Bad Request
quantity sai kiểu → 400 Bad Request
Gửi field dư      → 400 Bad Request
Body hợp lệ       → 201 Created
```

---

# 2. Ôn lại Bài 3

Ở Bài 3, chúng ta đã tạo CRUD API cho Books:

```txt id="ma6kj8"
GET    /books
GET    /books/:id
POST   /books
PATCH  /books/:id
DELETE /books/:id
```

Service hiện tại có thể thêm sách mới:

```ts id="h31gmr"
create(data: Omit<Book, 'id'>): Book {
  const newBook: Book = {
    id: this.books.length + 1,
    ...data,
  };

  this.books.push(newBook);

  return newBook;
}
```

Controller nhận body từ client:

```ts id="0hpgqi"
@Post()
create(@Body() body: Omit<Book, 'id'>): Book {
  return this.booksService.create(body);
}
```

API chạy được, nhưng vẫn có vấn đề lớn: chưa kiểm tra dữ liệu đầu vào.

---

# 3. Vấn đề khi chưa validate dữ liệu

Hiện tại client có thể gửi body sai như sau:

```json id="sgrmjp"
{
  "title": "",
  "author": "",
  "isbn": "",
  "quantity": -10
}
```

Hoặc gửi thiếu field:

```json id="kxzsf3"
{
  "author": "Robert C. Martin",
  "isbn": "123",
  "quantity": 5
}
```

Hoặc gửi sai kiểu dữ liệu:

```json id="a8j5g6"
{
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "123",
  "quantity": "five"
}
```

Hoặc gửi field dư:

```json id="tt0hug"
{
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "123",
  "quantity": 5,
  "isAdmin": true
}
```

Nếu API không validate, dữ liệu xấu có thể đi vào Service.

Điều này nguy hiểm vì:

```txt id="xemfmw"
Dữ liệu trong hệ thống bị sai
Logic xử lý dễ lỗi
Database sau này có thể lưu dữ liệu không hợp lệ
Test khó ổn định
API thiếu chuyên nghiệp
```

Vì vậy ta cần DTO và ValidationPipe.

---

# 4. DTO là gì?

DTO là viết tắt của:

```txt id="u6mod9"
Data Transfer Object
```

Hiểu đơn giản:

```txt id="2t8j3e"
DTO là class mô tả dữ liệu được phép gửi vào API.
```

Ví dụ khi tạo sách, client chỉ được gửi:

```txt id="vbrcae"
title
author
isbn
quantity
```

Ta sẽ tạo class:

```ts id="rfh26n"
export class CreateBookDto {
  title: string;
  author: string;
  isbn: string;
  quantity: number;
}
```

Nhưng chỉ khai báo như vậy chưa đủ.

Ta cần thêm rule validate:

```ts id="l1fk2j"
@IsString()
@IsNotEmpty()
title: string;
```

Nghĩa là:

```txt id="aaqjpk"
title phải là string
title không được rỗng
```

---

# 5. DTO khác Interface như thế nào?

Ở Bài 3, ta có interface:

```ts id="ue1bn7"
export interface Book {
  id: number;
  title: string;
  author: string;
  isbn: string;
  quantity: number;
}
```

Interface chủ yếu dùng cho TypeScript kiểm tra kiểu trong lúc code.

Nhưng interface không tồn tại khi app chạy.

DTO là class, có thể dùng được khi runtime để validate dữ liệu.

So sánh nhanh:

| Tiêu chí                         | Interface | DTO Class |
| -------------------------------- | --------- | --------- |
| Dùng để type checking            | Có        | Có        |
| Tồn tại khi runtime              | Không     | Có        |
| Dùng được với decorator validate | Không     | Có        |
| Phù hợp validate request         | Không nên | Nên dùng  |

Vì vậy:

```txt id="dhh73j"
Interface dùng để mô tả dữ liệu trong code.
DTO dùng để mô tả và validate dữ liệu request.
```

---

# 6. Cài package validate

Trong NestJS, validate thường dùng 2 package:

```txt id="a312n2"
class-validator
class-transformer
```

Cài bằng lệnh:

```bash id="jl3byr"
npm i class-validator class-transformer
```

Giải thích:

```txt id="vfrp6f"
class-validator   cung cấp decorator như @IsString(), @IsInt(), @Min()
class-transformer hỗ trợ chuyển đổi dữ liệu khi validate
```

---

# 7. Cấu hình ValidationPipe global

Mở file:

```txt id="cu1xfz"
src/main.ts
```

Hiện tại có thể đang như sau:

```ts id="4l3okh"
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}

bootstrap();
```

Sửa thành:

```ts id="zkfxpn"
import { ValidationPipe } from '@nestjs/common';
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,
      transform: true,
      forbidNonWhitelisted: true,
    }),
  );

  await app.listen(3000);
}

bootstrap();
```

---

# 8. Giải thích ValidationPipe

## 8.1. `ValidationPipe` là gì?

`ValidationPipe` là cơ chế của NestJS dùng để kiểm tra dữ liệu request trước khi request đi vào controller hoặc service.

Luồng xử lý sau khi có ValidationPipe:

```txt id="vajpp6"
Client gửi request
        ↓
ValidationPipe kiểm tra body
        ↓
Nếu sai → trả 400 Bad Request
        ↓
Nếu đúng → Controller
        ↓
Service
        ↓
Response
```

Điểm quan trọng:

```txt id="yfv7g8"
Request sai sẽ bị chặn trước khi vào Service.
```

Nhờ vậy Service không cần xử lý quá nhiều dữ liệu bẩn.

---

## 8.2. `whitelist: true`

```ts id="ssxziu"
whitelist: true
```

Ý nghĩa:

```txt id="aub649"
Chỉ giữ lại những field có khai báo trong DTO.
```

Ví dụ DTO chỉ có:

```txt id="esbwh2"
title
author
isbn
quantity
```

Client gửi thêm:

```json id="7zi7as"
{
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "123",
  "quantity": 5,
  "isAdmin": true
}
```

Nếu chỉ dùng `whitelist: true`, field `isAdmin` sẽ bị loại bỏ.

---

## 8.3. `forbidNonWhitelisted: true`

```ts id="tjoiab"
forbidNonWhitelisted: true
```

Ý nghĩa:

```txt id="38yd4p"
Nếu client gửi field không có trong DTO, API sẽ trả lỗi 400.
```

Ví dụ gửi:

```json id="55ibyb"
{
  "title": "Clean Code",
  "author": "Robert C. Martin",
  "isbn": "123",
  "quantity": 5,
  "isAdmin": true
}
```

Response mong muốn:

```json id="vqb0zu"
{
  "message": [
    "property isAdmin should not exist"
  ],
  "error": "Bad Request",
  "statusCode": 400
}
```

Khi học API cơ bản, nên bật option này để tránh client gửi dữ liệu không mong muốn.

---

## 8.4. `transform: true`

```ts id="60vmkp"
transform: true
```

Ý nghĩa:

```txt id="b0xjyr"
Cho phép NestJS chuyển đổi dữ liệu dựa trên DTO.
```

Ví dụ sau này khi dùng:

```ts id="23d358"
@Type(() => Number)
quantity: number;
```

`class-transformer` có thể hỗ trợ chuyển chuỗi thành số trong một số tình huống.

Tuy nhiên trong bài này, để tránh nhập sai kiểu dữ liệu, ta vẫn nên yêu cầu client gửi đúng JSON number:

```json id="z51srx"
{
  "quantity": 5
}
```

Không nên gửi:

```json id="bzqaee"
{
  "quantity": "5"
}
```

---

# 9. Tạo thư mục DTO

Trong thư mục:

```txt id="m52010"
src/books/
```

Tạo thêm thư mục:

```txt id="qpzlpm"
dto
```

Cấu trúc mong muốn:

```txt id="yx8asj"
src/
  books/
    dto/
      create-book.dto.ts
    book.interface.ts
    books.controller.ts
    books.module.ts
    books.service.ts
```

---

# 10. Tạo CreateBookDto

Tạo file:

```txt id="cq76gy"
src/books/dto/create-book.dto.ts
```

Nội dung:

```ts id="nyclcq"
import { IsInt, IsNotEmpty, IsString, Min } from 'class-validator';

export class CreateBookDto {
  @IsString()
  @IsNotEmpty()
  title: string;

  @IsString()
  @IsNotEmpty()
  author: string;

  @IsString()
  @IsNotEmpty()
  isbn: string;

  @IsInt()
  @Min(0)
  quantity: number;
}
```

---

# 11. Giải thích CreateBookDto

## 11.1. `@IsString()`

```ts id="ckj2lc"
@IsString()
title: string;
```

Ý nghĩa:

```txt id="tlu554"
Field này phải là string.
```

Ví dụ hợp lệ:

```json id="c5z7vn"
{
  "title": "Clean Code"
}
```

Không hợp lệ:

```json id="hg824e"
{
  "title": 123
}
```

---

## 11.2. `@IsNotEmpty()`

```ts id="8z3ku9"
@IsNotEmpty()
title: string;
```

Ý nghĩa:

```txt id="ih95m7"
Field này không được rỗng.
```

Không hợp lệ:

```json id="iduy7q"
{
  "title": ""
}
```

Nếu thiếu luôn field `title`, cũng không hợp lệ.

---

## 11.3. `@IsInt()`

```ts id="0zll78"
@IsInt()
quantity: number;
```

Ý nghĩa:

```txt id="l3salw"
quantity phải là số nguyên.
```

Hợp lệ:

```json id="zz6awe"
{
  "quantity": 5
}
```

Không hợp lệ:

```json id="k61j9w"
{
  "quantity": 5.5
}
```

Không hợp lệ:

```json id="ubpbo7"
{
  "quantity": "5"
}
```

---

## 11.4. `@Min(0)`

```ts id="0ybu9i"
@Min(0)
quantity: number;
```

Ý nghĩa:

```txt id="ju2hsa"
quantity phải lớn hơn hoặc bằng 0.
```

Hợp lệ:

```json id="syrm6n"
{
  "quantity": 0
}
```

Hợp lệ:

```json id="8gr00c"
{
  "quantity": 10
}
```

Không hợp lệ:

```json id="sywvmc"
{
  "quantity": -1
}
```

---

# 12. Cập nhật BooksController dùng DTO

Mở file:

```txt id="ganz58"
src/books/books.controller.ts
```

Trước đó method `create()` có thể đang như sau:

```ts id="9mzbfi"
@Post()
create(@Body() body: Omit<Book, 'id'>): Book {
  return this.booksService.create(body);
}
```

Bây giờ sửa lại dùng `CreateBookDto`.

Code đầy đủ:

```ts id="d790n0"
import { Body, Controller, Delete, Get, Param, Patch, Post } from '@nestjs/common';
import { Book } from './book.interface';
import { BooksService } from './books.service';
import { CreateBookDto } from './dto/create-book.dto';

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
  create(@Body() body: CreateBookDto): Book {
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

Điểm thay đổi quan trọng:

```ts id="w291y2"
create(@Body() body: CreateBookDto): Book {
  return this.booksService.create(body);
}
```

Từ giờ body của `POST /books` phải đi qua rule validate trong `CreateBookDto`.

---

# 13. Cập nhật BooksService cho rõ type

Mở file:

```txt id="rzy3xf"
src/books/books.service.ts
```

Import DTO:

```ts id="kpeoh1"
import { CreateBookDto } from './dto/create-book.dto';
```

Sửa method `create()`:

```ts id="y3ismz"
create(data: CreateBookDto): Book {
  const newBook: Book = {
    id: this.books.length + 1,
    ...data,
  };

  this.books.push(newBook);

  return newBook;
}
```

Code đầy đủ phần đầu file:

```ts id="iwwp9h"
import { Injectable, NotFoundException } from '@nestjs/common';
import { Book } from './book.interface';
import { CreateBookDto } from './dto/create-book.dto';

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

  create(data: CreateBookDto): Book {
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

# 14. Test case 1: Body hợp lệ

Gửi request:

```txt id="kpi7ah"
POST http://localhost:3000/books
```

Body:

```json id="xgk2ga"
{
  "title": "Refactoring",
  "author": "Martin Fowler",
  "isbn": "9780201485677",
  "quantity": 4
}
```

Kết quả mong muốn:

```txt id="dpdksk"
Status: 201 Created
```

Response:

```json id="ggu8nl"
{
  "id": 3,
  "title": "Refactoring",
  "author": "Martin Fowler",
  "isbn": "9780201485677",
  "quantity": 4
}
```

---

# 15. Test case 2: Thiếu title

Gửi body:

```json id="w05zg0"
{
  "author": "Martin Fowler",
  "isbn": "9780201485677",
  "quantity": 4
}
```

Kết quả mong muốn:

```txt id="uu3553"
Status: 400 Bad Request
```

Response có thể gần giống:

```json id="b6s3is"
{
  "message": [
    "title should not be empty",
    "title must be a string"
  ],
  "error": "Bad Request",
  "statusCode": 400
}
```

Ý nghĩa:

```txt id="ta5vgo"
Request bị chặn trước khi vào BooksService.create().
```

---

# 16. Test case 3: title rỗng

Gửi body:

```json id="fbdnlk"
{
  "title": "",
  "author": "Martin Fowler",
  "isbn": "9780201485677",
  "quantity": 4
}
```

Kết quả mong muốn:

```txt id="nizbrf"
Status: 400 Bad Request
```

Vì `title` vi phạm rule:

```ts id="yxhhvr"
@IsNotEmpty()
```

---

# 17. Test case 4: quantity âm

Gửi body:

```json id="q8fslf"
{
  "title": "Refactoring",
  "author": "Martin Fowler",
  "isbn": "9780201485677",
  "quantity": -1
}
```

Kết quả mong muốn:

```txt id="smh7lv"
Status: 400 Bad Request
```

Vì `quantity` vi phạm rule:

```ts id="xz1rba"
@Min(0)
```

---

# 18. Test case 5: quantity sai kiểu

Gửi body:

```json id="y5g6t6"
{
  "title": "Refactoring",
  "author": "Martin Fowler",
  "isbn": "9780201485677",
  "quantity": "4"
}
```

Kết quả mong muốn:

```txt id="zhtg27"
Status: 400 Bad Request
```

Vì `quantity` phải là number, không phải string.

Body đúng phải là:

```json id="ulh4w7"
{
  "quantity": 4
}
```

Không phải:

```json id="g0y54f"
{
  "quantity": "4"
}
```

---

# 19. Test case 6: Gửi field dư

Gửi body:

```json id="qh74p2"
{
  "title": "Refactoring",
  "author": "Martin Fowler",
  "isbn": "9780201485677",
  "quantity": 4,
  "isAdmin": true
}
```

Kết quả mong muốn:

```txt id="xv61g5"
Status: 400 Bad Request
```

Response có thể gần giống:

```json id="j7tqoh"
{
  "message": [
    "property isAdmin should not exist"
  ],
  "error": "Bad Request",
  "statusCode": 400
}
```

Lý do là ta đã bật:

```ts id="fxd0ag"
forbidNonWhitelisted: true
```

---

# 20. Bài tập thực hành cuối buổi

Lưu ý: Bài tập dưới đây dùng dữ liệu khác với ví dụ chính trong bài để bạn tự áp dụng lại DTO và ValidationPipe, không copy y nguyên `CreateBookDto`.

## Bài tập 1: Tạo DTO cho Authors

Ở Bài 3, bạn đã có thể tạo module `authors`.

Bây giờ tạo thư mục:

```txt id="e8nd7c"
src/authors/dto/
```

Tạo file:

```txt id="iod8qf"
src/authors/dto/create-author.dto.ts
```

Author khi tạo mới gồm:

```txt id="hnw93d"
name
country
birthYear
```

Yêu cầu validate:

```txt id="vqqet9"
name phải là string
name không được rỗng
country phải là string
country không được rỗng
birthYear phải là số nguyên
birthYear phải >= 0
```

Gợi ý decorator cần dùng:

```ts id="k7chm0"
import { IsInt, IsNotEmpty, IsString, Min } from 'class-validator';
```

---

## Bài tập 2: Áp dụng CreateAuthorDto cho POST /authors

Trong `AuthorsController`, sửa method `create()` để dùng DTO:

```ts id="p693gm"
@Post()
create(@Body() body: CreateAuthorDto) {
  return this.authorsService.create(body);
}
```

Trong `AuthorsService`, sửa method `create()` để nhận `CreateAuthorDto`.

Yêu cầu test:

```txt id="eouyyg"
POST /authors body hợp lệ → 201
POST /authors thiếu name → 400
POST /authors name rỗng → 400
POST /authors birthYear âm → 400
POST /authors có field dư → 400
```

---

## Bài tập 3: Tạo DTO cho Categories

Tạo file:

```txt id="2zi1kx"
src/categories/dto/create-category.dto.ts
```

Category khi tạo mới gồm:

```txt id="d50z0h"
name
description
```

Yêu cầu validate:

```txt id="81bpfi"
name phải là string
name không được rỗng
description phải là string
description không được rỗng
```

Test với body hợp lệ:

```json id="f9nyxx"
{
  "name": "Programming",
  "description": "Books about software development"
}
```

Test với body sai:

```json id="wot2al"
{
  "name": "",
  "description": "Books about software development"
}
```

Kết quả mong muốn:

```txt id="g123u3"
Status: 400 Bad Request
```

---

## Bài tập 4: Kiểm tra field dư

Gửi request:

```txt id="td51xr"
POST http://localhost:3000/categories
```

Body:

```json id="jgy9sg"
{
  "name": "Novel",
  "description": "Fiction books",
  "isHidden": true
}
```

Kết quả mong muốn:

```txt id="fyti2o"
Status: 400 Bad Request
```

Vì `isHidden` không được khai báo trong DTO.

---

## Bài tập 5: Nâng cao 1 - Tạo Update DTO

Hiện tại ta mới validate `POST`.

Hãy tạo thêm DTO cho update:

```txt id="co13qj"
src/books/dto/update-book.dto.ts
```

Yêu cầu:

```txt id="if1npx"
Khi PATCH /books/:id, cho phép gửi một hoặc nhiều field
Không bắt buộc gửi đủ title, author, isbn, quantity
Nếu gửi title thì phải là string và không rỗng
Nếu gửi author thì phải là string và không rỗng
Nếu gửi isbn thì phải là string và không rỗng
Nếu gửi quantity thì phải là số nguyên >= 0
Không cho gửi field dư
```

Gợi ý dùng `@IsOptional()`:

```ts id="f14nl5"
import { IsInt, IsNotEmpty, IsOptional, IsString, Min } from 'class-validator';

export class UpdateBookDto {
  @IsOptional()
  @IsString()
  @IsNotEmpty()
  title?: string;

  @IsOptional()
  @IsString()
  @IsNotEmpty()
  author?: string;

  @IsOptional()
  @IsString()
  @IsNotEmpty()
  isbn?: string;

  @IsOptional()
  @IsInt()
  @Min(0)
  quantity?: number;
}
```

Sau đó sửa controller:

```ts id="tbxi4q"
@Patch(':id')
update(
  @Param('id') id: string,
  @Body() body: UpdateBookDto,
): Book {
  return this.booksService.update(Number(id), body);
}
```

Test các case:

```txt id="fw0nlw"
PATCH /books/1 với quantity hợp lệ → 200
PATCH /books/1 với quantity âm → 400
PATCH /books/1 với title rỗng → 400
PATCH /books/1 với field dư → 400
PATCH /books/999 với body hợp lệ → 404
```

---

## Bài tập 6: Nâng cao 2 - Validate id trên URL

Hiện tại trong controller, ta đang dùng:

```ts id="wlvfl3"
Number(id)
```

Nếu client gọi:

```txt id="a7cxk1"
GET /books/abc
```

Thì `Number('abc')` sẽ thành `NaN`.

Yêu cầu nâng cao:

```txt id="fp7v5p"
Nếu id trên URL không phải số, API nên trả 400 Bad Request.
```

Gợi ý dùng `ParseIntPipe`:

```ts id="ni36or"
import { ParseIntPipe } from '@nestjs/common';
```

Sửa:

```ts id="yo9au8"
@Get(':id')
findOne(@Param('id') id: string): Book {
  return this.booksService.findOne(Number(id));
}
```

Thành:

```ts id="vncr99"
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number): Book {
  return this.booksService.findOne(id);
}
```

Áp dụng tương tự cho:

```txt id="qhpy4e"
GET /books/:id
PATCH /books/:id
DELETE /books/:id
```

Test:

```txt id="rdh231"
GET /books/abc → 400 Bad Request
GET /books/999 → 404 Not Found
GET /books/1   → 200 OK
```

Điểm khác nhau cần nhớ:

```txt id="2ob6ii"
id sai kiểu       → 400 Bad Request
id đúng kiểu nhưng không tồn tại → 404 Not Found
```

---

## Bài tập 7: Checklist tự kiểm tra

Sau khi hoàn thành, tự kiểm tra:

```txt id="vj0cy8"
[ ] Đã cài class-validator
[ ] Đã cài class-transformer
[ ] Đã cấu hình ValidationPipe trong main.ts
[ ] Đã bật whitelist: true
[ ] Đã bật transform: true
[ ] Đã bật forbidNonWhitelisted: true
[ ] Đã tạo CreateBookDto
[ ] POST /books body hợp lệ trả 201
[ ] POST /books thiếu title trả 400
[ ] POST /books quantity âm trả 400
[ ] POST /books field dư trả 400
[ ] Đã tạo CreateAuthorDto
[ ] Đã tạo CreateCategoryDto
[ ] Đã thử ít nhất 5 request sai bằng Postman hoặc Thunder Client
[ ] Hoàn thành UpdateBookDto nâng cao
[ ] Hoàn thành ParseIntPipe nâng cao
```

---

# 21. Mini quiz kiểm tra hiểu bài

## Câu 1

DTO là viết tắt của gì?

A. Data Transfer Object
B. Database Type Object
C. Dynamic Test Output
D. Data Table Option

<details>
<summary>Show answer</summary>

Đáp án đúng: **A. Data Transfer Object**

Giải thích: DTO dùng để mô tả dữ liệu được truyền vào hoặc truyền ra khỏi API.

</details>

---

## Câu 2

Trong NestJS, DTO thường được tạo bằng gì?

A. Interface
B. Class
C. Function
D. JSON file

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Class**

Giải thích: DTO nên dùng class để có thể sử dụng decorator validate như `@IsString()` và `@IsNotEmpty()`.

</details>

---

## Câu 3

Package nào cung cấp decorator như `@IsString()`, `@IsInt()`, `@Min()`?

A. `class-validator`
B. `class-transformer`
C. `express`
D. `typeorm`

<details>
<summary>Show answer</summary>

Đáp án đúng: **A. `class-validator`**

Giải thích: `class-validator` cung cấp các decorator dùng để kiểm tra dữ liệu.

</details>

---

## Câu 4

`ValidationPipe` dùng để làm gì?

A. Tạo database
B. Validate request trước khi vào controller/service
C. Chạy unit test
D. Tạo module mới

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Validate request trước khi vào controller/service**

Giải thích: `ValidationPipe` giúp chặn request sai và trả lỗi `400 Bad Request`.

</details>

---

## Câu 5

`@IsNotEmpty()` dùng để làm gì?

A. Kiểm tra field là number
B. Kiểm tra field không được rỗng
C. Kiểm tra field là array
D. Kiểm tra field là boolean

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Kiểm tra field không được rỗng**

Giải thích: `@IsNotEmpty()` giúp chặn giá trị rỗng như `""`.

</details>

---

## Câu 6

`@Min(0)` trên field `quantity` có ý nghĩa gì?

A. `quantity` phải nhỏ hơn 0
B. `quantity` phải là string
C. `quantity` phải lớn hơn hoặc bằng 0
D. `quantity` không được tồn tại

<details>
<summary>Show answer</summary>

Đáp án đúng: **C. `quantity` phải lớn hơn hoặc bằng 0**

Giải thích: `@Min(0)` chặn các giá trị âm như `-1`, `-5`.

</details>

---

## Câu 7

Option nào giúp reject field dư trong request body?

A. `transform: true`
B. `whitelist: false`
C. `forbidNonWhitelisted: true`
D. `ignoreExtraFields: true`

<details>
<summary>Show answer</summary>

Đáp án đúng: **C. `forbidNonWhitelisted: true`**

Giải thích: Khi bật option này, nếu client gửi field không khai báo trong DTO, API sẽ trả `400 Bad Request`.

</details>

---

## Câu 8

Nếu client gửi `quantity: -5`, API nên trả status code nào?

A. 200
B. 201
C. 400
D. 404

<details>
<summary>Show answer</summary>

Đáp án đúng: **C. 400**

Giải thích: Đây là lỗi dữ liệu đầu vào, nên API trả `400 Bad Request`.

</details>

---

## Câu 9

Nếu client gọi `GET /books/999` nhưng book không tồn tại, API nên trả status code nào?

A. 400
B. 401
C. 404
D. 500

<details>
<summary>Show answer</summary>

Đáp án đúng: **C. 404**

Giải thích: ID hợp lệ về kiểu dữ liệu nhưng resource không tồn tại, nên trả `404 Not Found`.

</details>

---

## Câu 10

Nếu client gọi `GET /books/abc` và ta dùng `ParseIntPipe`, API nên trả gì?

A. 200 OK
B. 201 Created
C. 400 Bad Request
D. 404 Not Found

<details>
<summary>Show answer</summary>

Đáp án đúng: **C. 400 Bad Request**

Giải thích: `abc` không thể parse thành number, nên đây là lỗi request sai định dạng.

</details>

---

# 22. Ghi nhớ sau Bài 4

Sau bài này, cần nhớ 7 ý chính:

```txt id="yjav9t"
1. DTO dùng để mô tả dữ liệu request.
2. DTO nên viết bằng class.
3. class-validator dùng để validate field.
4. ValidationPipe dùng để kích hoạt validate trong NestJS.
5. whitelist giúp chỉ nhận field có trong DTO.
6. forbidNonWhitelisted giúp reject field dư.
7. Dữ liệu sai đầu vào nên trả 400 Bad Request.
```

---

# 23. Chuẩn bị cho Bài 5

Sau Bài 4, API đã có validate đầu vào.

Từ Bài 5, ta bắt đầu học testing.

Nội dung chính của Bài 5:

```txt id="yl7kdn"
Jest là gì
Unit test là gì
File .spec.ts là gì
describe, it, expect
TestingModule trong NestJS
Test service độc lập
```

Mục tiêu sắp tới:

```txt id="a788zq"
Viết unit test cho BooksService
Không gọi HTTP thật
Không dùng database thật
Kiểm tra logic chạy đúng bằng Jest
```
