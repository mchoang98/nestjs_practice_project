# Bài 6: Unit Test cho Service

## 1. Mục tiêu bài học

Sau buổi này, bạn cần làm được:

1. Viết unit test đầy đủ hơn cho `BooksService`.
2. Biết test cả case đúng và case lỗi.
3. Biết kiểm tra exception bằng Jest.
4. Biết test `NotFoundException`.
5. Biết áp dụng mô hình Arrange – Act – Assert.
6. Biết chạy riêng một file test cụ thể.
7. Hoàn thành ít nhất 8 test cases cho `BooksService`.

Trong bài này, ta sẽ test các method:

```txt id="e5z8p7"
findAll()
findOne()
create()
update()
remove()
```

Yêu cầu quan trọng:

```txt id="o6mxkc"
Không gọi HTTP thật
Không dùng Postman
Không dùng database thật
Chỉ test trực tiếp BooksService
```

---

# 2. Ôn lại Bài 5

Ở Bài 5, ta đã học các khái niệm cơ bản:

```txt id="ccfzrw"
Unit test
Jest
File .spec.ts
describe
it
expect
TestingModule
```

Ta cũng đã viết một số test đơn giản cho `BooksService`:

```txt id="v1lwl3"
should be defined
should return all books
should return a book by id
should create a book
should add new book to books list
```

Tuy nhiên, các test ở Bài 5 chủ yếu là happy case.

Happy case nghĩa là trường hợp mọi thứ chạy đúng.

Ví dụ:

```txt id="i0ld5s"
findOne(1) tìm thấy book
create(data hợp lệ) tạo được book
findAll() trả về danh sách books
```

Ở Bài 6, ta cần test thêm error case.

Error case là trường hợp có lỗi xảy ra.

Ví dụ:

```txt id="i14ry5"
findOne(999) không tìm thấy book
update(999, data) không tìm thấy book
remove(999) không tìm thấy book
```

---

# 3. Vì sao cần test cả case đúng và case lỗi?

Một API tốt không chỉ chạy đúng khi dữ liệu hợp lệ.

API còn phải xử lý đúng khi dữ liệu không hợp lệ hoặc không tìm thấy dữ liệu.

Ví dụ với `BooksService.findOne()`:

```ts id="dvlcvu"
findOne(id: number): Book {
  const book = this.books.find((item) => item.id === id);

  if (!book) {
    throw new NotFoundException(`Book with id ${id} not found`);
  }

  return book;
}
```

Ta cần test cả 2 trường hợp:

```txt id="m67pn8"
findOne(1)   → trả về book
findOne(999) → throw NotFoundException
```

Nếu chỉ test `findOne(1)`, ta chưa chắc logic lỗi 404 có hoạt động đúng không.

---

# 4. Happy case và error case

## 4.1. Happy case

Happy case là trường hợp chạy thành công.

Ví dụ:

```txt id="rt4ztf"
findAll() trả về danh sách books
findOne(1) trả về book id = 1
create(data) tạo book mới
update(1, data) cập nhật book
remove(1) xóa book
```

## 4.2. Error case

Error case là trường hợp có lỗi cần được xử lý.

Ví dụ:

```txt id="qw4bwy"
findOne(999) không tìm thấy book
update(999, data) không tìm thấy book
remove(999) không tìm thấy book
```

Kết quả mong muốn:

```txt id="feu39e"
Throw NotFoundException
```

---

# 5. Checklist test cần viết cho BooksService

Trong bài này, ta sẽ viết test theo checklist sau:

```txt id="er7qk1"
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

Tổng cộng có thể viết ít nhất 9–10 test cases.

Giáo trình yêu cầu ít nhất 8 test cases.

---

# 6. Chuẩn bị BooksService

Giả sử `BooksService` hiện tại đang như sau:

```ts id="x3t2ax"
import { Injectable, NotFoundException } from '@nestjs/common';
import { Book } from './book.interface';
import { CreateBookDto } from './dto/create-book.dto';
import { UpdateBookDto } from './dto/update-book.dto';

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
    const nextId =
      this.books.length > 0
        ? Math.max(...this.books.map((book) => book.id)) + 1
        : 1;

    const newBook: Book = {
      id: nextId,
      ...data,
    };

    this.books.push(newBook);

    return newBook;
  }

  update(id: number, data: UpdateBookDto): Book {
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

Lưu ý: Nếu code của bạn đang tạo id bằng:

```ts id="ovsyk9"
id: this.books.length + 1
```

vẫn test được bài cơ bản.

Nhưng cách tốt hơn là lấy id lớn nhất + 1 để tránh trùng id sau khi xóa.

---

# 7. Cấu trúc file test

Mở file:

```txt id="tqkm46"
src/books/books.service.spec.ts
```

Cấu trúc cơ bản:

```ts id="jf9tm4"
import { NotFoundException } from '@nestjs/common';
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

Giải thích:

```txt id="tz76pb"
beforeEach chạy trước mỗi test
TestingModule tạo môi trường test cho Service
service là instance của BooksService
```

---

# 8. Test `findAll()`

Method thật:

```ts id="up45vl"
findAll(): Book[] {
  return this.books;
}
```

Test:

```ts id="ywoj5q"
it('should return all books', () => {
  const result = service.findAll();

  expect(result).toHaveLength(2);
  expect(result[0].title).toBe('Clean Code');
});
```

Ý nghĩa:

```txt id="t7gqph"
Gọi findAll()
Mong muốn trả về 2 books ban đầu
Book đầu tiên có title là Clean Code
```

---

# 9. Test `findOne()` tìm thấy book

Method thật:

```ts id="k8weev"
findOne(id: number): Book {
  const book = this.books.find((item) => item.id === id);

  if (!book) {
    throw new NotFoundException(`Book with id ${id} not found`);
  }

  return book;
}
```

Test happy case:

```ts id="a2fypl"
it('should return a book by id', () => {
  const result = service.findOne(1);

  expect(result).toBeDefined();
  expect(result.id).toBe(1);
  expect(result.title).toBe('Clean Code');
});
```

Ý nghĩa:

```txt id="b76tse"
Gọi findOne(1)
Mong muốn tìm thấy book
Book có id = 1
Book có title = Clean Code
```

---

# 10. Test `findOne()` không tìm thấy book

Khi gọi:

```ts id="iux3cc"
service.findOne(999)
```

Book không tồn tại, nên service phải throw `NotFoundException`.

Test:

```ts id="h1cvj4"
it('should throw NotFoundException when book does not exist', () => {
  expect(() => service.findOne(999)).toThrow(NotFoundException);
});
```

Giải thích:

```ts id="pqgfci"
expect(() => service.findOne(999))
```

Ta phải bọc function trong arrow function.

Không viết như này:

```ts id="sxatw4"
expect(service.findOne(999)).toThrow(NotFoundException);
```

Vì nếu viết như vậy, function sẽ chạy ngay lập tức trước khi Jest kiểm tra `toThrow()`.

Cách đúng:

```ts id="yhhk2v"
expect(() => service.findOne(999)).toThrow(NotFoundException);
```

---

# 11. Test `create()` tạo book mới

Method thật:

```ts id="lalwma"
create(data: CreateBookDto): Book {
  const nextId =
    this.books.length > 0
      ? Math.max(...this.books.map((book) => book.id)) + 1
      : 1;

  const newBook: Book = {
    id: nextId,
    ...data,
  };

  this.books.push(newBook);

  return newBook;
}
```

Test:

```ts id="ncdpe4"
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
  expect(result.author).toBe('Martin Fowler');
  expect(result.quantity).toBe(4);
});
```

Ý nghĩa:

```txt id="m37ovg"
Gọi create(data)
Mong muốn tạo được book mới
Book mới có id
Book mới có title đúng
Book mới có quantity đúng
```

---

# 12. Test `create()` thêm book vào danh sách

Test trước chỉ kiểm tra kết quả trả về.

Bây giờ test thêm xem danh sách có tăng số lượng không:

```ts id="j9no0p"
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
```

Ý nghĩa:

```txt id="d94yon"
Ban đầu có 2 books
Sau khi create thêm 1 book
Danh sách phải có 3 books
```

---

# 13. Test `update()` thành công

Method thật:

```ts id="eh06cz"
update(id: number, data: UpdateBookDto): Book {
  const book = this.findOne(id);

  Object.assign(book, data);

  return book;
}
```

Test:

```ts id="snc8fa"
it('should update a book', () => {
  const result = service.update(1, {
    quantity: 10,
  });

  expect(result.id).toBe(1);
  expect(result.quantity).toBe(10);
  expect(result.title).toBe('Clean Code');
});
```

Ý nghĩa:

```txt id="a976yd"
Gọi update(1, { quantity: 10 })
Book id = 1 phải có quantity mới là 10
Các field không update vẫn giữ nguyên
```

---

# 14. Test `update()` không tìm thấy book

Nếu gọi:

```ts id="mqpq1y"
service.update(999, { quantity: 10 })
```

Book không tồn tại, service phải throw `NotFoundException`.

Test:

```ts id="qs5wi8"
it('should throw NotFoundException when updating non-existing book', () => {
  expect(() => service.update(999, { quantity: 10 })).toThrow(
    NotFoundException,
  );
});
```

Lý do test này pass là vì `update()` gọi lại `findOne()`:

```ts id="q0ph7f"
const book = this.findOne(id);
```

Nếu không tìm thấy, `findOne()` throw lỗi.

---

# 15. Test `remove()` thành công

Method thật:

```ts id="s5ui5i"
remove(id: number): Book {
  const book = this.findOne(id);

  this.books = this.books.filter((item) => item.id !== id);

  return book;
}
```

Test:

```ts id="m4bmtg"
it('should remove a book', () => {
  const result = service.remove(1);

  expect(result.id).toBe(1);

  const books = service.findAll();

  expect(books).toHaveLength(1);
  expect(() => service.findOne(1)).toThrow(NotFoundException);
});
```

Ý nghĩa:

```txt id="puq7dg"
Xóa book id = 1
Service trả về book vừa xóa
Danh sách còn 1 book
Gọi lại findOne(1) phải throw NotFoundException
```

---

# 16. Test `remove()` không tìm thấy book

Nếu gọi:

```ts id="xz5qcb"
service.remove(999)
```

Book không tồn tại, service phải throw `NotFoundException`.

Test:

```ts id="e1yiuh"
it('should throw NotFoundException when removing non-existing book', () => {
  expect(() => service.remove(999)).toThrow(NotFoundException);
});
```

---

# 17. Test nâng cao: tạo id không trùng sau khi xóa

Nếu service dùng cách tạo id tốt:

```ts id="sgnv54"
const nextId =
  this.books.length > 0
    ? Math.max(...this.books.map((book) => book.id)) + 1
    : 1;
```

Ta có thể test như sau:

```ts id="q2u95b"
it('should create new book with next max id after removing a book', () => {
  service.remove(2);

  const result = service.create({
    title: 'Working Effectively with Legacy Code',
    author: 'Michael Feathers',
    isbn: '9780131177055',
    quantity: 1,
  });

  expect(result.id).toBe(3);
});
```

Trường hợp này kiểm tra:

```txt id="a60xg6"
Ban đầu có id 1, 2
Xóa id 2
Tạo book mới
Book mới nên có id 3, không phải id 2
```

Nếu test fail và nhận id = 2, nghĩa là service vẫn đang dùng:

```ts id="ovh6v6"
this.books.length + 1
```

Cách đó dễ tạo id trùng trong một số trường hợp.

---

# 18. File `books.service.spec.ts` hoàn chỉnh

```ts id="i7b5ij"
import { NotFoundException } from '@nestjs/common';
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

  it('should throw NotFoundException when book does not exist', () => {
    expect(() => service.findOne(999)).toThrow(NotFoundException);
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
    expect(result.author).toBe('Martin Fowler');
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

  it('should update a book', () => {
    const result = service.update(1, {
      quantity: 10,
    });

    expect(result.id).toBe(1);
    expect(result.quantity).toBe(10);
    expect(result.title).toBe('Clean Code');
  });

  it('should throw NotFoundException when updating non-existing book', () => {
    expect(() => service.update(999, { quantity: 10 })).toThrow(
      NotFoundException,
    );
  });

  it('should remove a book', () => {
    const result = service.remove(1);

    expect(result.id).toBe(1);

    const books = service.findAll();

    expect(books).toHaveLength(1);
    expect(() => service.findOne(1)).toThrow(NotFoundException);
  });

  it('should throw NotFoundException when removing non-existing book', () => {
    expect(() => service.remove(999)).toThrow(NotFoundException);
  });

  it('should create new book with next max id after removing a book', () => {
    service.remove(2);

    const result = service.create({
      title: 'Working Effectively with Legacy Code',
      author: 'Michael Feathers',
      isbn: '9780131177055',
      quantity: 1,
    });

    expect(result.id).toBe(3);
  });
});
```

File trên có 11 test cases.

Đã vượt yêu cầu tối thiểu 8 test cases.

---

# 19. Chạy test

Chạy toàn bộ test:

```bash id="xzzqks"
npm run test
```

Chạy test ở chế độ watch:

```bash id="ag9k94"
npm run test:watch
```

Chạy riêng file `books.service.spec.ts`:

```bash id="vopn40"
npm run test -- books.service.spec.ts
```

Hoặc:

```bash id="g0tqau"
npx jest books.service.spec.ts
```

Kết quả mong muốn:

```txt id="w4ibaf"
PASS src/books/books.service.spec.ts
BooksService
  ✓ should be defined
  ✓ should return all books
  ✓ should return a book by id
  ✓ should throw NotFoundException when book does not exist
  ✓ should create a book
  ✓ should add new book to books list
  ✓ should update a book
  ✓ should throw NotFoundException when updating non-existing book
  ✓ should remove a book
  ✓ should throw NotFoundException when removing non-existing book
  ✓ should create new book with next max id after removing a book
```

---

# 20. Giải thích Arrange – Act – Assert qua test thực tế

Lấy ví dụ test `create()`:

```ts id="szvliy"
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
  expect(result).toBeDefined();
  expect(result.id).toBeDefined();
  expect(result.title).toBe('Refactoring');
  expect(result.quantity).toBe(4);
});
```

Giải thích:

```txt id="nhdzkz"
Arrange: chuẩn bị dữ liệu test
Act: gọi method cần test
Assert: kiểm tra kết quả
```

Cách viết này rất nên dùng khi test bắt đầu dài hơn.

Nó giúp người đọc dễ hiểu test đang làm gì.

---

# 21. Những lỗi thường gặp khi test exception

## Lỗi 1: Gọi function trực tiếp trong expect

Sai:

```ts id="la9hwn"
expect(service.findOne(999)).toThrow(NotFoundException);
```

Đúng:

```ts id="b5lavh"
expect(() => service.findOne(999)).toThrow(NotFoundException);
```

Lý do:

```txt id="q0fyar"
toThrow cần nhận một function.
Nếu gọi service.findOne(999) trực tiếp, lỗi sẽ xảy ra trước khi Jest kiểm tra.
```

---

## Lỗi 2: Không import NotFoundException

Nếu dùng:

```ts id="f3i3br"
toThrow(NotFoundException)
```

thì phải import:

```ts id="jth4w2"
import { NotFoundException } from '@nestjs/common';
```

Nếu quên import, TypeScript sẽ báo lỗi.

---

## Lỗi 3: Test phụ thuộc thứ tự chạy

Không nên viết test mà phụ thuộc vào test trước.

Ví dụ không nên nghĩ rằng test trước đã tạo book, rồi test sau dùng book đó.

Vì mỗi test nên độc lập.

Trong file test, `beforeEach` tạo lại service mới trước mỗi test:

```ts id="dhb2w7"
beforeEach(async () => {
  const module: TestingModule = await Test.createTestingModule({
    providers: [BooksService],
  }).compile();

  service = module.get<BooksService>(BooksService);
});
```

Nhờ vậy mỗi test bắt đầu với dữ liệu sạch.

---

# 22. Có nên test DTO trong Service test không?

Không nên test DTO trong unit test của Service.

Ví dụ DTO validate:

```txt id="b2sfiw"
title không được rỗng
quantity phải >= 0
field dư bị reject
```

Những phần này thuộc về validation layer.

Nên test bằng:

```txt id="n2aaxv"
E2E test
Hoặc test riêng pipe/validation nếu cần
```

Service test nên tập trung vào business logic:

```txt id="lvw42l"
Tìm sách
Tạo sách
Cập nhật sách
Xóa sách
Throw exception khi không tìm thấy
```

---

# 23. Có nên gọi Controller trong Service test không?

Không.

Service test chỉ test Service.

Không gọi:

```ts id="o927dp"
controller.findAll()
```

Chỉ gọi:

```ts id="u47s05"
service.findAll()
```

Controller sẽ được test riêng ở Bài 7.

---

# 24. Checklist hoàn thành Bài 6

Bạn đạt yêu cầu Bài 6 nếu làm được:

```txt id="e340xs"
[ ] Hiểu happy case là gì
[ ] Hiểu error case là gì
[ ] Hiểu Arrange – Act – Assert
[ ] Viết được test cho findAll()
[ ] Viết được test cho findOne() success
[ ] Viết được test cho findOne() fail
[ ] Viết được test cho create()
[ ] Viết được test create làm danh sách tăng số lượng
[ ] Viết được test cho update() success
[ ] Viết được test cho update() fail
[ ] Viết được test cho remove() success
[ ] Viết được test cho remove() fail
[ ] Biết dùng toThrow(NotFoundException)
[ ] Có ít nhất 8 test cases
[ ] Tất cả test pass
[ ] Không gọi HTTP thật
[ ] Không dùng database thật
```

---

# 25. Bài tập thực hành cuối buổi

Lưu ý: Bài tập dùng module khác ví dụ chính để bạn tự luyện lại unit test Service.

## Bài tập 1: Viết test đầy đủ cho AuthorsService

Giả sử `AuthorsService` có các method:

```txt id="hjbtzk"
findAll()
findOne(id)
create(data)
update(id, data)
remove(id)
```

Author có cấu trúc:

```ts id="hjy2ad"
export interface Author {
  id: number;
  name: string;
  country: string;
  birthYear: number;
}
```

Dữ liệu mẫu:

```ts id="c8hnpv"
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

```txt id="sw3w84"
findAll() trả về danh sách authors
findOne(1) trả về Nguyễn Nhật Ánh
findOne(999) throw NotFoundException
create() tạo author mới
create() làm danh sách tăng từ 2 lên 3
update(1, data) cập nhật author thành công
update(999, data) throw NotFoundException
remove(1) xóa author thành công
remove(999) throw NotFoundException
```

---

## Bài tập 2: Viết test đầy đủ cho CategoriesService

Category có cấu trúc:

```ts id="mwfwvg"
export interface Category {
  id: number;
  name: string;
  description: string;
}
```

Dữ liệu mẫu:

```ts id="q0ec8c"
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

```txt id="kf9y1y"
findAll() trả về 2 categories
findOne(2) trả về Novel
findOne(999) throw NotFoundException
create() tạo category mới
update(1, { name: 'Software Engineering' }) cập nhật name
remove(2) xóa category
remove(999) throw NotFoundException
```

---

## Bài tập 3: Nâng cao 1 - Test không cho update id

Nếu bạn đã xử lý không cho cập nhật `id`, hãy viết test:

```txt id="w2z20w"
update(1, { id: 999, name: 'Updated Name' })
```

Kết quả mong muốn:

```txt id="hqw7ck"
Author vẫn có id = 1
Name được cập nhật
id không bị đổi thành 999
```

Gợi ý:

```ts id="rtf33k"
it('should not allow updating author id', () => {
  const result = service.update(1, {
    id: 999,
    name: 'Updated Name',
  } as any);

  expect(result.id).toBe(1);
  expect(result.name).toBe('Updated Name');
});
```

Lưu ý:

```txt id="nrclev"
as any chỉ dùng trong test để giả lập client gửi dữ liệu sai.
Trong code thật, DTO/type nên ngăn id từ đầu.
```

---

## Bài tập 4: Nâng cao 2 - Test tạo id không trùng sau khi xóa

Viết test cho `AuthorsService`:

```txt id="q7owfl"
Ban đầu có id 1, 2
Xóa id 2
Tạo author mới
Author mới phải có id = 3
```

Gợi ý:

```ts id="hz44th"
it('should create author with next max id after remove', () => {
  service.remove(2);

  const result = service.create({
    name: 'George Orwell',
    country: 'United Kingdom',
    birthYear: 1903,
  });

  expect(result.id).toBe(3);
});
```

---

## Bài tập 5: Nâng cao 3 - Test remove xong không tìm lại được

Sau khi xóa category id = 1:

```ts id="j4bu3n"
service.remove(1);
```

Gọi lại:

```ts id="fhyaox"
service.findOne(1);
```

Phải throw `NotFoundException`.

Gợi ý:

```ts id="vlbk5b"
it('should not find category after removing it', () => {
  service.remove(1);

  expect(() => service.findOne(1)).toThrow(NotFoundException);
});
```

---

## Bài tập 6: Checklist tự kiểm tra

Sau khi hoàn thành, tự kiểm tra:

```txt id="rytvay"
[ ] AuthorsService có ít nhất 8 test cases
[ ] CategoriesService có ít nhất 7 test cases
[ ] Có test success case
[ ] Có test error case
[ ] Có test NotFoundException
[ ] Có test create làm tăng số lượng item
[ ] Có test update thành công
[ ] Có test remove thành công
[ ] Có test remove xong không tìm lại được
[ ] Tất cả test pass
```

---

# 26. Mini quiz kiểm tra hiểu bài

## Câu 1

Trong unit test Service, ta có nên gọi HTTP thật không?

A. Có
B. Không

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Không**

Giải thích: Unit test Service chỉ gọi trực tiếp method của Service, không gửi HTTP request thật.

</details>

---

## Câu 2

Happy case là gì?

A. Trường hợp chạy thành công
B. Trường hợp database bị lỗi
C. Trường hợp request sai dữ liệu
D. Trường hợp server bị tắt

<details>
<summary>Show answer</summary>

Đáp án đúng: **A. Trường hợp chạy thành công**

Giải thích: Happy case là luồng xử lý chính khi dữ liệu hợp lệ và không có lỗi.

</details>

---

## Câu 3

Error case là gì?

A. Trường hợp code được format đẹp
B. Trường hợp có lỗi cần xử lý
C. Trường hợp test luôn pass
D. Trường hợp chạy build production

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Trường hợp có lỗi cần xử lý**

Giải thích: Error case kiểm tra hệ thống xử lý lỗi có đúng không, ví dụ không tìm thấy dữ liệu thì throw exception.

</details>

---

## Câu 4

Matcher nào dùng để kiểm tra một function có throw lỗi?

A. `toBe()`
B. `toEqual()`
C. `toThrow()`
D. `toHaveLength()`

<details>
<summary>Show answer</summary>

Đáp án đúng: **C. `toThrow()`**

Giải thích: `toThrow()` dùng để kiểm tra function có throw lỗi hay không.

</details>

---

## Câu 5

Cách viết nào đúng khi test exception?

A. `expect(service.findOne(999)).toThrow(NotFoundException)`
B. `expect(() => service.findOne(999)).toThrow(NotFoundException)`
C. `expect(service.findOne).toEqual(NotFoundException)`
D. `expect(NotFoundException).toBe(service.findOne(999))`

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. `expect(() => service.findOne(999)).toThrow(NotFoundException)`**

Giải thích: `toThrow()` cần nhận một function, nên phải bọc lời gọi bằng arrow function.

</details>

---

## Câu 6

Arrange – Act – Assert có nghĩa là gì?

A. Cài đặt – Build – Deploy
B. Chuẩn bị dữ liệu – Gọi function – Kiểm tra kết quả
C. Tạo module – Tạo controller – Tạo service
D. Request – Response – Database

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Chuẩn bị dữ liệu – Gọi function – Kiểm tra kết quả**

Giải thích: Đây là cách tổ chức test giúp code test dễ đọc hơn.

</details>

---

## Câu 7

`beforeEach` trong test dùng để làm gì?

A. Chạy trước mỗi test case
B. Chạy sau mỗi test case
C. Chỉ chạy khi deploy
D. Chỉ chạy khi test fail

<details>
<summary>Show answer</summary>

Đáp án đúng: **A. Chạy trước mỗi test case**

Giải thích: `beforeEach` thường dùng để tạo lại service/module mới trước từng test.

</details>

---

## Câu 8

Khi `findOne(999)` không tìm thấy book, service nên throw lỗi gì?

A. `BadRequestException`
B. `NotFoundException`
C. `UnauthorizedException`
D. `ForbiddenException`

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. `NotFoundException`**

Giải thích: Không tìm thấy resource thì nên dùng `NotFoundException`, tương ứng HTTP 404.

</details>

---

## Câu 9

Unit test Service nên tập trung kiểm tra điều gì?

A. Giao diện người dùng
B. Business logic trong Service
C. CSS layout
D. Deploy server

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Business logic trong Service**

Giải thích: Service chứa logic chính như tạo, tìm, cập nhật, xóa dữ liệu.

</details>

---

## Câu 10

Vì sao mỗi test case nên độc lập?

A. Để test dễ đọc và không phụ thuộc thứ tự chạy
B. Để chạy server nhanh hơn
C. Để không cần viết expect
D. Để bỏ qua lỗi TypeScript

<details>
<summary>Show answer</summary>

Đáp án đúng: **A. Để test dễ đọc và không phụ thuộc thứ tự chạy**

Giải thích: Nếu test phụ thuộc nhau, khi đổi thứ tự chạy hoặc một test fail, các test khác có thể bị ảnh hưởng.

</details>

---

# 27. Ghi nhớ sau Bài 6

Sau bài này, cần nhớ 7 ý chính:

```txt id="dbvqo3"
1. Service test dùng để kiểm tra business logic.
2. Cần test cả happy case và error case.
3. NotFoundException nên được test khi dữ liệu không tồn tại.
4. toThrow() dùng để kiểm tra exception.
5. Khi test exception phải bọc function trong arrow function.
6. Arrange – Act – Assert giúp test dễ đọc hơn.
7. Unit test Service không gọi HTTP thật và không dùng database thật.
```

---

# 28. Chuẩn bị cho Bài 7

Sau Bài 6, bạn đã biết test Service thật.

Ở Bài 7, ta sẽ test Controller.

Điểm khác biệt quan trọng:

```txt id="i1tj5g"
Bài 6: Test Service thật
Bài 7: Test Controller bằng mock Service
```

Nội dung chính của Bài 7:

```txt id="j1vzp3"
Mock service bằng jest.fn()
Kiểm tra controller gọi đúng service
Kiểm tra controller truyền đúng tham số
Kiểm tra controller trả đúng response
```

API controller cần test:

```txt id="d8892z"
BooksController.findAll()
BooksController.findOne()
BooksController.create()
BooksController.update()
BooksController.remove()
```
