# [UnitTesting] 10장. 데이터베이스 테스트

# Testing the database: Overview

- 통합 테스트의 마지막 퍼즐 조각은 외부 프로세스 dependency를 관리하는 것
    - ex. 애플리케이션 데이터베이스
- 실제로 데이터베이스를 테스트를 하면 회귀 방지가 아주 뛰어나지만, 유지 보수가 쉽지 않다.

데이터베이스 테스트를 위해 어디까지 해야하는가?

# Prerequisites for testing the database

- 테스트 전제 조건
    - 데이터베이스를 소스 제어 시스템에 유지
    - 모든 개발자에 대해 별도의 데이터베이스 인스턴스 사용
    - 데이터베이스 전달에 migration-based 접근 방식 적용
    

## 데이터베이스를 소스 제어 시스템에 유지하라

- 데이터베이스 테스트의 첫 단계는 데이터베이스 스키마를 일반 코드처럼 취급하는 것
    - 스키마도 Git과 같은 소스 제어 시스템에 저장되어야 한다.
- 개발하는 동안 Model DB 인스턴스 → `develop` 처럼 최신화, Production DB → `main(master)`
- Model DB와 Production DB를 비교하고 업그레이드 스크립트를 생성 후 production 환경에서 해당 스크립트를 실행한다.

![Untitled](%5BUnitTesting%5D%2010%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%90%E1%85%A6%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%205f450b19d69944949b77c9d5021dd8f9/Untitled.png)

- Model DB를 바로 사용하는 것은 데이터베이스 스키마를 유지/보수하는데 끔찍한 방법이다.
    - 변경 내역 추적 불가 (운영 환경에서 버그 재생성 가능성)
    - single source of truth를 지킬수 없다.
- 소스 제어 시스템에서 DB 스키마를 업데이트하면 단일 진리 소스를 유지할 수 있고, 일반 코드의 변경 사항과 함께 DB 변경 사항을 추적할 수 있다.

## Reference data는 데이터베이스 스키마의 일부이다

- 참조 데이터는 응용 프로그램이 제대로 작동하기 위해 미리 채워야 하는 데이터이다.
- ex. CRM 시스템
    - UserType : Customer 또는 Employee
    - 유저에 대한 테이블 생성 (이름, 나이 등 입력 가능)

- application에서 데이터를 수정할 수 있는 경우 → `일반 데이터`
- 수정할수 없는 경우 → `참조 데이터`
    - 보통 두 데이터는 별도 저장되지만, 동일한 테이블에 공존하는 경우도 있음
        - 수정가능한 데이터(regular data), 수정할 수 없는 데이터(reference data)를 구별하는 플래그를 도입
- 참조 데이터는 필수적인 데이터이므로 SQL INSERT문 형식으로 스키마의 테이블, view 등과 함께 소스 제어 시스템에 보관해야 한다.

## 모든 개발자를 위한 별도의 인스턴스

- 다른 개발자와 데이터베이스를 공유한다면 테스트는 더욱 어려워진다.
    - 서로 다른 개발자가 실행하는 테스트는 서로 간섭할 것
    - 이전 버전과 호환되지 않는 변경은 다른 개발자의 작업을 방해할 것
- 모든 개발자에 대해 별도의 데이터베이스 인스턴스를 유지해야 한다.

## State-based vs Migration-based Database

- State-based
    - 위의 그림과 다른 점은 실제 소스로써 물리적인 모델 데이터베이스를 가지고 있지 않고, DB를 만드는 데 사용할 수 있는 SQL 스크립트가 존재 → 이 스크립트가 저장되는 것
    - comparison tool이 데이터베이스 최신화 상태를 가져오기 위한 모든 것을 수행
        - 테이블 삭제, 새 테이블 생성, column명 변경 등

- Migration-based
    - 데이터베이스를 한 버전에서 다른 버전으로 전환하는 명시적인 마이그레이션 사용
        - tool을 사용하여 동기화 하는 것이 아니라 직접 업그레이드 스크립트를 생성한다. (tool은 스키마에서 문서화 되지 않은 변경 사항 탐지에 용이)

![Untitled](%5BUnitTesting%5D%2010%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%90%E1%85%A6%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%205f450b19d69944949b77c9d5021dd8f9/Untitled%201.png)

- State가 아닌 Migration이 SQL 스크립트로 표시되어 git에 저장된다.
    - Flyway, Liquibase
- ex. FluentMigrator를 사용하여 마이그레이션

![Untitled](%5BUnitTesting%5D%2010%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%90%E1%85%A6%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%205f450b19d69944949b77c9d5021dd8f9/Untitled%202.png)

- `state-based`는 (git에 해당 상태를 저장함으로써) state를 명시적으로 만들고, comparison tool이 암묵적으로 마이그레이션을 제어
- `migartion-based`는 마이그레이션을 명시적으로 만들지만 state는 암시적으로 유지.
    - 데이터베이스 상태를 직접 보는 것은 불가능하므로 마이그레이션에서 구성해야함

![Untitled](%5BUnitTesting%5D%2010%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%90%E1%85%A6%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%205f450b19d69944949b77c9d5021dd8f9/Untitled%203.png)

- DB의 상태는 merge conflicts를 쉽게 처리 vs 마이그레이션은 데이터 이동을 쉽게 처리
    - 마이그레이션을 통해 스키마에 모든 수정 사항을 적용하라
    - 깃에 커밋 된 마이그레이션을 수정하지 마라
    - 마이그레이션이 잘못된 경우 이전 것을 수정하지 말고 새 마이그레이션을 생성해라
    - 잘못됐을 때 데이터 손실이 발생하는 경우만 예외를 둬라

---

# Test data life cycle

- Shared DB는 통합 테스트를 서로 분리해야하는 문제가 존재
    - 모든 통합 테스트를 순차적으로 실행하라
    - 테스트 실행 전에는 남은 데이터를 제거하라
- 테스트는 데이터베이스의 상태에 의존해서는 안된다.

## Parallel vs Sequential test execution

- 병렬로 테스트를 수행하면 테스트 수행 시간이 단축된다. But, 유지 보수 비용이 크다.
- 시간이 조금 더 걸리더라도 순차적으로 데이터베이스를 테스트 하는 것이 좋다.

## 테스트 사이의 데이터 삭제(정리)

- 권장하는 방법은 테스트를 시작하는 시점에 모든 데이터를 삭제하고 시작하는 것.
    - 데이터를 삭제하는 것은 공통 클래드 & 메서드로 추출한 다음에 `@BeforeEach()` 등으로 매 테스트마다 실행되도록 할 수 있다.
    

이 외에 테스트 사이에 데이터를 정리하는 방법은 다음과 같다.

- 각 테스트 전에 데이터베이스 백업 복원하기 (Restoring)
    - DB를 백업하고 복원하기 때문에 수행 시간이 길어진다.
- 테스트 종료 시점에 데이터 정리하기 (Cleaning End)
    - 빠르지만 정리 단계를 까먹을 수도 있다. (ex. 디버거에서 테스트를 종료해버릴때)
- 데이터베이스 트랜잭션에 각 테스트를 래핑한 뒤로는 커밋하지 않기 (Wrapping)
    - 추가 트랜잭션을 사용하기 때문에 운영 환경과 다른 설정이 생성된다.
- **테스트 시작 시점에 데이터 정리하기** (Cleaning Beginning) ✨
    - 빠르게 작동하고 일관성있게 동작하며, 정리 단계를 실수로 건너뛰지 않는다

## 인메모리 데이터 베이스는 피하기

- 테스트에서 데이터베이스 간 격리를 위해서 `SQLite`, `H2` 같은 인메모리 데이터베이스를 사용할 수도 있다.
    - But, 인메모리 데이터베이스를 사용하는 것은 지양해야 함.
- 장점
    - 테스트 데이터를 제거할 필요가 없음.
    - 작업 속도 향상
    - 테스트가 실행될 때 마다 인스턴스화 가능
- 단점
    - 실제 사용하는 데이터베이스와 기능적으로 일관성이 없음.
    - 테스트 환경과 실제 프로덕션 환경 사이의 불일치

- 프로덕션과 동일한 DBMS를 테스트에 사용하자. (DB 공급 업체는 동일하게 유지해야 한다.)

---

# Reusing code in test sections

- 통합 테스트는 가능한 짧게 유지하고, 가독성에 영향을 주지 않는 것이 중요하다.
    - 매우 짧은 테스트라도 서로 의존해서는 안됨.
    - 단축하는 방법 → 비즈니스 로직과 관련이 없는 부분을 **private, helper 메서드나 클래스로 추출하고 재사용 한다.**

## Arrange sections (Given)

- Ex)

```java
// [Fact]
public void Changing_email_from_corporate_to_non_corporate() {

    // Arrange
		User user;

		using (var context = new CrmContext(ConnectionString)) {
			var userRepository = new UserRepository(context);
			var companyRepository = new CompanyRepository(context);
			user = new User(0, "user@mycorp.com",
			UserType.Employee, false);
			userRepository.SaveUser(user);

			var company = new Company("mycorp.com", 1);
			companyRepository.SaveCompany(company);
			context.SaveChanges();
		}

		var busSpy = new BusSpy();
		var messageBus = new MessageBus(busSpy);
		var loggerMock = new Mock<IDomainLogger>();

		string result;
		using (var context = new CrmContext(ConnectionString)) {
			var sut = new UserController(
				context, 
				messageBus, 
				loggerMock.Object
			);

			// Act
			result = sut.ChangeEmail(user.UserId, "new@gmail.com");
		}

		// Assert
		Assert.Equal("OK", result);
		using (var context = new CrmContext(ConnectionString)) {
			var userRepository = new UserRepository(context);
			var companyRepository = new CompanyRepository(context);

			User userFromDb = userRepository.GetUserById(user.UserId);
			Assert.Equal("new@gmail.com", userFromDb.Email);
			Assert.Equal(UserType.Customer, userFromDb.Type);

			Company companyFromDb = companyRepository.GetCompany();
			Assert.Equal(0, companyFromDb.NumberOfEmployees);

			busSpy.ShouldSendNumberOfMessages(1)
				.WithEmailChangedMessage(user.UserId, "new@gmail.com");

			loggerMock.Verify(
					x => x.UserTypeHasChanged(
						user.UserId, UserType.Employee, UserType.Customer
					),
					Times.Once
			);
		}
}
```

- 테스트의 given 단계에서 코드를 재사용하는 가장 좋은 방법은 `private factory methods`를 도입하는 것
    - ex. User를 만들 때

```java
private User CreateUser(
		string email, 
		UserType type, 
		bool isEmailConfirmed
) {
		using (var context = new CrmContext(ConnectionString)) {
			var user = new User(0, email, type, isEmailConfirmed);
			var repository = new UserRepository(context);

			repository.SaveUser(user);

			context.SaveChanges();

			return user;
		}
}
```

- argument에 대한 기본값을 정의한다면?
    - 기본값을 사용하면 테스트를 더욱 단축할 수 있다.
    - argument를 선택적으로 사용하면 어떤 것이 테스트 시나리오와 관련있는지 파악하기 쉽다.

```java
private User CreateUser(
	string email = "user@mycorp.com",
	UserType type = UserType.Employee,
	bool isEmailConfirmed = false
) {
	/* ... */
}
```

- factory 메서드 사용

```java
User user = CreateUser(
	email: "user@mycorp.com",
	type: UserType.Employee
);
```

- Object Mother vs Test Data Builder
    - 위에서 CreateUser를 통해 객체를 만드는 패턴 → Object Mother
        - 테스트 고정장치(테스트가 실행되는 객체) 를 만드는데 도움이 되는 클래스 또는 메서드
    - 또다른 패턴 → Test Data Builder
        - 일반 메소드 대신 인터페이스를 제공
        - 가독성은 올라가지만 보일러 플레이트 양산
        - Object Mother를 고수하는 것이 좋다.

```java
User user = new UserBuilder()
	.WithEmail("user@mycorp.com")
	.WithType(UserType.Employee)
	.Build();
```

## Act sections (When)

```java
string result;
using (var context = new CrmContext(ConnectionString)) {
		var sut = new UserController(
			context, 
			messageBus, 
			loggerMock.Object
		);

		// Act
		result = sut.ChangeEmail(user.UserId, "new@gmail.com");
}
```

- 어떤 컨트롤러 기능이 호출해야 하는지 → Delegate (대리인에게 위임)

```java
private string Execute(
	Func<UserController, string> func, // Delegate : controller 기능 정의
	MessageBus messageBus,
	IDomainLogger logger
) {
		using (var context = new CrmContext(ConnectionString)) {
			var controller = new UserController(
				context,
				messageBus,
				logger
			);

			return func(controller);
		}
}

// 데코레이터 메서드를 사용해서 단축 가능
string result = Execute(
	x => x.ChangeEmail(user.UserId, "new@gmail.com"),
	messageBus, loggerMock.Object
);
```

## Assert sections (Then)

- 가장 쉬운 방법은 CreateUser 과 같이 Helper Method를 도입하는 것
    - 더 나아가 인터페이스로 확장 가능

```java
User userFromDb = QueryUser(user.UserId);
Assert.Equal("new@gmail.com", userFromDb.Email);
Assert.Equal(UserType.Customer, userFromDb.Type);

Company companyFromDb = QueryCompany();
Assert.Equal(0, companyFromDb.NumberOfEmployees);
```

```java
public static class UserExternsionz {

	public static User ShouldExist(this User user) {
		Assert.NotNull(user);
		return user;
	}

	public static User WithEmail(this User user, string email) {
		Assert.Equal(email, user.Email);
		return user;
	}
}

// 읽기 쉽게 바뀐다.
User userFromDb = QueryUser(user.UserId);

userFromDb
	.ShouldExist()
	.WithEmail("new@gmail.com")
	.WithType(UserType.Customer);

Company companyFromDb = QueryCompany();

companyFromDb
	.ShouldExist()
	.WithNumberOfEmployees(0);
```

---

# Common database testing questions

## 읽기(reads) 테스트를 해야하는가?

- 가장 복잡하거나 중요한 읽기만 테스트하고, 나머지는 읽기 테스트 하지 않는것이 좋다.

- 쓰기는 철저히 테스트 해야한다.
    - 데이터 손상 및 DB와 외부 애플리케이션에도 영향을 미칠 수 있기 때문
    - 쓰기 테스트는 회귀 방지를 제공
- 읽기는 DB에 영향을 주지 않는다.
    - 읽기에는 도메인 모델이 필요하지 않다.
    - 읽기 테스트의 경우는 실제 데이터베이스에서 바로 통합 테스트 가능

![Untitled](%5BUnitTesting%5D%2010%E1%84%8C%E1%85%A1%E1%86%BC%20%E1%84%83%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%90%E1%85%A5%E1%84%87%E1%85%A6%E1%84%8B%E1%85%B5%E1%84%89%E1%85%B3%20%E1%84%90%E1%85%A6%E1%84%89%E1%85%B3%E1%84%90%E1%85%B3%205f450b19d69944949b77c9d5021dd8f9/Untitled%204.png)

## Repository 테스트를 해야하는가?

- 레포지토리는 DB 접근에 대해 추상화한다.

```java
User user = _userRepository.GetUserById(userId);
_userRepository.SaveUser(user);
```

- 레포지토리는 **일반적인 경우에는 테스트 하지 않는 것이 좋다.**
    - 만약, 복잡한 로직 (Entity Mapping, Slicing, Paging) 등이 있다면 **이 로직들을 분리하고 해당 메서드만 테스트**
    - 비용 대비 테스트 효과가 좋지 않기 때문

---

# Summary

- 통합 테스트는 순차적으로 실행하자. (병렬 실행은 노력에 비해 효과가 적다.)
- 테스트 시작 시점에, 남아있는 데이터를 정리하자.
- 인메모리 DB를 테스트 용으로 사용하지 말자. 외부 업체의 데이터베이스로 테스트를 실행하면 보호 수준이 떨어진다.
- 테스트에서 private / helper 메서드 및 클래스를 이용해 테스트 코드를 단축할 수 있다.
    - `Given` : Object Mother, Test Builder
    - `When` : Delegate, Decorator Method
    - `Then` : Fluent Interface (Method Chaining)
- 데이터 베이스의 읽기 테스트 코드는 꼭 필요하고 복잡한 로직만 하자. (버그 가능성이 쓰기보다 현저히 낮음)
- 레포지토리는 직접 테스트 하지 말고 통합 테스트의 일부로 취급하자. (유지비용은 높으면서 회귀방지는 낮음)