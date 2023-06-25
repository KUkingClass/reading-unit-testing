# 3장. 단위 테스트 구조

```
- 단위 테스트 구조: AAA패턴; 준비 Arrange, 실행 Act, 검증 Assert
- 좋은 단위 테스트 명명법
- 단위 테스트 간소화를 위한 팁
```

## AAA패턴

테스트를 준비, 실행, 검증 세 부분으로 나눠서 작성하는 것

- 준비 구절: 테스트 대상 시스템(SUT)이나 해당 의존성을 원하는 상태로 만듦
- 실행 구절: SUT에서 메서드를 호출하면서 준비된 의존성 전달 / 출력값 캡처
- 검증 구절: 결과 검증. 이 때 결과는 반환값 or SUT과 협력자의 최종 상태 or SUT가 협력자에 호출한 메서드 등

### Given-When-Then 패턴

```java
class CalculatorTest {

    @Test
    void sum_of_two_numbers() {
      	// 준비
        // given
        double first = 10;
        double second = 20;
        Calculator calculator = new Calculator();

      	// 실행
        // when
        double result = calculator.sum(first, second);

      	// 검증
        // then
        assertThat(result).isEqualTo(30);
    }
}
```

- AAA패턴과 같은 구성
- 차이점은 프로그래머가 아닌 사람에게 Given-When-Then 구조가 더 읽기 쉬움. 따라서 비기술자들과 공유하는 테스트의 경우 더 적합

### 지켜야할 것

- 여러개의 준비, 실행, 검증 구절 피하기. 각각은 한번만
  - 여러개가 있다는 건 이미 너무 많은 것을 한 번에 검증한다는 것.
  - 하지만 이미 ‘느린 통합 테스트’일 때 여러 실행과 검증이 있는 단일한 테스트로 묶을 수도 있음
- 테스트 내 if문 피하기: if 문 자체가 테스트가 한 번에 너무 많은 것을 검증한다는 증거

### 각 구절의 크기

1. 일반적으로 준비 구절이 가장 큼
    - 실행과 검증을 합친 만큼 커질 수 있음
    - 하지만 이것보다 훨씬 크면 같은 테스트 클래스 내 비공개 메서드 또는 별도의 팩토리 클래스로 도출하는 것이 좋다
      - 제품 코드에서도 생성자 복잡할 때 빌더 패턴 이용하는 것처럼...
      - 이 때 두가지 패턴으로 오브젝트 마더/테스트 데이터 빌더 → 10.4.1절
        
2. 실행 구절은 코드 한 줄
    - **두 줄 이상인 경우? SUT의 공개 API에 문제가 있을 수 있음**
      - 예를 들어 어떤 동작이 끝나려면 무조건 여러 메소드를 호출해야한다는 걸 클라이언트가 알고 있어야 한다는 것과 같음
      - 클라이언트에게 메서드 호출 강요하지말 것!
      - 다음 메서드를 호출하지 않으면 문제 발생 → 불변 위반(invariant violation) | 코드 캡슐화를 지킬 것
    - 다만 비즈니스 로직이 아닌 유틸리티나 인프라 코드는 덜 적용됨
      
3. 검증 구절은 하나의 검증??
    - **흔히 많이 듣는 검증 구절에 하나의 assert는 올바르지 않다!** **단위 테스트의 단위는 동작의 단위이지 코드의 단위가 아니다.**
    - 단일 동작 단위는 여러 결과를 낼 수 있으며, 하나의 테스트로 모든 결과를 평가하는 것이 좋음
    - 하지만 그래도 너무 커지는 건 경계할 것. 코드에 추상화가 누락된것일 수 있음
      - 예) SUT에서 반환된 객체 내의 모든 속성 검증 대신 객체 클래스에 동등 멤버(equality member)를 정의하는 것이 좋음.

### 테스트 대상 시스템 구별하기

- 동작이 여러 클래스에 걸쳐 있을 수도 있지만 진입점은 오직 하나
- 따라서 SUT(진입점)를 의존성과 구분 → 아예 이름을 sut로 지어라 ❓

### 각 구절마다 주석으로 구분을 해야할까?

- 일반적인 경우라면 주석 없이 빈 줄로 준비, 실행, 검증 구절 구분 가능
- 하지만 대규모 테스트의 경우 준비 단계 안에서도 설정을 구분하는 등의 경우가 발생 가능
  - 따라서 준비 및 검증 구절 안에 빈 줄을 추가해야 하는 테스트라면 주석 유지할 것

## 테스트 픽스처 재사용법

```
테스트 픽스처
- 테스트 '실행' 대상 객체. 정규 의존성. 
- 즉 SUT로 전달되는 인수. 
- 각 테스트 실행 전에 고정 상태로 유지하기 때문에 동일한 결과를 생성 -> 그래서 픽스
```

테스트 픽스처를 준비하기 위해 많은 코드 필요. 따라서 별도의 메서드나 클래스로 도출한 후 테스트 간에 재사용하는 것이 좋음.

그럼 어떻게 하면 잘 재사용할 수 있을까?

### 사전 셋업 구문 X: `@BeforeEach`

- 여러 테스트에서 공통 부분이 있다고 사전 셋업 구문에 추가하지 말자.

  ```java
  class AdderTest {
      private Adder adder;
  
      @BeforeEach
      void setup() {
          adder = new Adder(0);
      }
  
      @Test
      void add_number() {
          // given
  
          // when
          int result = adder.add(5);
  
          // then
          assertThat(result).isEqualTo(5);
      }
  
  }
  ```

  - 테스트 간 결합도 높아짐

  - 테스트 가독성 떨어짐

- 다만 예외로 대부분의 테스트에 사용되는 경우 가능

  - 예) 데이터베이스 연결 코드
  - 하지만 그 때도 base class에서 설정하고 그걸 상속하는 게 더 합리적

### 테스트 클래스에 private 팩토리 메서드 두기

```java
@Test
void add_number() {
    // given
    Adder adder = createAdderWithBase(0);

    // when
    int result = adder.add(5);

    // then
    assertThat(result).isEqualTo(5);
}

private Adder createAdderWithBase(int base) {
    return new Adder(base);
}
```

## 단위 테스트 명명법

규칙을 따르지 말 것.

- 예를 들어 [테스트 대상 메서드_시나리오_예상결과] 같은 정해진 이름 규칙

복잡한 동작에 대한 높은 수준의 설명을 이런 규칙에 넣을 수 없음. **비개발자에게 시나리오를 설명하는 것처럼 테스트 이름을 짓자.**

```java
// 예시: 과거 배송일이 유효하지 않은 걸 검증하는 테스트
// 하나씩 개선해보자.
Delivery_with_invalid_date_should_be_considered_invalid

// invalid가 정확히 어떤 경우? 여기선 과거 날짜
// considered를 제거해도 의미 동일
Delivery_with_past_date_should_be_invalid

// should be는 안티 패턴. 
// 하나의 테스트는 동작 단위에 대해 단순하고 원자적인 '사실'
// 사실을 서술할 때는 소망이나 욕구 X
Delivery_with_past_date_is_invalid

// 영문법 지키기
Delivery_with_a_past_date_is_invalid
```

- 참고

  테스트 클래스 이름 지정시 [클래스명]Test를 쓰긴 하지만, 그 클래스만 검증한다는 게 아니라, 어딘가에서 시작해야 하므로 진입점, API 정도를 나타내는 것.

## 매개변수화된 테스트: parameterized test

동작을 설명하기에 한 테스트는 보통 충분하지 않다. 이를 위해 단위 테스트 프레임워크에선 매개변수화된 테스트를 제공

예를 들어 날짜를 테스트한다면, 성공/실패 케이스 중에서도 성공하는 여러 날짜 종류와 실패하는 여러 날짜 종류가 있을 수 있다.

```java
// 가장 빠른 배송일이 오늘부터 이틀 후가 되도록 작동하는 배송 기능이 있다고 하자.
// 따라서 여러 날짜를 테스트해야한다.
// 이 때 날짜마다 함수를 따로 만들지 않고 테스트를 하나로 묶어 코드의 양을 줄일 수 있다.
// 그럴 때 날짜와 성공여부 결과 bool 변수를 매개변수로 넘겨서 하나의 테스트로 해결할 수 있다.
class DeliveryServiceTest {

    @ParameterizedTest
    @MethodSource("provideDaysForExpected")
    void can_detect_an_invalid_delivery_date(int daysFromNow, boolean expected) {
        // given
        DeliveryService deliveryService = new DeliveryService();
        LocalDate deliveryDate = LocalDate.now().plusDays(daysFromNow);
        Delivery delivery = new Delivery(deliveryDate);

        // when
        boolean isValid = deliveryService.isDeliveryValid(delivery);

        // then
        assertThat(isValid).isEqualTo(expected);
    }

    private static Stream<Arguments> provideDaysForExpected() {
        return Stream.of(
                Arguments.of(-1, false),
                Arguments.of(0, false),
                Arguments.of(1, false),
                Arguments.of(2, true)
        );
    }

}
```

**하지만 이제 테스트 메서드가 나타내는 사실을 파악하기가 어려워진다.**

따라서 절충안으로 **긍정적인 테스트 케이스는 고유한 테스트로 도출**하고, 가장 중요한 부분을 잘 설명하는 이름을 쓰면 좋다. 그리고 부정적인 테스트를 여러 매개변수로 넘길 수 있다.

```java
@ParameterizedTest
@ValueSource(ints = {-1, 0, 1})
void detects_an_invalid_delivery_date(int daysFromNow) {
    // given
    DeliveryService deliveryService = new DeliveryService();
    LocalDate deliveryDate = LocalDate.now().plusDays(daysFromNow);
    Delivery delivery = new Delivery(deliveryDate);

    // when
    boolean isValid = deliveryService.isDeliveryValid(delivery);

    // then
    assertThat(isValid).isFalse();
}

@Test
void the_soonest_delivery_date_is_two_days_from_now() {
    // given
    DeliveryService deliveryService = new DeliveryService();
    LocalDate deliveryDate = LocalDate.now().plusDays(2);
    Delivery delivery = new Delivery(deliveryDate);

    // when
    boolean isValid = deliveryService.isDeliveryValid(delivery);

    // then
    assertThat(isValid).isTrue();
}
```

하지만 동작이 너무 복잡하면 사용하지 말고 모두 각각 고유의 테스트 메서드로 나타내자.
