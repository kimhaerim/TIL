# 의존성 주입 = DI(Dependency Injection)

- 하나의 객체가 다른 객체의 의존성을 제공하는 테크닉
- 클래스 간의 결합도를 낮추기 위해 필요한 객체를 직접 생성하지 않고 주입받는 패턴
- Nest의 경우, IoC 컨테이너를 통해 인스턴스를 자동으로 주입
- @Injectable() 데코레이터와 constructor() 기반으로 DI가 이루어짐
  -> NestJS의 DI는 IoC 컨테이너를 기반으로 의존성 주입을 자동화하여 구조화된 모듈 설계를 가능하게 하고, 확장성과 유지보수성을 크게 높여줌

## DI 작동

- Nest는 내부적으로 Reflect Metadata를 활용해 의존성 그래프를 구성
- 모듈의 providers에 등록된 클래스 인스턴스를 생성 및 공유

## @Injectable() 데코레이터의 역할

- 해당 클래스가 Nest IoC 컨테이너에 의해 관리되도록 함
- 다른 클래스의 생성자에 주입될 수 있도록 설정

# Node와 Nest 의존성 주입

## ex) "홍길동"이 최근 한 달 동안 작성한 리뷰 목록을 조회하는 경우

### NodeJS

```js
// reviewRepository.js
class ReviewRepository {
  constructor(db) {
    this.db = db;
  }

  async getReviews(userId) {
    const oneMonthAgo = new Date();
    oneMonthAgo.setMonth(oneMonthAgo.getMonth() - 1);

    return this.db.query(
      `
            SELECT * FROM review WHERE createdAt >= ? AND userId = ?;
        `,
      [oneMonthAgo, userId]
    );
  }
}

module.exports = ReviewRepository;
```

```js
// userService.js
class UserService {
  constructor(reviewRepository) {
    this.reviewRepository = reviewRepository;
  }

  async getRecentlyReviews(userId) {
    return await this.reviewRepository.getReviews(userId);
  }
}

module.exports = UserService;
```

```js
// app.js
const mysql = require("mysql2/promise");
const ReviewRepository = require("./reviewRepository");
const UserService = require("./userService");

(async () => {
  const db = await mysql.createConnection({
    host: "localhost",
    user: "root",
    password: "",
    database: "app_db",
  });

  const reviewRepository = new ReviewRepository(db);
  // userService에서 reviewRepository를 사용할 수 있도록 의존성을 생성하고 주입하는 부분
  const userService = new UserService(reviewRepository);

  const result = await userService.getRecentlyReviews();
  console.log(result);
})();
```

- DI 없이 직접 객체를 생성해서 전달함
- 작고 단순한 프로젝트에서는 유용함
- 의존성이 많아질수록 테스트, 변경, 유지보수가 어려워짐

### NestJS

- Nest의 IoC 컨테이너가 의존성 트리를 자동으로 생성해줌

```ts
// user.service.ts
@Injectable()
export class UserService {
  constructor(private readonly reviewRepository: ReviewRepository) {}
}
```

```ts
// user.module.ts
@Module({
  providers: [UserService],
  imports: [ReviewModule], // 여기서 ReviewRepository가 export 되어 있어야 함
})
export class UserModule {}
```

```ts
// review.module.ts
@Module({
  providers: [ReviewRepository],
  exports: [ReviewRepository], // 다른 모듈에서 쓸 수 있도록 export
})
export class ReviewModule {}
```

- reviewRepository인스턴스를 자동으로 생성해서 UserService에 주입함
- 모듈 기반이라 변경에 유연함
