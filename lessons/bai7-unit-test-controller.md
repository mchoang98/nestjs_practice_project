# Bài 7: Unit Test cho Controller

## 1. Mục tiêu bài học

Sau buổi này, bạn cần làm được:

1. Hiểu unit test Controller khác unit test Service như thế nào.
2. Biết vì sao cần mock Service khi test Controller.
3. Biết dùng `jest.fn()` để tạo mock function.
4. Biết kiểm tra Controller gọi đúng method của Service.
5. Biết kiểm tra Controller truyền đúng tham số vào Service.
6. Biết kiểm tra Controller trả đúng response.
7. Viết được unit test cho:

```txt id="n0yr43"
BooksController.findAll()
BooksController.findOne()
BooksController.create()
BooksController.update()
BooksController.remove()
```

Trong bài này, ta không dùng Service thật.

Ta sẽ mock `BooksService`.

---

# 2. Ôn lại Bài 6

Ở Bài 6, ta đã viết unit test cho `BooksService`.

Service test tập trung vào business logic:

```txt id="u7vqgu"
findAll() trả về danh sách books
findOne() tìm book theo id
findOne() không tìm thấy thì throw NotFoundException
create() tạo book mới
update() cập nhật book
remove() xóa book
```

Khi test Service, ta gọi trực tiếp:

```ts id="gyp5z6"
service.findAll()
service.findOne(1)
service.create(data)
```

Nhưng ở Bài 7, ta test Controller.

Controller không nên chứa logic phức tạp.

Controller chủ yếu làm việc này:

```txt id="tjgtga"
Nhận request
Lấy param/body
Gọi service
Trả kết quả từ service
```

Vì vậy khi test Controller, ta không cần test lại toàn bộ logic của Service.

---

# 3. Controller test khác Service test như thế nào?

## 3.1. Service test

Service test kiểm tra logic thật.

Ví dụ:

```ts id="b2290p"
const result = service.findOne(1);

expect(result.id).toBe(1);
expect(result.title).toBe('Clean Code');
```

Ở đây ta kiểm tra `BooksService.findOne()` có tìm đúng book không.

---

## 3.2. Controller test

Controller test kiểm tra Controller có gọi Service đúng không.

Ví dụ:

```ts id="k61ef8"
const result = controller.findOne(1);

expect(service.findOne).toHaveBeenCalledWith(1);
expect(result).toEqual(mockBook);
```

Ở đây ta không quan tâm Service thật tìm book như thế nào.

Ta chỉ kiểm tra:

```txt id="dmntc7"
Controller có gọi service.findOne(1) không?
Controller có trả kết quả từ service không?
```

---

# 4. Vì sao phải mock Service?

Nếu test Controller mà dùng Service thật, test sẽ phụ thuộc vào logic của Service.

Ví dụ:

```txt id="i7xlma"
BooksController test fail
```

Lúc đó chưa chắc lỗi nằm ở Controller.

Có thể lỗi nằm ở:

```txt id="ezdx2v"
BooksService.findOne()
BooksService.update()
BooksService.remove()
Dữ liệu trong array
Logic tạo id
Logic throw exception
```

Như vậy test không còn cô lập.

Mục tiêu của unit test là kiểm tra từng phần nhỏ độc lập.

Vì vậy:

```txt id="e2zd2t"
Test Service thì dùng Service thật.
Test Controller thì mock Service.
```

---

# 5. Mock là gì?

Mock là object giả dùng để thay thế object thật trong test.

Ví dụ Service thật:

```ts id="dkq2op"
class BooksService {
  findAll() {
    return this.books;
  }

  findOne(id: number) {
    return this.books.find((book) => book.id === id);
  }
}
```

Mock Service:

```ts id="nmnule"
const mockBooksService = {
  findAll: jest.fn(),
  findOne: jest.fn(),
  create: jest.fn(),
  update: jest.fn(),
  remove: jest.fn(),
};
```

Ở đây, mỗi method là một mock function.

Ta có thể chủ động điều khiển kết quả trả về:

```ts id="fsd58y"
mockBooksService.findAll.mockReturnValue([mockBook]);
```

Hoặc:

```ts id="obp6my"
mockBooksService.findOne.mockReturnValue(mockBook);
```

---

# 6. `jest.fn()` là gì?

`jest.fn()` dùng để tạo function giả.

Ví dụ:

```ts id="ick8gu"
const mockFunction = jest.fn();
```

Ta có thể kiểm tra function đó đã được gọi chưa:

```ts id="eaf865"
expect(mockFunction).toHaveBeenCalled();
```

Kiểm tra được gọi với tham số nào:

```ts id="ix4bpk"
expect(mockFunction).toHaveBeenCalledWith(1);
```

Cho function giả trả về dữ liệu:

```ts id="y6nqw7"
mockFunction.mockReturnValue('Hello');
```

Ví dụ đầy đủ:

```ts id="bkpoxl"
const getName = jest.fn();

getName.mockReturnValue('Phu');

const result = getName();

expect(result).toBe('Phu');
expect(getName).toHaveBeenCalled();
```

---

# 7. Chuẩn bị BooksController

Giả sử `BooksController` hiện tại như sau:

```ts id="ww3ui8"
import {
  Body,
  Controller,
  Delete,
  Get,
  Param,
  ParseIntPipe,
  Patch,
  Post,
} from '@nestjs/common';
import { Book } from './book.interface';
import { BooksService } from './books.service';
import { CreateBookDto } from './dto/create-book.dto';
import { UpdateBookDto } from './dto/update-book.dto';

@Controller('books')
export class BooksController {
  constructor(private readonly booksService: BooksService) {}

  @Get()
  findAll(): Book[] {
    return this.booksService.findAll();
  }

  @Get(':id')
  findOne(@Param('id', ParseIntPipe) id: number): Book {
    return this.booksService.findOne(id);
  }

  @Post()
  create(@Body() body: CreateBookDto): Book {
    return this.booksService.create(body);
  }

  @Patch(':id')
  update(
    @Param('id', ParseIntPipe) id: number,
    @Body() body: UpdateBookDto,
  ): Book {
    return this.booksService.update(id, body);
  }

  @Delete(':id')
  remove(@Param('id', ParseIntPipe) id: number): Book {
    return this.booksService.remove(id);
  }
}
```

Lưu ý:

```txt id="cxl0zm"
Controller không tự xử lý logic.
Controller chỉ gọi BooksService.
```

Đây là thiết kế tốt để dễ test.

---

# 8. Tạo file test cho Controller

File test nằm tại:

```txt id="b5bjg5"
src/books/books.controller.spec.ts
```

Nếu file đã có sẵn từ CLI, ta sẽ sửa lại.

Nếu chưa có, tạo file mới.

---

# 9. Cấu trúc test controller cơ bản

Cấu trúc ban đầu:

```ts id="ik5ru6"
import { Test, TestingModule } from '@nestjs/testing';
import { BooksController } from './books.controller';
import { BooksService } from './books.service';

describe('BooksController', () => {
  let controller: BooksController;

  beforeEach(async () => {
    const module: TestingModule = await Test.createTestingModule({
      controllers: [BooksController],
      providers: [BooksService],
    }).compile();

    controller = module.get<BooksController>(BooksController);
  });

  it('should be defined', () => {
    expect(controller).toBeDefined();
  });
});
```

Nhưng code này đang dùng `BooksService` thật.

Trong bài này, ta cần thay bằng mock service.

---

# 10. Tạo mock BooksService

Tạo mock object:

```ts id="n1f6kg"
const mockBooksService = {
  findAll: jest.fn(),
  findOne: jest.fn(),
  create: jest.fn(),
  update: jest.fn(),
  remove: jest.fn(),
};
```

Sau đó dùng mock này trong TestingModule:

```ts id="wh9y6j"
const module: TestingModule = await Test.createTestingModule({
  controllers: [BooksController],
  providers: [
    {
      provide: BooksService,
      useValue: mockBooksService,
    },
  ],
}).compile();
```

Ý nghĩa:

```txt id="zcuzqz"
Khi BooksController cần BooksService,
NestJS sẽ đưa mockBooksService vào thay vì BooksService thật.
```

---

# 11. File test setup hoàn chỉnh

```ts id="ppz07r"
import { Test, TestingModule } from '@nestjs/testing';
import { BooksController } from './books.controller';
import { BooksService } from './books.service';

describe('BooksController', () => {
  let controller: BooksController;
  let service: typeof mockBooksService;

  const mockBooksService = {
    findAll: jest.fn(),
    findOne: jest.fn(),
    create: jest.fn(),
    update: jest.fn(),
    remove: jest.fn(),
  };

  beforeEach(async () => {
    jest.clearAllMocks();

    const module: TestingModule = await Test.createTestingModule({
      controllers: [BooksController],
      providers: [
        {
          provide: BooksService,
          useValue: mockBooksService,
        },
      ],
    }).compile();

    controller = module.get<BooksController>(BooksController);
    service = module.get(BooksService);
  });

  it('should be defined', () => {
    expect(controller).toBeDefined();
  });
});
```

Giải thích:

```ts id="hw2pzh"
jest.clearAllMocks();
```

Dòng này xóa lịch sử gọi mock trước mỗi test.

Nếu không clear, test sau có thể bị ảnh hưởng bởi test trước.

---

# 12. Chuẩn bị mock data

Thêm dữ liệu giả để test:

```ts id="mgp5nm"
const mockBook = {
  id: 1,
  title: 'Clean Code',
  author: 'Robert C. Martin',
  isbn: '9780132350884',
  quantity: 5,
};

const mockBooks = [
  mockBook,
  {
    id: 2,
    title: 'The Pragmatic Programmer',
    author: 'Andrew Hunt, David Thomas',
    isbn: '9780201616224',
    quantity: 3,
  },
];
```

Dữ liệu này chỉ dùng trong test Controller.

---

# 13. Test `findAll()`

Controller method:

```ts id="qvfo74"
@Get()
findAll(): Book[] {
  return this.booksService.findAll();
}
```

Test:

```ts id="ewmsq1"
it('should return all books', () => {
  service.findAll.mockReturnValue(mockBooks);

  const result = controller.findAll();

  expect(service.findAll).toHaveBeenCalled();
  expect(result).toEqual(mockBooks);
});
```

Giải thích:

```ts id="rgwdi8"
service.findAll.mockReturnValue(mockBooks);
```

Giả lập khi gọi `service.findAll()` thì trả về `mockBooks`.

```ts id="z6tang"
const result = controller.findAll();
```

Gọi method của Controller.

```ts id="frf7ji"
expect(service.findAll).toHaveBeenCalled();
```

Kiểm tra Controller có gọi Service hay không.

```ts id="axvi2g"
expect(result).toEqual(mockBooks);
```

Kiểm tra kết quả Controller trả ra có đúng không.

---

# 14. Test `findOne()`

Controller method:

```ts id="cpq2g8"
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number): Book {
  return this.booksService.findOne(id);
}
```

Test:

```ts id="bfh586"
it('should return a book by id', () => {
  service.findOne.mockReturnValue(mockBook);

  const result = controller.findOne(1);

  expect(service.findOne).toHaveBeenCalledWith(1);
  expect(result).toEqual(mockBook);
});
```

Ở đây cần kiểm tra 2 việc:

```txt id="vnux4m"
Controller gọi service.findOne với id = 1
Controller trả về mockBook
```

Lưu ý:

```txt id="wh6pd2"
Unit test Controller không kiểm tra ParseIntPipe.
ParseIntPipe thuộc framework/runtime behavior, thường test kỹ hơn trong e2e test.
```

---

# 15. Test `create()`

Controller method:

```ts id="auq80y"
@Post()
create(@Body() body: CreateBookDto): Book {
  return this.booksService.create(body);
}
```

Test:

```ts id="y6a6c3"
it('should create a book', () => {
  const createBookDto = {
    title: 'Refactoring',
    author: 'Martin Fowler',
    isbn: '9780201485677',
    quantity: 4,
  };

  const createdBook = {
    id: 3,
    ...createBookDto,
  };

  service.create.mockReturnValue(createdBook);

  const result = controller.create(createBookDto);

  expect(service.create).toHaveBeenCalledWith(createBookDto);
  expect(result).toEqual(createdBook);
});
```

Test này không kiểm tra logic tạo id.

Vì logic tạo id thuộc Service.

Controller test chỉ kiểm tra:

```txt id="sdgq8u"
Controller nhận body
Controller truyền body vào service.create()
Controller trả kết quả từ service.create()
```

---

# 16. Test `update()`

Controller method:

```ts id="hqbr5a"
@Patch(':id')
update(
  @Param('id', ParseIntPipe) id: number,
  @Body() body: UpdateBookDto,
): Book {
  return this.booksService.update(id, body);
}
```

Test:

```ts id="n6s5pr"
it('should update a book', () => {
  const updateBookDto = {
    quantity: 10,
  };

  const updatedBook = {
    ...mockBook,
    quantity: 10,
  };

  service.update.mockReturnValue(updatedBook);

  const result = controller.update(1, updateBookDto);

  expect(service.update).toHaveBeenCalledWith(1, updateBookDto);
  expect(result).toEqual(updatedBook);
});
```

Kiểm tra:

```txt id="s2d8s2"
Controller gọi service.update(1, updateBookDto)
Controller trả về updatedBook
```

---

# 17. Test `remove()`

Controller method:

```ts id="tid0jx"
@Delete(':id')
remove(@Param('id', ParseIntPipe) id: number): Book {
  return this.booksService.remove(id);
}
```

Test:

```ts id="x7081a"
it('should remove a book', () => {
  service.remove.mockReturnValue(mockBook);

  const result = controller.remove(1);

  expect(service.remove).toHaveBeenCalledWith(1);
  expect(result).toEqual(mockBook);
});
```

Kiểm tra:

```txt id="d9muh3"
Controller gọi service.remove(1)
Controller trả về book đã xóa
```

---

# 18. File `books.controller.spec.ts` hoàn chỉnh

```ts id="t8e05c"
import { Test, TestingModule } from '@nestjs/testing';
import { BooksController } from './books.controller';
import { BooksService } from './books.service';

describe('BooksController', () => {
  let controller: BooksController;
  let service: typeof mockBooksService;

  const mockBook = {
    id: 1,
    title: 'Clean Code',
    author: 'Robert C. Martin',
    isbn: '9780132350884',
    quantity: 5,
  };

  const mockBooks = [
    mockBook,
    {
      id: 2,
      title: 'The Pragmatic Programmer',
      author: 'Andrew Hunt, David Thomas',
      isbn: '9780201616224',
      quantity: 3,
    },
  ];

  const mockBooksService = {
    findAll: jest.fn(),
    findOne: jest.fn(),
    create: jest.fn(),
    update: jest.fn(),
    remove: jest.fn(),
  };

  beforeEach(async () => {
    jest.clearAllMocks();

    const module: TestingModule = await Test.createTestingModule({
      controllers: [BooksController],
      providers: [
        {
          provide: BooksService,
          useValue: mockBooksService,
        },
      ],
    }).compile();

    controller = module.get<BooksController>(BooksController);
    service = module.get(BooksService);
  });

  it('should be defined', () => {
    expect(controller).toBeDefined();
  });

  it('should return all books', () => {
    service.findAll.mockReturnValue(mockBooks);

    const result = controller.findAll();

    expect(service.findAll).toHaveBeenCalled();
    expect(result).toEqual(mockBooks);
  });

  it('should return a book by id', () => {
    service.findOne.mockReturnValue(mockBook);

    const result = controller.findOne(1);

    expect(service.findOne).toHaveBeenCalledWith(1);
    expect(result).toEqual(mockBook);
  });

  it('should create a book', () => {
    const createBookDto = {
      title: 'Refactoring',
      author: 'Martin Fowler',
      isbn: '9780201485677',
      quantity: 4,
    };

    const createdBook = {
      id: 3,
      ...createBookDto,
    };

    service.create.mockReturnValue(createdBook);

    const result = controller.create(createBookDto);

    expect(service.create).toHaveBeenCalledWith(createBookDto);
    expect(result).toEqual(createdBook);
  });

  it('should update a book', () => {
    const updateBookDto = {
      quantity: 10,
    };

    const updatedBook = {
      ...mockBook,
      quantity: 10,
    };

    service.update.mockReturnValue(updatedBook);

    const result = controller.update(1, updateBookDto);

    expect(service.update).toHaveBeenCalledWith(1, updateBookDto);
    expect(result).toEqual(updatedBook);
  });

  it('should remove a book', () => {
    service.remove.mockReturnValue(mockBook);

    const result = controller.remove(1);

    expect(service.remove).toHaveBeenCalledWith(1);
    expect(result).toEqual(mockBook);
  });
});
```

---

# 19. Chạy test

Chạy lệnh:

```bash id="abgm5p"
npm run test
```

Nếu chỉ muốn chạy file controller spec:

```bash id="g2vzae"
npm run test -- books.controller.spec.ts
```

Hoặc nếu dùng Jest trực tiếp:

```bash id="i9u328"
npx jest books.controller.spec.ts
```

Kết quả mong muốn:

```txt id="bwlh39"
PASS src/books/books.controller.spec.ts
BooksController
  ✓ should be defined
  ✓ should return all books
  ✓ should return a book by id
  ✓ should create a book
  ✓ should update a book
  ✓ should remove a book
```

---

# 20. Những lỗi thường gặp khi test Controller

## Lỗi 1: Quên provide mock service

Nếu viết:

```ts id="ky3n74"
const module: TestingModule = await Test.createTestingModule({
  controllers: [BooksController],
}).compile();
```

Có thể bị lỗi:

```txt id="cql7wm"
Nest can't resolve dependencies of the BooksController
```

Nguyên nhân:

```txt id="gg3q6l"
BooksController cần BooksService nhưng TestingModule chưa provide BooksService.
```

Cách sửa:

```ts id="q03zt0"
providers: [
  {
    provide: BooksService,
    useValue: mockBooksService,
  },
],
```

---

## Lỗi 2: Dùng Service thật thay vì mock Service

Không nên viết:

```ts id="n7o76f"
providers: [BooksService]
```

Trong bài test Controller này, nên viết:

```ts id="yxda69"
providers: [
  {
    provide: BooksService,
    useValue: mockBooksService,
  },
]
```

Vì mục tiêu là test Controller độc lập.

---

## Lỗi 3: Quên `mockReturnValue`

Ví dụ:

```ts id="tvu4w1"
const result = controller.findAll();

expect(result).toEqual(mockBooks);
```

Nhưng chưa set:

```ts id="eg4lio"
service.findAll.mockReturnValue(mockBooks);
```

Khi đó `result` có thể là `undefined`.

Cách sửa:

```ts id="r39gmt"
service.findAll.mockReturnValue(mockBooks);
```

---

## Lỗi 4: Dùng `toBe` để so sánh object hoặc array

Không nên:

```ts id="l3lsm1"
expect(result).toBe(mockBooks);
```

Nên dùng:

```ts id="nzmo3b"
expect(result).toEqual(mockBooks);
```

Với object/array, `toEqual()` thường phù hợp hơn.

---

## Lỗi 5: Test bị ảnh hưởng bởi mock call cũ

Nếu không clear mock, test sau có thể thấy mock đã từng được gọi từ test trước.

Nên thêm trong `beforeEach`:

```ts id="malv2q"
jest.clearAllMocks();
```

---

# 21. Có cần test exception trong Controller không?

Có thể, nhưng không phải trọng tâm chính của Bài 7.

Vì trong thiết kế hiện tại, Controller chỉ gọi Service.

Nếu Service throw lỗi, Controller thường không bắt lỗi mà để NestJS xử lý.

Ví dụ Service throw:

```ts id="am2tto"
throw new NotFoundException('Book not found');
```

Controller không cần catch:

```ts id="h8v4dg"
@Get(':id')
findOne(@Param('id', ParseIntPipe) id: number): Book {
  return this.booksService.findOne(id);
}
```

Trong unit test Controller, ta có thể test rằng lỗi được truyền ra ngoài:

```ts id="qrke7n"
it('should throw error from service when book does not exist', () => {
  service.findOne.mockImplementation(() => {
    throw new Error('Book not found');
  });

  expect(() => controller.findOne(999)).toThrow('Book not found');
});
```

Nhưng case này không bắt buộc ở bài cơ bản.

Những lỗi business như `NotFoundException` nên test kỹ trong Service test.

Những lỗi HTTP thật như status code `404`, `400` nên test kỹ hơn ở E2E test.

---

# 22. Checklist hoàn thành Bài 7

Bạn đạt yêu cầu Bài 7 nếu làm được:

```txt id="z98bid"
[ ] Hiểu Controller test khác Service test
[ ] Hiểu mock Service là gì
[ ] Biết dùng jest.fn()
[ ] Biết dùng mockReturnValue()
[ ] Biết dùng toHaveBeenCalled()
[ ] Biết dùng toHaveBeenCalledWith()
[ ] Viết được test cho BooksController.findAll()
[ ] Viết được test cho BooksController.findOne()
[ ] Viết được test cho BooksController.create()
[ ] Viết được test cho BooksController.update()
[ ] Viết được test cho BooksController.remove()
[ ] Controller test không dùng BooksService thật
[ ] Tất cả test pass khi chạy npm run test
```

---

# 23. Bài tập thực hành cuối buổi

Lưu ý: Bài tập dưới đây dùng module khác với ví dụ chính trong bài để bạn tự luyện lại cách mock Service.

## Bài tập 1: Viết unit test cho AuthorsController

Giả sử bạn đã có:

```txt id="faqfbn"
AuthorsController
AuthorsService
```

Hãy viết file:

```txt id="a31e7r"
src/authors/authors.controller.spec.ts
```

Mock `AuthorsService` gồm các method:

```ts id="kg7veq"
const mockAuthorsService = {
  findAll: jest.fn(),
  findOne: jest.fn(),
  create: jest.fn(),
  update: jest.fn(),
  remove: jest.fn(),
};
```

Test các controller method:

```txt id="xu71uo"
findAll()
findOne()
create()
update()
remove()
```

Mock data:

```ts id="lmz6h9"
const mockAuthor = {
  id: 1,
  name: 'Nguyễn Nhật Ánh',
  country: 'Vietnam',
  birthYear: 1955,
};

const mockAuthors = [
  mockAuthor,
  {
    id: 2,
    name: 'J. K. Rowling',
    country: 'United Kingdom',
    birthYear: 1965,
  },
];
```

Yêu cầu:

```txt id="ud9bdd"
findAll() phải gọi authorsService.findAll()
findOne(1) phải gọi authorsService.findOne(1)
create(body) phải gọi authorsService.create(body)
update(1, body) phải gọi authorsService.update(1, body)
remove(1) phải gọi authorsService.remove(1)
```

---

## Bài tập 2: Viết unit test cho CategoriesController

Giả sử bạn đã có:

```txt id="spie8x"
CategoriesController
CategoriesService
```

Viết file:

```txt id="a85izt"
src/categories/categories.controller.spec.ts
```

Mock service:

```ts id="dkgjo2"
const mockCategoriesService = {
  findAll: jest.fn(),
  findOne: jest.fn(),
  create: jest.fn(),
  update: jest.fn(),
  remove: jest.fn(),
};
```

Mock data:

```ts id="ge2vaq"
const mockCategory = {
  id: 1,
  name: 'Programming',
  description: 'Books about software development',
};

const mockCategories = [
  mockCategory,
  {
    id: 2,
    name: 'Novel',
    description: 'Fiction and literature books',
  },
];
```

Yêu cầu test:

```txt id="k5g8i8"
Controller được defined
findAll() trả về mockCategories
findOne(1) trả về mockCategory
create() trả về category mới
update() trả về category đã cập nhật
remove() trả về category đã xóa
```

---

## Bài tập 3: Nâng cao 1 - Kiểm tra số lần gọi Service

Với `AuthorsController.findAll()`, hãy kiểm tra Service chỉ được gọi đúng 1 lần.

Gợi ý:

```ts id="w6u9mi"
expect(service.findAll).toHaveBeenCalledTimes(1);
```

Yêu cầu:

```txt id="sf2eqs"
Gọi controller.findAll()
Kiểm tra authorsService.findAll được gọi 1 lần
Không gọi thừa service khác
```

Có thể kiểm tra thêm:

```ts id="wlv6r7"
expect(service.findOne).not.toHaveBeenCalled();
expect(service.create).not.toHaveBeenCalled();
```

---

## Bài tập 4: Nâng cao 2 - Test Controller truyền đúng body

Với `CategoriesController.create()`, tạo body:

```ts id="v9o3g6"
const createCategoryDto = {
  name: 'Science',
  description: 'Books about science',
};
```

Yêu cầu test:

```txt id="zodxb0"
Controller gọi categoriesService.create(createCategoryDto)
Controller trả về createdCategory
Body truyền vào service phải đúng object ban đầu
```

Gợi ý:

```ts id="cjyvx5"
expect(service.create).toHaveBeenCalledWith(createCategoryDto);
```

---

## Bài tập 5: Nâng cao 3 - Test lỗi từ Service được truyền ra ngoài

Giả sử `AuthorsService.findOne(999)` throw lỗi.

Mock như sau:

```ts id="vzseog"
service.findOne.mockImplementation(() => {
  throw new Error('Author not found');
});
```

Yêu cầu test:

```ts id="mcp8r3"
expect(() => controller.findOne(999)).toThrow('Author not found');
```

Mục tiêu:

```txt id="l07dz4"
Hiểu rằng Controller không cần tự catch lỗi nếu lỗi đến từ Service.
```

---

## Bài tập 6: Checklist tự kiểm tra

Sau khi hoàn thành, tự kiểm tra:

```txt id="nj72ig"
[ ] AuthorsController test không dùng AuthorsService thật
[ ] CategoriesController test không dùng CategoriesService thật
[ ] Biết dùng useValue để provide mock service
[ ] Biết dùng jest.fn()
[ ] Biết dùng mockReturnValue()
[ ] Biết dùng mockImplementation()
[ ] Biết kiểm tra service được gọi bằng toHaveBeenCalled()
[ ] Biết kiểm tra tham số bằng toHaveBeenCalledWith()
[ ] Biết kiểm tra số lần gọi bằng toHaveBeenCalledTimes()
[ ] Tất cả controller test đều pass
```

---

# 24. Mini quiz kiểm tra hiểu bài

## Câu 1

Khi unit test Controller, ta nên dùng Service thật hay mock Service?

A. Dùng Service thật
B. Dùng mock Service
C. Không cần Service
D. Dùng database thật

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Dùng mock Service**

Giải thích: Controller test nên cô lập Controller, không phụ thuộc logic thật của Service.

</details>

---

## Câu 2

`jest.fn()` dùng để làm gì?

A. Tạo module mới
B. Tạo function giả để test
C. Chạy server
D. Validate request body

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Tạo function giả để test**

Giải thích: `jest.fn()` tạo mock function để kiểm tra function có được gọi không, gọi với tham số nào, và trả về gì.

</details>

---

## Câu 3

`mockReturnValue()` dùng để làm gì?

A. Cài package test
B. Cho mock function trả về giá trị giả lập
C. Xóa database
D. Tạo controller mới

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Cho mock function trả về giá trị giả lập**

Giải thích: Ví dụ `service.findAll.mockReturnValue(mockBooks)` nghĩa là khi gọi `findAll()` sẽ trả về `mockBooks`.

</details>

---

## Câu 4

Matcher nào dùng để kiểm tra function đã được gọi?

A. `toEqual()`
B. `toHaveBeenCalled()`
C. `toHaveLength()`
D. `toBeNull()`

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. `toHaveBeenCalled()`**

Giải thích: Matcher này kiểm tra mock function đã được gọi ít nhất một lần.

</details>

---

## Câu 5

Matcher nào dùng để kiểm tra function được gọi với đúng tham số?

A. `toHaveBeenCalledWith()`
B. `toHaveLength()`
C. `toBeDefined()`
D. `toContain()`

<details>
<summary>Show answer</summary>

Đáp án đúng: **A. `toHaveBeenCalledWith()`**

Giải thích: Ví dụ `expect(service.findOne).toHaveBeenCalledWith(1)` kiểm tra service được gọi với tham số `1`.

</details>

---

## Câu 6

Trong TestingModule, `useValue` dùng để làm gì?

A. Dùng giá trị mock thay cho provider thật
B. Tạo database mới
C. Chạy e2e test
D. Cấu hình port server

<details>
<summary>Show answer</summary>

Đáp án đúng: **A. Dùng giá trị mock thay cho provider thật**

Giải thích: `useValue: mockBooksService` giúp thay `BooksService` thật bằng object mock.

</details>

---

## Câu 7

Controller nên chứa logic phức tạp không?

A. Có
B. Không

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Không**

Giải thích: Controller nên mỏng, chỉ nhận request và gọi Service. Business logic nên nằm trong Service.

</details>

---

## Câu 8

Khi test `BooksController.create()`, ta chủ yếu kiểm tra điều gì?

A. Database có lưu dữ liệu không
B. Controller có gọi `booksService.create(body)` và trả đúng kết quả không
C. Server có chạy port 3000 không
D. DTO có validate không

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Controller có gọi `booksService.create(body)` và trả đúng kết quả không**

Giải thích: Unit test Controller không test database hoặc HTTP thật.

</details>

---

## Câu 9

Vì sao cần `jest.clearAllMocks()` trong `beforeEach`?

A. Để xóa database
B. Để xóa lịch sử gọi mock trước mỗi test
C. Để build project
D. Để tắt server

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Để xóa lịch sử gọi mock trước mỗi test**

Giải thích: Nếu không clear mock, số lần gọi từ test trước có thể ảnh hưởng test sau.

</details>

---

## Câu 10

Unit test Controller có gọi HTTP thật không?

A. Có
B. Không

<details>
<summary>Show answer</summary>

Đáp án đúng: **B. Không**

Giải thích: Unit test Controller gọi trực tiếp method của controller, không gửi HTTP request thật.

</details>

---

# 25. Ghi nhớ sau Bài 7

Sau bài này, cần nhớ 7 ý chính:

```txt id="jwp38t"
1. Controller test nên mock Service.
2. jest.fn() dùng để tạo mock function.
3. mockReturnValue() dùng để giả lập kết quả trả về.
4. toHaveBeenCalled() kiểm tra function có được gọi không.
5. toHaveBeenCalledWith() kiểm tra function được gọi với tham số nào.
6. useValue dùng để inject mock provider trong TestingModule.
7. Controller test không gọi HTTP thật và không dùng database thật.
```

---

# 26. Chuẩn bị cho Bài 8

Sau Bài 7, bạn đã biết:

```txt id="yqcc7n"
Unit test Service
Unit test Controller
Mock Service
Kiểm tra function được gọi đúng tham số
```

Ở Bài 8, ta sẽ chuyển sang E2E test.

Nội dung chính:

```txt id="had83o"
E2E test là gì
Supertest là gì
Test API thật qua HTTP request
Kiểm tra status code
Kiểm tra response body
```

API sẽ được test thật:

```txt id="uvhb7h"
GET    /books
POST   /books
GET    /books/:id
PATCH  /books/:id
DELETE /books/:id
```

Khác với unit test:

```txt id="i3fsbe"
Unit test gọi controller/service method trực tiếp.
E2E test gửi HTTP request thật vào app test.
```
