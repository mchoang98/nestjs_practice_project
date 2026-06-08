# GIÁO TRÌNH NESTJS CƠ BẢN & UNIT TEST API

## 1. Thông tin khóa học

**Tên khóa học:** NestJS cơ bản và kiểm thử API
**Thời lượng đề xuất:** 6 tuần
**Tần suất học:** 3 buổi/tuần
**Thời lượng mỗi buổi:** 1.5 – 2 giờ
**Đối tượng:** Lập trình viên backend đã biết JavaScript/TypeScript cơ bản
**Project thực hành:** Library API – hệ thống quản lý sách và mượn/trả sách

---

## 2. Mục tiêu khóa học

Sau khi hoàn thành giáo trình, học viên có thể:

1. Hiểu kiến trúc cơ bản của NestJS.
2. Tạo REST API bằng Controller, Service, Module.
3. Sử dụng DTO để validate request.
4. Tách business logic để dễ bảo trì và dễ test.
5. Viết unit test cho Service.
6. Viết unit test cho Controller.
7. Viết e2e test để kiểm tra API thật.
8. Mock database/repository khi viết test.
9. Biết cách tổ chức source code NestJS sạch và dễ mở rộng.

---

## 3. Kiến thức cần có trước khi học

Học viên nên biết trước:

* JavaScript ES6+
* TypeScript cơ bản
* REST API
* HTTP method: GET, POST, PATCH, DELETE
* HTTP status code: 200, 201, 400, 401, 403, 404, 500
* npm hoặc yarn
* Git cơ bản

---

# PHẦN 1: LÀM QUEN NESTJS

## Buổi 1: NestJS là gì? Cài đặt môi trường

### Mục tiêu

* Hiểu NestJS dùng để làm gì.
* Tạo được project NestJS đầu tiên.
* Chạy được server local.
* Hiểu sơ bộ cấu trúc thư mục.

### Nội dung

1. NestJS là framework backend Node.js.
2. NestJS dùng TypeScript.
3. NestJS tổ chức code theo module.
4. So sánh nhanh NestJS với Express.
5. Cài NestJS CLI.
6. Tạo project đầu tiên.

### Thực hành

Cài NestJS CLI:

```bash
npm i -g @nestjs/cli
```

Tạo project:

```bash
nest new nest-library-api
cd nest-library-api
npm run start:dev
```

Mở trình duyệt:

```txt
http://localhost:3000
```

### Bài tập

1. Tạo project NestJS mới.
2. Chạy server thành công.
3. Đọc các file:

   * `main.ts`
   * `app.module.ts`
   * `app.controller.ts`
   * `app.service.ts`

---

## Buổi 2: Controller, Service, Module

### Mục tiêu

* Hiểu vai trò của Controller.
* Hiểu vai trò của Service.
* Hiểu vai trò của Module.
* Biết tạo module bằng CLI.

### Nội dung

1. Controller nhận request.
2. Service xử lý business logic.
3. Module gom nhóm các thành phần liên quan.
4. Dependency Injection trong NestJS.
5. Luồng xử lý request:

```txt
Client → Controller → Service → Response
```

### Thực hành

Tạo module books:

```bash
nest g module books
nest g controller books
nest g service books
```

Tạo API đơn giản:

```txt
GET /books
GET /books/:id
POST /books
```

Dữ liệu tạm thời lưu bằng array trong service.

### Bài tập

Tạo `BooksService` có các method:

```ts
findAll()
findOne(id: number)
create(data)
```

---

# PHẦN 2: XÂY DỰNG API CƠ BẢN

## Buổi 3: Tạo CRUD API cho Books

### Mục tiêu

* Tạo được CRUD API cơ bản.
* Biết dùng params và body.
* Biết trả lỗi khi không tìm thấy dữ liệu.

### Nội dung

1. `@Get()`
2. `@Post()`
3. `@Patch()`
4. `@Delete()`
5. `@Param()`
6. `@Body()`
7. `NotFoundException`

### API cần hoàn thành

```txt
GET    /books
GET    /books/:id
POST   /books
PATCH  /books/:id
DELETE /books/:id
```

### Model tạm thời

```ts
export interface Book {
  id: number;
  title: string;
  author: string;
  isbn: string;
  quantity: number;
}
```

### Bài tập

1. Tạo API thêm sách.
2. Tạo API lấy danh sách sách.
3. Tạo API tìm sách theo id.
4. Tạo API cập nhật sách.
5. Tạo API xóa sách.
6. Nếu không tìm thấy sách, trả lỗi 404.

---

## Buổi 4: DTO và ValidationPipe

### Mục tiêu

* Biết DTO là gì.
* Biết validate dữ liệu đầu vào.
* Biết ngăn request sai trước khi vào service.

### Nội dung

1. DTO là gì?
2. `class-validator`
3. `class-transformer`
4. `ValidationPipe`
5. Validate string, number, required field.

### Cài package

```bash
npm i class-validator class-transformer
```

### Cấu hình global pipe

Trong `main.ts`:

```ts
app.useGlobalPipes(
  new ValidationPipe({
    whitelist: true,
    transform: true,
    forbidNonWhitelisted: true,
  }),
);
```

### CreateBookDto

```ts
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

### Bài tập

1. Nếu thiếu `title`, API phải trả 400.
2. Nếu `quantity < 0`, API phải trả 400.
3. Nếu gửi field dư, API phải reject.
4. Sử dụng DTO cho `POST /books`.

---

# PHẦN 3: UNIT TEST CƠ BẢN

## Buổi 5: Jest và TestingModule

### Mục tiêu

* Hiểu unit test là gì.
* Biết cấu trúc file `.spec.ts`.
* Biết dùng `describe`, `it`, `expect`.
* Biết tạo TestingModule trong NestJS.

### Nội dung

1. Unit test là gì?
2. Vì sao cần unit test?
3. Jest cơ bản.
4. File `.spec.ts`.
5. `TestingModule`.
6. Test service độc lập.

### Ví dụ test đơn giản

```ts
describe('BooksService', () => {
  it('should create a book', () => {
    const result = service.create({
      title: 'Clean Code',
      author: 'Robert C. Martin',
      isbn: '123',
      quantity: 5,
    });

    expect(result.title).toBe('Clean Code');
    expect(result.quantity).toBe(5);
  });
});
```

### Bài tập

Viết unit test cho:

```txt
BooksService.findAll()
BooksService.findOne()
BooksService.create()
BooksService.update()
BooksService.remove()
```

---

## Buổi 6: Unit Test cho Service

### Mục tiêu

* Viết test cho business logic.
* Test cả case đúng và case lỗi.
* Biết kiểm tra exception.

### Nội dung

1. Test happy case.
2. Test error case.
3. Test `NotFoundException`.
4. Arrange – Act – Assert.

### Checklist test BooksService

```txt
findAll()
- trả về danh sách books

findOne()
- tìm thấy book
- không tìm thấy book thì throw NotFoundException

create()
- tạo book mới
- book có id
- book được thêm vào danh sách

update()
- cập nhật book thành công
- không tìm thấy book thì throw NotFoundException

remove()
- xóa book thành công
- không tìm thấy book thì throw NotFoundException
```

### Bài tập

Hoàn thành file:

```txt
books.service.spec.ts
```

Yêu cầu:

```txt
Ít nhất 8 test cases
Tất cả test phải pass
Không gọi HTTP thật
Không dùng database thật
```

---

## Buổi 7: Unit Test cho Controller

### Mục tiêu

* Biết test Controller.
* Biết mock Service.
* Hiểu Controller không nên chứa logic phức tạp.

### Nội dung

1. Controller test khác Service test như thế nào?
2. Mock service bằng `jest.fn()`.
3. Kiểm tra controller gọi đúng service.
4. Kiểm tra controller trả đúng response.

### Mock service mẫu

```ts
const mockBooksService = {
  findAll: jest.fn(),
  findOne: jest.fn(),
  create: jest.fn(),
  update: jest.fn(),
  remove: jest.fn(),
};
```

### Bài tập

Viết unit test cho:

```txt
BooksController.findAll()
BooksController.findOne()
BooksController.create()
BooksController.update()
BooksController.remove()
```

Yêu cầu:

```txt
- Controller không dùng service thật
- Service được mock
- Kiểm tra service được gọi đúng tham số
```

---

# PHẦN 4: E2E TEST API

## Buổi 8: E2E Test với Supertest

### Mục tiêu

* Hiểu e2e test là gì.
* Biết kiểm tra API thật qua HTTP request.
* Biết test status code và response body.

### Nội dung

1. Unit test vs e2e test.
2. Supertest là gì?
3. File `test/app.e2e-spec.ts`.
4. Tạo Nest app trong test.
5. Gửi request thật vào API.

### Ví dụ e2e test

```ts
it('/books (GET)', () => {
  return request(app.getHttpServer())
    .get('/books')
    .expect(200);
});
```

### API cần test

```txt
GET    /books
POST   /books
GET    /books/:id
PATCH  /books/:id
DELETE /books/:id
```

### Bài tập

Viết e2e test cho:

```txt
POST /books body hợp lệ → 201
POST /books thiếu title → 400
POST /books quantity âm → 400
GET /books/:id tồn tại → 200
GET /books/:id không tồn tại → 404
```

---

# PHẦN 5: DATABASE VÀ MOCK DATABASE

## Buổi 9: Thêm Database Layer

### Mục tiêu

* Hiểu vì sao không nên viết logic database trực tiếp trong controller.
* Biết tạo PrismaService hoặc Repository layer.
* Biết service gọi database thông qua dependency.

### Nội dung

1. Database layer là gì?
2. Prisma hoặc TypeORM.
3. Repository pattern cơ bản.
4. Vì sao unit test không nên phụ thuộc DB thật.
5. Mock database trong test.

### Gợi ý chọn công nghệ

Trong khóa học cơ bản, nên dùng:

```txt
Prisma + SQLite
```

Lý do:

```txt
- Setup nhanh
- Ít lỗi môi trường
- Phù hợp học test
- Có thể đổi sang MySQL/PostgreSQL sau
```

### Bài tập

1. Thêm Prisma vào project.
2. Tạo model `Book`.
3. Viết lại `BooksService` để dùng database.
4. API vẫn giữ nguyên.

---

## Buổi 10: Unit Test Service có Mock Database

### Mục tiêu

* Biết mock PrismaService.
* Biết test service mà không connect database.
* Biết kiểm tra hàm database được gọi đúng.

### Mock Prisma mẫu

```ts
const mockPrisma = {
  book: {
    findMany: jest.fn(),
    findUnique: jest.fn(),
    create: jest.fn(),
    update: jest.fn(),
    delete: jest.fn(),
  },
};
```

### Test cần viết

```txt
BooksService.findAll()
- gọi prisma.book.findMany

BooksService.findOne()
- nếu tìm thấy book → trả book
- nếu không tìm thấy → throw NotFoundException

BooksService.create()
- gọi prisma.book.create với data đúng

BooksService.update()
- kiểm tra book tồn tại trước khi update
- nếu không tồn tại → throw NotFoundException
```

### Bài tập

Hoàn thành test service với mock database.

Yêu cầu:

```txt
Không connect database thật
Không seed dữ liệu thật
Test chạy nhanh
Có test cả success case và fail case
```

---

# PHẦN 6: BUSINESS LOGIC THỰC TẾ

## Buổi 11: Module Borrow - Mượn sách

### Mục tiêu

* Tạo API có business logic phức tạp hơn CRUD.
* Biết test nhiều điều kiện nghiệp vụ.

### API cần làm

```txt
POST /borrows
```

### Logic

```txt
1. Kiểm tra user có tồn tại không.
2. Kiểm tra book có tồn tại không.
3. Kiểm tra quantity > 0.
4. Nếu hợp lệ:
   - tạo borrow record
   - giảm quantity của book đi 1
```

### DTO

```ts
export class CreateBorrowDto {
  userId: number;
  bookId: number;
}
```

### Test cần viết

```txt
Case 1:
user tồn tại, book tồn tại, quantity > 0
→ tạo borrow thành công

Case 2:
user không tồn tại
→ NotFoundException

Case 3:
book không tồn tại
→ NotFoundException

Case 4:
book quantity = 0
→ BadRequestException

Case 5:
tạo borrow thành công thì quantity giảm 1
```

---

## Buổi 12: Module Borrow - Trả sách

### Mục tiêu

* Hoàn thiện logic mượn/trả sách.
* Biết test trạng thái dữ liệu.

### API cần làm

```txt
PATCH /borrows/:id/return
```

### Logic

```txt
1. Kiểm tra borrow record có tồn tại không.
2. Nếu không tồn tại → 404.
3. Nếu sách đã trả rồi → 400.
4. Nếu hợp lệ:
   - cập nhật returnedAt
   - tăng quantity của book lên 1
```

### Test cần viết

```txt
Case 1:
borrow tồn tại và chưa trả
→ trả sách thành công

Case 2:
borrow không tồn tại
→ NotFoundException

Case 3:
borrow đã có returnedAt
→ BadRequestException

Case 4:
trả sách thành công thì quantity tăng 1
```

---

# PHẦN 7: AUTH VÀ GUARD CƠ BẢN

## Buổi 13: Guard và quyền truy cập API

### Mục tiêu

* Hiểu Guard là gì.
* Biết chặn API nếu user chưa đăng nhập.
* Biết phân quyền admin/user cơ bản.

### Nội dung

1. Guard dùng để kiểm tra quyền.
2. AuthGuard.
3. RolesGuard.
4. Public route vs protected route.
5. Test API có auth.

### Rule đề xuất

```txt
GET /books
→ public

POST /books
→ admin only

PATCH /books/:id
→ admin only

DELETE /books/:id
→ admin only

POST /borrows
→ logged in user

PATCH /borrows/:id/return
→ logged in user
```

### Bài tập

1. Tạo guard fake auth.
2. Nếu không có token → 401.
3. Nếu role không phải admin → 403.
4. Nếu là admin → cho phép tạo sách.

---

## Buổi 14: E2E Test cho Auth

### Mục tiêu

* Viết e2e test cho API có guard.
* Kiểm tra 401, 403, 201.

### Test cần viết

```txt
POST /books không token
→ 401

POST /books token user thường
→ 403

POST /books token admin
→ 201

POST /borrows không token
→ 401

POST /borrows token hợp lệ
→ 201
```

### Bài tập

Hoàn thành file:

```txt
auth.e2e-spec.ts
```

---

# PHẦN 8: ÔN TẬP VÀ HOÀN THIỆN PROJECT

## Buổi 15: Refactor Project

### Mục tiêu

* Tổ chức lại source code sạch.
* Tách module rõ ràng.
* Chuẩn hóa response và error.

### Cấu trúc mong muốn

```txt
src/
  books/
    dto/
    books.controller.ts
    books.service.ts
    books.module.ts
    books.controller.spec.ts
    books.service.spec.ts

  users/
    dto/
    users.controller.ts
    users.service.ts
    users.module.ts
    users.service.spec.ts

  borrows/
    dto/
    borrows.controller.ts
    borrows.service.ts
    borrows.module.ts
    borrows.service.spec.ts

  auth/
    auth.controller.ts
    auth.service.ts
    guards/
    decorators/

  prisma/
    prisma.module.ts
    prisma.service.ts
```

### Bài tập

1. Kiểm tra lại naming.
2. Xóa code dư.
3. Đảm bảo tất cả test pass.
4. Chạy test coverage.

```bash
npm run test:cov
```

---

## Buổi 16: Tổng ôn và kiểm tra cuối khóa

### Mục tiêu

* Đánh giá lại toàn bộ kiến thức.
* Hoàn thiện project cuối khóa.
* Biết tự viết API mới và test tương ứng.

### Bài kiểm tra cuối khóa

Thêm module mới:

```txt
Categories
```

API cần làm:

```txt
GET    /categories
GET    /categories/:id
POST   /categories
PATCH  /categories/:id
DELETE /categories/:id
```

Yêu cầu:

```txt
1. Có DTO validate input.
2. Có service xử lý logic.
3. Có unit test cho service.
4. Có unit test cho controller.
5. Có e2e test cho API chính.
6. Nếu category không tồn tại, trả 404.
```

---

# KẾ HOẠCH HỌC THEO TUẦN

## Tuần 1: NestJS Core

```txt
Buổi 1: Cài đặt, tạo project, hiểu cấu trúc
Buổi 2: Controller, Service, Module
Buổi 3: CRUD API cho Books
```

Kết quả cuối tuần:

```txt
- Chạy được NestJS app
- Có BooksModule
- Có CRUD API cơ bản
```

---

## Tuần 2: DTO và Unit Test cơ bản

```txt
Buổi 4: DTO và ValidationPipe
Buổi 5: Jest và TestingModule
Buổi 6: Unit test cho Service
```

Kết quả cuối tuần:

```txt
- Validate được request body
- Viết được unit test service
- Biết test exception
```

---

## Tuần 3: Controller Test và E2E Test

```txt
Buổi 7: Unit test cho Controller
Buổi 8: E2E test với Supertest
```

Kết quả cuối tuần:

```txt
- Test được controller bằng mock service
- Test API thật bằng HTTP request
```

---

## Tuần 4: Database và Mock Database

```txt
Buổi 9: Thêm database layer
Buổi 10: Unit test với mock database
```

Kết quả cuối tuần:

```txt
- API dùng database
- Unit test không phụ thuộc database thật
```

---

## Tuần 5: Business Logic

```txt
Buổi 11: API mượn sách
Buổi 12: API trả sách
```

Kết quả cuối tuần:

```txt
- Có business logic thực tế
- Test được nhiều case nghiệp vụ
```

---

## Tuần 6: Auth, Guard và Tổng ôn

```txt
Buổi 13: Guard và phân quyền
Buổi 14: E2E test auth
Buổi 15: Refactor project
Buổi 16: Kiểm tra cuối khóa
```

Kết quả cuối khóa:

```txt
- Có project NestJS hoàn chỉnh
- Có unit test và e2e test
- Hiểu cách tổ chức backend NestJS
```

---

# CHECKLIST HOÀN THÀNH KHÓA HỌC

## NestJS Core

```txt
[ ] Hiểu Module
[ ] Hiểu Controller
[ ] Hiểu Service
[ ] Hiểu Provider
[ ] Hiểu Dependency Injection
[ ] Biết tạo module bằng CLI
[ ] Biết tạo REST API
[ ] Biết xử lý lỗi bằng Exception
```

## DTO và Validation

```txt
[ ] Biết tạo DTO
[ ] Biết dùng class-validator
[ ] Biết dùng ValidationPipe
[ ] Biết validate required field
[ ] Biết validate number/string
[ ] Biết reject field dư
```

## Testing

```txt
[ ] Biết viết unit test cho service
[ ] Biết viết unit test cho controller
[ ] Biết mock service
[ ] Biết mock database
[ ] Biết test exception
[ ] Biết viết e2e test
[ ] Biết test status code
[ ] Biết test response body
```

## Project

```txt
[ ] Có BooksModule
[ ] Có UsersModule
[ ] Có BorrowsModule
[ ] Có AuthModule
[ ] Có PrismaModule hoặc DatabaseModule
[ ] Có unit test
[ ] Có e2e test
[ ] Có test coverage
```

---

# BÀI TẬP TỰ ÔN

## Bài 1: Tạo API Products

Tạo module `products` với API:

```txt
GET    /products
GET    /products/:id
POST   /products
PATCH  /products/:id
DELETE /products/:id
```

Product gồm:

```txt
id
name
price
stock
createdAt
```

Yêu cầu:

```txt
- name không được rỗng
- price phải >= 0
- stock phải >= 0
- nếu product không tồn tại trả 404
- viết unit test cho service
- viết e2e test cho POST và GET
```

---

## Bài 2: Tạo API Orders

Tạo module `orders`.

Logic:

```txt
1. User tạo order.
2. Product phải tồn tại.
3. Stock phải đủ.
4. Tạo order thành công thì giảm stock.
5. Nếu stock không đủ thì trả 400.
```

Test cases:

```txt
- tạo order thành công
- product không tồn tại
- stock không đủ
- tạo order xong stock giảm đúng
```

---

## Bài 3: Viết Guard phân quyền

Rule:

```txt
GET /products → public
POST /products → admin only
POST /orders → user login required
```

Test cases:

```txt
- không token → 401
- user thường gọi API admin → 403
- admin gọi API admin → thành công
```

---

# TIÊU CHÍ ĐÁNH GIÁ CUỐI KHÓA

Học viên đạt yêu cầu nếu:

```txt
1. Tự tạo được module mới.
2. Tự viết được CRUD API.
3. Tự tạo DTO validate request.
4. Tự viết service chứa business logic.
5. Tự viết unit test cho service.
6. Tự viết controller test bằng mock service.
7. Tự viết e2e test kiểm tra API.
8. Biết mock database.
9. Biết debug test fail.
10. Biết tổ chức project rõ ràng.
```

---

# KẾT QUẢ MONG MUỐN

Sau khóa học, học viên có một project hoàn chỉnh:

```txt
NestJS Library API
- Books CRUD
- Users CRUD cơ bản
- Borrow book
- Return book
- Auth guard cơ bản
- DTO validation
- Unit test
- E2E test
- Mock database
- Test coverage
```

Đây có thể dùng làm project mẫu để tiếp tục học:

```txt
- JWT authentication thật
- PostgreSQL/MySQL
- Docker
- CI/CD
- Swagger
- Role-based access control
- Clean Architecture
```
