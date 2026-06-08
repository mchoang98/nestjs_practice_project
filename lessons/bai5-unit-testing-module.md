# Bài 5: Jest và TestingModule trong NestJS

## 1. Mục tiêu bài học

Sau buổi này, bạn cần làm được:

1. Hiểu unit test là gì.
2. Hiểu vì sao backend API cần unit test.
3. Biết Jest là gì.
4. Hiểu file `.spec.ts` dùng để làm gì.
5. Biết dùng `describe`, `it`, `expect`.
6. Biết tạo `TestingModule` trong NestJS.
7. Biết viết unit test cơ bản cho `BooksService`.
8. Biết chạy test bằng lệnh `npm run test`.

Trong bài này, ta sẽ tập trung test Service, chưa test Controller và chưa gọi HTTP thật.

---

# 2. Ôn lại Bài 4

Ở Bài 4, chúng ta đã học DTO và ValidationPipe.

API `POST /books` đã có thể kiểm tra request body:

```txt id="ag5ce7"
Thiếu title       → 400 Bad Request
quantity < 0      → 400 Bad Request
Gửi field dư      → 400 Bad Request
Body hợp lệ       → 201 Created
```

Tức là API đã bắt đầu an toàn hơn.

Tuy nhiên, khi project lớn lên, chỉ test bằng tay qua Postman sẽ rất mất thời gian.

Ví dụ mỗi lần sửa code, bạn phải test lại:

```txt id="pveblr"
GET /books
GET /books/:id
POST /books
PATCH /books/:id
DELETE /books/:id
POST /books body sai
POST /books thiếu title
POST /books quantity âm
```

Nếu làm thủ công nhiều lần sẽ dễ sót lỗi.

Vì vậy ta cần học test tự động.

---

# 3. Unit test là gì?

Unit test là kiểm tra một đơn vị nhỏ trong code.

Trong NestJS, đơn vị nhỏ thường là:

```txt id="v9b5j8"
Service
Controller
Function
Method
Guard
Pipe
```

Ví dụ với `BooksService`, ta có các method:

```ts id="sa0pvr"
findAll()
findOne(id)
create(data)
update(id, data)
remove(id)
```

Unit test sẽ kiểm tra từng method này có chạy đúng không.

Ví dụ:

```txt id="uj2k0e"
findAll() phải trả về danh sách books
findOne(1) phải trả về book id = 1
findOne(999) phải throw NotFoundException
create(data) phải tạo book mới
```

Điểm quan trọng:

```txt id="gt72cv"
Unit test không gọi HTTP thật.
Unit test không cần mở trình duyệt.
Unit test không cần Postman.
Unit test không cần database thật.
```

---

# 4. Vì sao cần unit test?

Unit test giúp bạn kiểm tra code nhanh hơn và tự tin hơn khi sửa code.

Ví dụ bạn sửa logic trong `BooksService.update()`.

Nếu không có test, bạn phải tự gọi API bằng Postman để kiểm tra.

Nếu có test, chỉ cần chạy:

```bash id="en6b8y"
npm run test
```

Nếu tất cả test pass, bạn có thể yên tâm hơn là code vẫn hoạt động đúng.

Unit test giúp:

```txt id="z0e0pi"
Phát hiện lỗi sớm
Giảm test tay bằng Postman
Tự tin refactor code
Giúp code dễ maintain
Giúp kiểm tra business logic rõ ràng
Chuẩn bị tốt cho CI/CD sau này
```

---

# 5. Unit test khác e2e test như thế nào?

| Tiêu chí                 | Unit test                    | E2E test                                    |
| ------------------------ | ---------------------------- | ------------------------------------------- |
| Kiểm tra                 | Một phần nhỏ của code        | Toàn bộ API thật                            |
| Có gọi HTTP thật không   | Không                        | Có                                          |
| Có cần server thật không | Không                        | Có app test server                          |
| Tốc độ                   | Nhanh                        | Chậm hơn                                    |
| Ví dụ                    | Test `BooksService.create()` | Test `POST /books`                          |
| Mục tiêu                 | Kiểm tra logic nhỏ           | Kiểm tra luồng thật từ request đến response |

Trong Bài 5, ta học unit test.

E2E test sẽ học ở Bài 8.

---

# 6. Jest là gì?

Jest là framework dùng để viết và chạy test trong JavaScript/TypeScript.

NestJS mặc định đã tích hợp Jest khi tạo project bằng CLI.

Trong project NestJS, bạn sẽ thấy các file test có đuôi:

```txt id="yam8gy"
.spec.ts
```

Ví dụ:

```txt id="ckr995"
app.controller.spec.ts
books.service.spec.ts
books.controller.spec.ts
```

Lệnh chạy test:

```bash id="zxyewt"
npm run test
```

Lệnh chạy test ở chế độ watch:

```bash id="rm9x7h"
npm run test:watch
```

Lệnh xem test coverage:

```bash id="asx1pr"
npm run test:cov
```

---

# 7. File `.spec.ts` là gì?

File `.spec.ts` là file chứa test case.

Ví dụ:

```txt id="y8pcf8"
books.service.ts       → code thật
books.service.spec.ts  → test cho BooksService
```

Quy ước thường dùng:

```txt id="qf70va"
Tên file code thật: books.service.ts
Tên file test:      books.service.spec.ts
```

Điều này giúp dễ tìm test tương ứng với file code.

---

# 8. Cấu trúc test cơ bản trong Jest

Một test Jest thường có dạng:

```ts id="nt8f2r"
describe('Tên nhóm test', () => {
  it('mô tả test case', () => {
    expect(kết_quả_thực_tế).toBe(kết_quả_mong_muốn);
  });
});
```

Ví dụ đơn giản:

```ts id="hrkdkz"
describe('Math example', () => {
  it('should return 4 when 2 + 2', () => {
    expect(2 + 2).toBe(4);
  });
});
```

Giải thích:

```txt id="az808a"
describe dùng để gom nhóm test
it dùng để mô tả một test case
expect dùng để kiểm tra kết quả
toBe dùng để so sánh kết quả chính xác
```

---

# 9. Hiểu `describe`, `it`, `expect`

## 9.1. `describe`

`describe` dùng để gom nhóm các test liên quan.

Ví dụ:

```ts id="exqvbt"
describe('BooksService', () => {
  // các test của BooksService nằm ở đây
});
```

Nghĩa là nhóm test này dành cho `BooksService`.

---

## 9.2. `it`

`it` là một test case cụ thể.

Ví dụ:

```ts id="pq6q5x"
it('should return all books', () => {
  // kiểm tra findAll()
});
```

Tên test nên mô tả hành vi mong muốn.

Ví dụ tốt:

```txt id="re583o"
should return all books
should create a new book
should throw NotFoundException when book does not exist
```

Không nên đặt tên quá chung chung như:

```txt id="cmc62c"
test 1
case 1
check function
```

---

## 9.3. `expect`

`expect` dùng để kiểm tra kết quả.

Ví dụ:

```ts id="pkb3as"
expect(result).toBe(5);
```

Nghĩa là mong muốn `result` bằng `5`.

Một số matcher thường dùng:

| Matcher          | Ý nghĩa                               |
| ---------------- | ------------------------------------- |
| `toBe()`         | So sánh giá trị đơn giản              |
| `toEqual()`      | So sánh object hoặc array             |
| `toHaveLength()` | Kiểm tra độ dài array                 |
| `toBeDefined()`  | Kiểm tra có tồn tại                   |
| `toThrow()`      | Kiểm tra có throw lỗi                 |
| `toContain()`    | Kiểm tra array/string có chứa giá trị |

Ví dụ:

```ts id="hl5jdt"
expect(result).toBeDefined();
expect(result).toHaveLength(2);
expect(result.title).toBe('Clean Code');
expect(result).toEqual({ id: 1, title: 'Clean Code' });
```

---

# 10. Arrange - Act - Assert

Khi viết test, nên chia tư duy thành 3 bước:

```txt id="kzgrpd"
Arrange → chuẩn bị dữ liệu
Act     → gọi function cần test
Assert  → kiểm tra kết quả
```

Ví dụ:

```ts id="xce5nj"
it('should create a book', () => {
  // Arrange
  const data = {
    title: 'Refactoring',
    author: 'Martin Fowler',
    isbn: '9780201485677',
    quantity: 4,
  };

  // Act
  const result = service.create(data);

  // Assert
  expect(result.title).toBe('Refactoring');
  expect(result.quantity).toBe(4);
});
```

Cách viết này giúp test dễ đọc hơn.

---

# 11. TestingModule trong NestJS là gì?

Trong NestJS, Service thường được quản lý bởi Dependency Injection.

Ví dụ `BooksService` có decorator:

```ts id="ckg0vp"
@Injectable()
export class BooksService {}
```

Thay vì tự tạo service bằng:

```ts id="eifbxw"
const service = new BooksService();
```

NestJS thường khuyến khích tạo môi trường test bằng `TestingModule`.

Ví dụ:

```ts id="x4k0b9"
const module: TestingModule = await Test.createTestingModule({
  providers: [BooksService],
}).compile();

service = module.get<BooksService>(BooksService);
```

Hiểu đơn giản:

```txt id="vnonkb"
TestingModule là module giả lập nhỏ dùng riêng cho test.
```

Nó giúp NestJS tạo service giống cách app thật tạo service.

---

# 12. Kiểm tra file `books.service.spec.ts`

Khi bạn chạy:

```bash id="bq2sja"
nest g service books
```

NestJS thường tự tạo file:

```txt id="y6wqgr"
src/books/books.service.spec.ts
```

Nội dung ban đầu có thể như sau:

```ts id="b08mrl"
import { Test, TestingModule } from '@nestjs/testing';
import { BooksService } from './books.service';

describe('BooksService', () => {
  let service: BooksService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [BooksService],
    }).compile();

    service = module.get<BooksService>(BooksService);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
  });
});
```

---

# 13. Giải thích file test mặc định

## 13.1. Import Test và TestingModule

```ts id="yhqmch"
import { Test, TestingModule } from '@nestjs/testing';
```

`Test` dùng để tạo testing module.

`TestingModule` là kiểu dữ liệu của module test.

---

## 13.2. Khai báo biến service

```ts id="dzfgp8"
let service: BooksService;
```

Biến này sẽ chứa instance của `BooksService` để test.

---

## 13.3. `beforeEach`

```ts id="q30ohy"
beforeEach(async () => {
  const module: TestingModule = await Test.createTestingModule({
    providers: [BooksService],
  }).compile();

  service = module.get<BooksService>(BooksService);
});
```

`beforeEach` chạy trước mỗi test case.

Nghĩa là trước mỗi `it`, NestJS sẽ tạo lại `BooksService` mới.

Lợi ích:

```txt id="sgbi8d"
Mỗi test độc lập hơn
Dữ liệu test không bị ảnh hưởng lẫn nhau
Tránh test case này làm hỏng test case khác
```

---

## 13.4. Test service tồn tại

```ts id="pa0zwb"
it('should be defined', () => {
  expect(service).toBeDefined();
});
```

Test này kiểm tra `BooksService` có được tạo thành công hay không.

Đây là test cơ bản nhất.

---

# 14. Viết test cho `findAll()`

Giả sử `BooksService` có sẵn dữ liệu ban đầu:

```ts id="wt540l"
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

Thêm test:

```ts id="x0b0qh"
it('should return all books', () => {
  const result = service.findAll();

  expect(result).toHaveLength(2);
  expect(result[0].title).toBe('Clean Code');
});
```

Giải thích:

```ts id="hzutgs"
const result = service.findAll();
```

Gọi method cần test.

```ts id="cfhjs6"
expect(result).toHaveLength(2);
```

Mong muốn danh sách có 2 phần tử.

```ts id="nqgq5p"
expect(result[0].title).toBe('Clean Code');
```

Mong muốn book đầu tiên có title là `Clean Code`.

---

# 15. Viết test cho `findOne()`

Thêm test:

```ts id="ylj2l4"
it('should return a book by id', () => {
  const result = service.findOne(1);

  expect(result).toBeDefined();
  expect(result.id).toBe(1);
  expect(result.title).toBe('Clean Code');
});
```

Test này kiểm tra:

```txt id="ty250z"
Gọi findOne(1)
Phải tìm được book
Book trả về có id = 1
Book trả về có title đúng
```

---

# 16. Viết test cho `create()`

Thêm test:

```ts id="d5bxn0"
it('should create a book', () => {
  const result = service.create({
    title: 'Refactoring',
    author: 'Martin Fowler',
    isbn: '9780201485677',
    quantity: 4,
  });

  expect(result).toBeDefined();
  expect(result.id).toBeDefined();
  expect(result.title).toBe('Refactoring');
  expect(result.quantity).toBe(4);
});
```

Test này kiểm tra book mới được tạo có dữ liệu đúng.

Có thể kiểm tra thêm danh sách tăng số lượng:

```ts id="pl4u0q"
it('should add new book to books list', () => {
  service.create({
    title: 'Refactoring',
    author: 'Martin Fowler',
    isbn: '9780201485677',
    quantity: 4,
  });

  const books = service.findAll();

  expect(books).toHaveLength(3);
});
```

---

# 17. File `books.service.spec.ts` hoàn chỉnh cho Bài 5

Ở Bài 5, ta mới viết test cơ bản cho 3 method:

```txt id="d764ps"
findAll()
findOne()
create()
```

Mở file:

```txt id="wj3z5z"
src/books/books.service.spec.ts
```

Sửa thành:

```ts id="sdvpxi"
import { Test, TestingModule } from '@nestjs/testing';
import { BooksService } from './books.service';

describe('BooksService', () => {
  let service: BooksService;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      providers: [BooksService],
    }).compile();

    service = module.get<BooksService>(BooksService);
  });

  it('should be defined', () => {
    expect(service).toBeDefined();
  });

  it('should return all books', () => {
    const result = service.findAll();

    expect(result).toHaveLength(2);
    expect(result[0].title).toBe('Clean Code');
  });

  it('should return a book by id', () => {
    const result = service.findOne(1);

    expect(result).toBeDefined();
    expect(result.id).toBe(1);
    expect(result.title).toBe('Clean Code');
  });

  it('should create a book', () => {
    const result = service.create({
      title: 'Refactoring',
      author: 'Martin Fowler',
      isbn: '9780201485677',
      quantity: 4,
    });

    expect(result).toBeDefined();
    expect(result.id).toBeDefined();
    expect(result.title).toBe('Refactoring');
    expect(result.quantity).toBe(4);
  });

  it('should add new book to books list', () => {
    service.create({
      title: 'Domain-Driven Design',
      author: 'Eric Evans',
      isbn: '9780321125217',
      quantity: 2,
    });

    const books = service.findAll();

    expect(books).toHaveLength(3);
  });
});
```

---

# 18. Chạy test

Chạy lệnh:

```bash id="z7xw3z"
npm run test
```

Nếu test pass, bạn sẽ thấy kết quả gần giống:

```txt id="m5lr17"
PASS src/books/books.service.spec.ts
BooksService
  ✓ should be defined
  ✓ should return all books
  ✓ should return a book by id
  ✓ should create a book
  ✓ should add new book to books list
```

Nếu muốn test tự chạy lại khi lưu file:

```bash id="v52cxu"
npm run test:watch
```

---

# 19. Lỗi thường gặp khi viết test

## Lỗi 1: Import sai Service

Ví dụ lỗi:

```txt id="h2e0kw"
Cannot find module './books.service'
```

Nguyên nhân:

```txt id="jmga09"
Sai đường dẫn import
File test không nằm cùng thư mục với service
Tên file service sai
```

Cách kiểm tra:

```ts id="bzf2gu"
import { BooksService } from './books.service';
```

Nếu `books.service.spec.ts` nằm cùng thư mục với `books.service.ts`, import như trên là đúng.

---

## Lỗi 2: Test expect sai số lượng

Ví dụ:

```txt id="uerxi6"
Expected length: 2
Received length: 3
```

Nguyên nhân có thể là bạn đã thêm dữ liệu mặc định trong service nhiều hơn ví dụ.

Cách xử lý:

```txt id="luu2sx"
Kiểm tra lại mảng books ban đầu
Sửa expect(result).toHaveLength(...) cho đúng
```

---

## Lỗi 3: Dữ liệu test ảnh hưởng lẫn nhau

Ví dụ test này tạo thêm book:

```ts id="bo4ggu"
service.create(...)
```

Sau đó test khác thấy danh sách bị tăng số lượng.

Trong bài này, `beforeEach` sẽ tạo lại service trước mỗi test, nên thường không bị ảnh hưởng.

Nhưng nếu bạn dùng biến global hoặc static data, cần cẩn thận.

---

## Lỗi 4: Dùng `toBe` để so sánh object

Ví dụ:

```ts id="d7jv1d"
expect(result).toBe({
  id: 1,
  title: 'Clean Code',
});
```

Cách này dễ fail vì object trong JavaScript so sánh theo reference.

Nên dùng:

```ts id="fre7ak"
expect(result).toEqual({
  id: 1,
  title: 'Clean Code',
});
```

Hoặc kiểm tra từng field:

```ts id="bd4p3p"
expect(result.id).toBe(1);
expect(result.title).toBe('Clean Code');
```

---

# 20. Bài tập thực hành cuối buổi

Lưu ý: Bài tập dưới đây dùng dữ liệu khác với ví dụ chính trong bài để bạn tự áp dụng lại cách viết unit test.

## Bài tập 1: Viết unit test cho AuthorsService

Giả sử bạn đã làm `AuthorsService` ở Bài 3.

Viết file:

```txt id="udf4cy"
src/authors/authors.service.spec.ts
```

Test các method:

```txt id="zwhltw"
findAll()
findOne(id)
create(data)
```

Dữ liệu mẫu trong service:

```ts id="v6sw41"
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

Yêu cầu test:

```txt id="lk0mbd"
findAll() trả về danh sách có 2 authors
findOne(1) trả về author có name là Nguyễn Nhật Ánh
create() tạo author mới có id
create() làm danh sách tăng từ 2 lên 3
```

---

## Bài tập 2: Viết unit test cho CategoriesService

Viết file:

```txt id="qyjri2"
src/categories/categories.service.spec.ts
```

Test các method:

```txt id="fpimep"
findAll()
findOne(id)
create(data)
```

Dữ liệu mẫu:

```ts id="m8bts8"
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

Yêu cầu test:

```txt id="f0u22g"
findAll() trả về array
findAll() có length = 2
findOne(2) trả về category Novel
create() tạo category mới
create() trả về đúng name đã gửi vào
```

---

## Bài tập 3: Viết test theo Arrange - Act - Assert

Chọn một test trong `AuthorsService` và viết rõ theo cấu trúc:

```ts id="i4x984"
it('should create a new author', () => {
  // Arrange

  // Act

  // Assert
});
```

Ví dụ dữ liệu tạo mới:

```ts id="ekmbv0"
const data = {
  name: 'George Orwell',
  country: 'United Kingdom',
  birthYear: 1903,
};
```

Yêu cầu:

```txt id="h3w4j1"
Có comment Arrange
Có comment Act
Có comment Assert
Test phải pass
```

---

## Bài tập 4: Nâng cao 1 - Test NotFoundException

Trong Bài 5, ta mới test happy case.

Hãy viết thêm test cho trường hợp không tìm thấy author:

```txt id="l0lq60"
findOne(999) phải throw NotFoundException
```

Gợi ý:

```ts id="f4w3f7"
import { NotFoundException } from '@nestjs/common';

it('should throw NotFoundException when author does not exist', () => {
  expect(() => service.findOne(999)).toThrow(NotFoundException);
});
```

Yêu cầu:

```txt id="lq27wx"
Test phải kiểm tra được exception
Không gọi HTTP thật
Không dùng Postman
```

---

## Bài tập 5: Nâng cao 2 - Test create id không trùng sau khi xóa

Nếu ở Bài 3 bạn đã làm logic tạo id an toàn hơn, hãy viết test kiểm tra:

```txt id="a86l7j"
Ban đầu có id 1, 2
Xóa id 2
Tạo author mới
Author mới phải có id = 3, không phải id = 2
```

Gợi ý test:

```ts id="dfmiev"
it('should create new author with next max id after remove', () => {
  service.remove(2);

  const result = service.create({
    name: 'George Orwell',
    country: 'United Kingdom',
    birthYear: 1903,
  });

  expect(result.id).toBe(3);
});
```

Nếu test fail, nghĩa là logic tạo id của bạn vẫn đang dùng:

```ts id="hk5bd1"
this.authors.length + 1
```

Cần sửa lại bằng cách lấy id lớn nhất hiện tại + 1.

---

## Bài tập 6: Checklist tự kiểm tra

Sau khi hoàn thành, tự kiểm tra:

```txt id="ijp1fp"
[ ] Chạy được npm run test
[ ] Hiểu describe dùng để gom nhóm test
[ ] Hiểu it là một test case
[ ] Hiểu expect dùng để kiểm tra kết quả
[ ] Hiểu beforeEach chạy trước mỗi test
[ ] Viết được test cho BooksService.findAll()
[ ] Viết được test cho BooksService.findOne()
[ ] Viết được test cho BooksService.create()
[ ] Viết được test cho AuthorsService
[ ] Viết được test cho CategoriesService
[ ] Có ít nhất 1 test dùng Arrange - Act - Assert
[ ] Có ít nhất 1 test kiểm tra NotFoundException
```

---

# 21. Mini quiz kiểm tra hiểu bài

## Câu 1

Unit test dùng để làm gì?

A. Kiểm tra một đơn vị nhỏ trong code
B. Chạy toàn bộ app thật trên production
C. Deploy backend lên server
D. Thiết kế database

<details>
<summary>Show answer</summary>

Đáp án đúng: **A. Kiểm tra một đơn vị nhỏ trong code**

Giải thích: Unit test thường kiểm tra từng service, function hoặc method riêng lẻ.

</details>

---

## Câu 2

Trong NestJS, file test thường có đuôi gì?

A. `.test.js`
B. `.spec.ts`
C. `.controller.ts`
D. `.module.ts`

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. `.spec.ts`**

Giải thích: NestJS mặc định tạo file test có dạng `*.spec.ts`.

</details>

---

## Câu 3

Lệnh nào dùng để chạy unit test trong NestJS?

A. `npm run start`
B. `npm run build`
C. `npm run test`
D. `npm run serve`

<details>
<summary>Show answer</summary>

Đáp án đúng: **C. `npm run test`**

Giải thích: Script `npm run test` dùng để chạy Jest test trong project NestJS.

</details>

---

## Câu 4

`describe` trong Jest dùng để làm gì?

A. Tạo database
B. Gom nhóm các test liên quan
C. Gửi HTTP request
D. Cấu hình port server

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Gom nhóm các test liên quan**

Giải thích: `describe('BooksService', () => {})` dùng để gom các test của `BooksService`.

</details>

---

## Câu 5

`it` trong Jest là gì?

A. Một test case cụ thể
B. Một service trong NestJS
C. Một controller
D. Một package npm

<details>
<summary>Show answer</summary>

Đáp án đúng: **A. Một test case cụ thể**

Giải thích: Mỗi `it()` mô tả một hành vi cần kiểm tra.

</details>

---

## Câu 6

`expect` dùng để làm gì?

A. Khởi động server
B. Kiểm tra kết quả thực tế có đúng như mong muốn không
C. Tạo module mới
D. Cài thư viện test

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Kiểm tra kết quả thực tế có đúng như mong muốn không**

Giải thích: `expect(result).toBe(4)` nghĩa là mong muốn `result` bằng `4`.

</details>

---

## Câu 7

`beforeEach` chạy khi nào?

A. Sau khi tất cả test chạy xong
B. Trước mỗi test case
C. Chỉ chạy một lần khi build project
D. Khi deploy server

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Trước mỗi test case**

Giải thích: `beforeEach` giúp chuẩn bị môi trường mới trước từng test case.

</details>

---

## Câu 8

Unit test cho Service có nên gọi HTTP thật không?

A. Có
B. Không

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Không**

Giải thích: Unit test Service nên gọi trực tiếp method của Service, không gọi HTTP thật.

</details>

---

## Câu 9

Matcher nào thường dùng để kiểm tra array có bao nhiêu phần tử?

A. `toBe()`
B. `toHaveLength()`
C. `toThrow()`
D. `toStart()`

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. `toHaveLength()`**

Giải thích: `expect(result).toHaveLength(2)` kiểm tra array có 2 phần tử.

</details>

---

## Câu 10

Matcher nào thường dùng để kiểm tra function có throw lỗi không?

A. `toThrow()`
B. `toEqual()`
C. `toHaveLength()`
D. `toBeNull()`

<details>
<summary>Show answer</summary>

Đáp án đúng: **A. `toThrow()`**

Giải thích: `expect(() => service.findOne(999)).toThrow()` dùng để kiểm tra function có throw lỗi.

</details>

---

# 22. Ghi nhớ sau Bài 5

Sau bài này, cần nhớ 7 ý chính:

```txt id="bj7kkc"
1. Unit test kiểm tra một phần nhỏ của code.
2. Jest là framework test mặc định trong NestJS.
3. File test thường có đuôi .spec.ts.
4. describe dùng để gom nhóm test.
5. it là một test case.
6. expect dùng để kiểm tra kết quả.
7. TestingModule giúp tạo môi trường test cho NestJS service.
```

---

# 23. Còn bao nhiêu bài trong giáo trình?

Giáo trình này có tổng cộng:

```txt id="blgxtk"
16 bài
```

Bạn đã có:

```txt id="e9ta4m"
Bài 1: NestJS là gì? Cài đặt môi trường
Bài 2: Controller, Service, Module
Bài 3: Tạo CRUD API cho Books
Bài 4: DTO và ValidationPipe
Bài 5: Jest và TestingModule
```

Vậy sau Bài 5 còn:

```txt id="wnt32w"
11 bài nữa
```

Danh sách các bài còn lại:

```txt id="vk5hl3"
Bài 6: Unit Test cho Service
Bài 7: Unit Test cho Controller
Bài 8: E2E Test với Supertest
Bài 9: Thêm Database Layer
Bài 10: Unit Test Service có Mock Database
Bài 11: Module Borrow - Mượn sách
Bài 12: Module Borrow - Trả sách
Bài 13: Guard và quyền truy cập API
Bài 14: E2E Test cho Auth
Bài 15: Refactor Project
Bài 16: Tổng ôn và kiểm tra cuối khóa
```

---

# 24. Chuẩn bị cho Bài 6

Ở Bài 5, ta mới viết test cơ bản cho:

```txt id="qhfk80"
findAll()
findOne()
create()
```

Sang Bài 6, ta sẽ viết test kỹ hơn cho `BooksService`, gồm cả case đúng và case lỗi:

```txt id="k7qpyp"
findAll()
findOne()
create()
update()
remove()
NotFoundException
```

Mục tiêu Bài 6:

```txt id="xns38w"
Ít nhất 8 test cases
Tất cả test phải pass
Không gọi HTTP thật
Không dùng database thật
```
