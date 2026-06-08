# Bài 1: NestJS là gì? Cài đặt môi trường

## 1. Mục tiêu bài học

Sau buổi này, bạn cần làm được 4 việc:

1. Hiểu NestJS dùng để làm gì.
2. Biết NestJS khác Express ở điểm nào.
3. Tạo được project NestJS đầu tiên.
4. Chạy được server local và hiểu sơ bộ các file chính:

```txt
main.ts
app.module.ts
app.controller.ts
app.service.ts
```

---

# 2. NestJS là gì?

NestJS là một framework backend chạy trên Node.js.

Nói dễ hiểu:

```txt
Node.js = môi trường chạy JavaScript phía server
Express = framework nhỏ gọn để viết API
NestJS = framework có cấu trúc rõ ràng hơn, phù hợp project lớn
```

Ví dụ bạn muốn làm backend cho hệ thống quản lý thư viện:

```txt
GET /books          lấy danh sách sách
POST /books         thêm sách mới
GET /books/:id      lấy chi tiết sách
PATCH /books/:id    cập nhật sách
DELETE /books/:id   xóa sách
```

Bạn có thể viết bằng Express hoặc NestJS. Nhưng NestJS giúp code được chia rõ ràng hơn:

```txt
Controller → nhận request
Service    → xử lý logic
Module     → gom nhóm chức năng
DTO        → kiểm tra dữ liệu đầu vào
Test       → kiểm tra code chạy đúng hay không
```

---

# 3. Vì sao nên học NestJS?

Với project nhỏ, Express rất nhanh và đơn giản.

Nhưng khi project lớn hơn, Express dễ bị rối nếu không tự tổ chức kỹ:

```txt
routes/
controllers/
services/
middlewares/
validators/
database/
```

NestJS thì ép mình đi theo cấu trúc chuẩn ngay từ đầu.

Ví dụ project Library API sau này sẽ có:

```txt
books/
users/
borrows/
auth/
prisma/
```

Mỗi phần có controller, service, module riêng. Nhờ vậy code dễ đọc, dễ test và dễ mở rộng.

---

# 4. So sánh nhanh NestJS và Express

| Tiêu chí             | Express               | NestJS                     |
| -------------------- | --------------------- | -------------------------- |
| Cấu trúc             | Tự tổ chức            | Có chuẩn sẵn               |
| Ngôn ngữ             | JavaScript/TypeScript | TypeScript là chính        |
| Phù hợp              | App nhỏ, API đơn giản | App vừa/lớn, cần maintain  |
| Testing              | Tự setup nhiều        | Có sẵn Jest, TestingModule |
| Dependency Injection | Không có sẵn          | Có sẵn                     |
| Module hóa           | Tự làm                | Có sẵn Module              |

Nói ngắn gọn:

```txt
Express giống tự xây nhà từ đầu.
NestJS giống có sẵn bản thiết kế kiến trúc.
```

---

# 5. NestJS hoạt động theo luồng nào?

Ở mức cơ bản nhất, request đi như sau:

```txt
Client → Controller → Service → Response
```

Ví dụ:

```txt
Người dùng gọi GET /books
        ↓
BooksController nhận request
        ↓
BooksService xử lý lấy danh sách sách
        ↓
Controller trả response về client
```

Trong Bài 1, bạn chưa cần viết Books API ngay. Nhưng cần hiểu trước luồng này vì từ Bài 2 trở đi sẽ dùng liên tục.

---

# 6. Cài đặt môi trường

## 6.1. Kiểm tra Node.js

Mở terminal hoặc CMD, chạy:

```bash
node -v
```

Nếu thấy dạng như:

```txt
v20.x.x
```

hoặc:

```txt
v22.x.x
```

là được.

Tiếp theo kiểm tra npm:

```bash
npm -v
```

Nếu có version hiện ra là ổn.

Nếu chưa có Node.js, cài bản LTS từ trang chính thức Node.js.

---

## 6.2. Cài NestJS CLI

NestJS CLI là công cụ dòng lệnh giúp tạo project, tạo module, controller, service nhanh hơn.

Chạy lệnh:

```bash
npm i -g @nestjs/cli
```

Sau khi cài xong, kiểm tra:

```bash
nest --version
```

Nếu hiện version, ví dụ:

```txt
11.x.x
```

là đã cài thành công.

---

# 7. Tạo project NestJS đầu tiên

Chạy lệnh:

```bash
nest new nest-library-api
```

CLI sẽ hỏi chọn package manager:

```txt
npm
yarn
pnpm
```

Ở giai đoạn học cơ bản, chọn:

```txt
npm
```

Sau khi tạo xong:

```bash
cd nest-library-api
```

Chạy project:

```bash
npm run start:dev
```

Nếu chạy thành công, terminal sẽ hiện kiểu như:

```txt
[Nest] ... LOG [NestFactory] Starting Nest application...
[Nest] ... LOG [RoutesResolver] AppController {/}:
[Nest] ... LOG [RouterExplorer] Mapped {/, GET} route
[Nest] ... LOG [NestApplication] Nest application successfully started
```

Mở trình duyệt:

```txt
http://localhost:3000
```

Kết quả mặc định thường là:

```txt
Hello World!
```

---

# 8. Ý nghĩa `npm run start:dev`

Trong project NestJS, mở file `package.json`, bạn sẽ thấy phần scripts:

```json
{
  "scripts": {
    "start": "nest start",
    "start:dev": "nest start --watch"
  }
}
```

Lệnh:

```bash
npm run start:dev
```

có nghĩa là chạy server ở chế độ development.

Điểm quan trọng:

```txt
--watch = tự reload server khi code thay đổi
```

Ví dụ bạn sửa text trong service, lưu file, NestJS tự restart server.

---

# 9. Cấu trúc thư mục ban đầu

Sau khi tạo project, bạn sẽ thấy cấu trúc gần giống:

```txt
nest-library-api/
  src/
    app.controller.ts
    app.controller.spec.ts
    app.module.ts
    app.service.ts
    main.ts
  test/
    app.e2e-spec.ts
    jest-e2e.json
  package.json
  tsconfig.json
  nest-cli.json
```

Trong Bài 1, tập trung vào thư mục:

```txt
src/
```

Đây là nơi chứa source code chính của ứng dụng.

---

# 10. Giải thích từng file quan trọng

## 10.1. File `main.ts`

File này là điểm bắt đầu của ứng dụng NestJS.

Ví dụ mặc định:

```ts
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}

bootstrap();
```

Giải thích dễ hiểu:

```ts
const app = await NestFactory.create(AppModule);
```

Dòng này tạo app NestJS từ `AppModule`.

```ts
await app.listen(3000);
```

Dòng này chạy server ở port `3000`.

Nghĩa là khi bạn mở:

```txt
http://localhost:3000
```

thì request sẽ đi vào ứng dụng NestJS.

Có thể hiểu `main.ts` là:

```txt
Cửa khởi động của toàn bộ backend.
```

---

## 10.2. File `app.module.ts`

Ví dụ:

```ts
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

Đây là module gốc của ứng dụng.

Trong NestJS, module dùng để gom nhóm các thành phần liên quan.

Ở đây:

```ts
controllers: [AppController]
```

nghĩa là module này đang dùng `AppController`.

```ts
providers: [AppService]
```

nghĩa là module này đang dùng `AppService`.

Dễ hiểu hơn:

```txt
AppModule quản lý AppController và AppService.
```

Sau này khi tạo `BooksModule`, `UsersModule`, `BorrowsModule`, các module đó sẽ được import vào module chính hoặc module liên quan.

---

## 10.3. File `app.controller.ts`

Ví dụ:

```ts
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }
}
```

Controller là nơi nhận request từ client.

Dòng này:

```ts
@Controller()
```

khai báo đây là một controller.

Dòng này:

```ts
@Get()
```

nghĩa là method bên dưới sẽ xử lý request GET.

Vì `@Get()` không truyền path, nên nó xử lý route gốc:

```txt
GET /
```

Khi bạn mở:

```txt
http://localhost:3000
```

NestJS sẽ gọi:

```ts
getHello()
```

Sau đó `getHello()` gọi tiếp service:

```ts
return this.appService.getHello();
```

---

## 10.4. File `app.service.ts`

Ví dụ:

```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}
```

Service là nơi xử lý logic.

Trong ví dụ này logic rất đơn giản:

```ts
return 'Hello World!';
```

Nhưng sau này service sẽ xử lý những việc như:

```txt
Tìm sách
Thêm sách
Cập nhật sách
Xóa sách
Kiểm tra sách có tồn tại không
Kiểm tra số lượng sách còn hay hết
```

Dòng này:

```ts
@Injectable()
```

cho NestJS biết class này có thể được inject vào nơi khác.

Ví dụ `AppService` được inject vào `AppController`:

```ts
constructor(private readonly appService: AppService) {}
```

Đây là Dependency Injection.

---

# 11. Hiểu Dependency Injection đơn giản

Thay vì controller tự tạo service như thế này:

```ts
const service = new AppService();
```

NestJS sẽ tự đưa service vào controller:

```ts
constructor(private readonly appService: AppService) {}
```

Lợi ích:

```txt
Dễ test hơn
Dễ thay service thật bằng service giả khi unit test
Dễ quản lý dependency trong project lớn
```

Đây là lý do NestJS rất phù hợp với mục tiêu khóa học của bạn: học backend có cấu trúc và viết unit test tốt.

---

# 12. Thử sửa response đầu tiên

Mở file:

```txt
src/app.service.ts
```

Đổi:

```ts
return 'Hello World!';
```

thành:

```ts
return 'Welcome to NestJS Library API!';
```

File sau khi sửa:

```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Welcome to NestJS Library API!';
  }
}
```

Lưu file.

Mở lại trình duyệt:

```txt
http://localhost:3000
```

Bạn sẽ thấy:

```txt
Welcome to NestJS Library API!
```

Việc này chứng minh bạn đã hiểu luồng cơ bản:

```txt
Browser gọi GET /
        ↓
AppController nhận request
        ↓
AppController gọi AppService
        ↓
AppService trả text
        ↓
Browser hiển thị kết quả
```

---

# 13. Sơ đồ dễ nhớ

```txt
main.ts
  ↓
AppModule
  ↓
AppController
  ↓
AppService
  ↓
Response
```

Giải thích:

```txt
main.ts       khởi động app
AppModule     khai báo controller/service nào được dùng
Controller    nhận request
Service       xử lý logic
Response      trả kết quả về client
```

---

# 14. Các lỗi thường gặp

## Lỗi 1: `nest` không được nhận diện

Thông báo có thể là:

```txt
'nest' is not recognized as an internal or external command
```

Nguyên nhân thường gặp:

```txt
Nest CLI chưa cài thành công
npm global path chưa được add vào PATH
```

Cách xử lý nhanh:

```bash
npm i -g @nestjs/cli
```

Sau đó tắt terminal, mở lại terminal mới.

Kiểm tra:

```bash
nest --version
```

---

## Lỗi 2: Port 3000 đang được dùng

Thông báo có thể là:

```txt
EADDRINUSE: address already in use :::3000
```

Nghĩa là có app khác đang chạy ở port 3000.

Cách 1: Tắt app đang dùng port 3000.

Cách 2: Đổi port trong `main.ts`:

```ts
await app.listen(3001);
```

Sau đó mở:

```txt
http://localhost:3001
```

---

## Lỗi 3: Chạy sai thư mục

Nếu chạy:

```bash
npm run start:dev
```

mà báo không tìm thấy `package.json`, có thể bạn chưa vào thư mục project.

Cần chạy:

```bash
cd nest-library-api
npm run start:dev
```

---

# 15. Checklist hoàn thành Bài 1

Bạn đạt yêu cầu Bài 1 nếu làm được:

```txt
[ ] Cài được Node.js
[ ] Kiểm tra được node -v
[ ] Kiểm tra được npm -v
[ ] Cài được NestJS CLI
[ ] Kiểm tra được nest --version
[ ] Tạo được project nest-library-api
[ ] Chạy được npm run start:dev
[ ] Mở được http://localhost:3000
[ ] Hiểu vai trò main.ts
[ ] Hiểu vai trò app.module.ts
[ ] Hiểu vai trò app.controller.ts
[ ] Hiểu vai trò app.service.ts
[ ] Sửa được response Hello World
```

---

# 16. Bài tập thực hành cuối buổi

## Bài tập 1: Tạo project

Tạo project tên:

```txt
nest-library-api
```

Chạy project thành công ở:

```txt
http://localhost:3000
```

---

## Bài tập 2: Đổi nội dung response

Trong file:

```txt
src/app.service.ts
```

Đổi response thành:

```txt
Library API is running
```

Kết quả khi mở browser phải là:

```txt
Library API is running
```

---

## Bài tập 3: Đổi port

Trong file:

```txt
src/main.ts
```

Đổi port từ:

```ts
await app.listen(3000);
```

thành:

```ts
await app.listen(3001);
```

Sau đó mở:

```txt
http://localhost:3001
```

---

## Bài tập 4: Giải thích lại bằng lời của bạn

Tự trả lời 4 câu sau:

```txt
1. main.ts dùng để làm gì?
2. app.module.ts dùng để làm gì?
3. app.controller.ts dùng để làm gì?
4. app.service.ts dùng để làm gì?
```

Gợi ý trả lời ngắn:

```txt
main.ts là nơi khởi động app.
app.module.ts là nơi khai báo các thành phần của app.
app.controller.ts là nơi nhận request.
app.service.ts là nơi xử lý logic.
```

---

# 17. Mini quiz kiểm tra hiểu bài

## Câu 1

NestJS là framework dùng để làm gì?

A. Viết giao diện frontend
B. Viết backend API
C. Thiết kế database
D. Chạy hệ điều hành

Đáp án: **B**

---

## Câu 2

File nào là điểm khởi động của ứng dụng NestJS?

A. `app.service.ts`
B. `app.controller.ts`
C. `main.ts`
D. `package.json`

Đáp án: **C**

---

## Câu 3

Controller có nhiệm vụ chính là gì?

A. Nhận request từ client
B. Lưu dữ liệu vào database
C. Cài package
D. Build frontend

Đáp án: **A**

---

## Câu 4

Service thường dùng để làm gì?

A. Nhận request trực tiếp từ browser
B. Xử lý business logic
C. Cấu hình TypeScript
D. Chạy npm install

Đáp án: **B**

---

## Câu 5

Luồng xử lý cơ bản trong NestJS là gì?

A. Service → Controller → Client
B. Client → Controller → Service → Response
C. Module → Client → Service
D. Database → Controller → main.ts

Đáp án: **B**

---

# 18. Ghi nhớ sau Bài 1

Sau bài này chỉ cần nhớ 5 ý chính:

```txt
1. NestJS là framework backend Node.js dùng TypeScript.
2. NestJS tổ chức code theo module.
3. main.ts là nơi khởi động app.
4. Controller nhận request.
5. Service xử lý logic.
```

Từ Bài 2, mình sẽ bắt đầu tạo module thật đầu tiên là:

```txt
BooksModule
BooksController
BooksService
```

và xây API:

```txt
GET /books
GET /books/:id
POST /books
```
