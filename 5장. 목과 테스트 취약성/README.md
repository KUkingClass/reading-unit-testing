# 5장 **목과 테스트 취약성**

```
Mock이 취약한 테스트를 만드는 경우, 리팩토링 내성 저하 없이 Mock을 사용하는 방법에 대해 살펴본다.

취약한 테스트 : ‘리팩토링 내성’ 이 없는 테스트
```
### Mock과 Stub 구분
테스트 대상 시스템 (SUT)와 그 협력자 사이의 상호 작용을 검사할 수 있는 객체들을 테스트 대역이라고 한다. 
Mock과 Stub은 테스트 대역의 일종이다.  

### 테스트 대역 유형
<img width="600" alt="image" src="https://github.com/KUkingClass/reading-unit-testing/assets/63101979/d0d694b1-1b2f-42f0-a0f4-ed727a96da15">
목 (Mock)  

- 외부로 나가는 상호 작용을 모방하고 검사하는데 도움이 된다. -> SUT가 상태를 변경하기 위한 의존성을 호출

스텁 (Stub)
- 내부로 들어오는 상호 작용을 모방하는데 도움이 된다. -> SUT가 입력 데이터를 얻기 위한 의존성을 호출하는 것에 해당

<img width="600" alt="image (1)" src="https://github.com/KUkingClass/reading-unit-testing/assets/63101979/9cf7776e-a212-4e2a-99e9-39cac8c253ff">
이메일 발송은 외부로 나가는 상호작용이다. 목은 이러한 상호 작용을 모방하는 테스트 대역에 해당한다.  

데이터베이스에서 데이터를 검색하는 것은 내부로 들어오는 상호 작용이다. 해당 테스트 대역은 스텁이다.

### 스텁으로 상호 작용을 검증하지 말라
목은 관련 의존성으로 나가는 상호 작용을 검사하지만 스텁은 내부로 들어오는 상호 작용을 모방한다.   

스텁은 단순히 SUT가 최종결과를 생성하도록 입력을 제공하기 때문에 SUT에서 스텁으로의 호출은 SUT가 생성하는 최종결과가 아니다.  


테스트에서 리팩토링 내성을 향상시키는 방법은 구현 세부 사항에서 멀어지고 결과만을 검증하는 것이다.  

→ 스텁과의 상호작용은 검증할 필요 없이, 단순히 인풋 데이터를 넣어주는 형태로 사용한다.  

반면 목은 상호작용을 검증하기도 한다.   

예를 들어, 이메일을 보내는 것은 사용자가 원하는 최종 결과이기 때문에 목을 통한 외부 의존성 호출을 검증하는 것이 의미가 있다. 

## 식별할 수 있는 동작과 구현 세부 사항

### 식별할 수 있는 동작은 공개 API와는 다르다.

모든 제품 코드는 두가지로 분류할 수 있다.
- 공개 API / 비공개 API
- 식별할 수 있는 동작 / 구현 세부 사항  

각 차원의 범주는 겹치지 않는다.  

아래 동작 중 하나라도 만족한다면 ‘식별할 수 있는 동작’이다.  
아래 동작 중 하나도 만족하지 않는다면 ‘구현 세부 사항’이다.  

- 클라이언트가 목표를 달성하는데 **도움이 되는 연산**을 노출
    - 연산은 계산을 수행하거나 부작용을 가져오거나 둘다하는 메서드이다.
- 클라이언트가 목표를 달성하는데 **도움이 되는 상태**를 노출
  
<img width="600" alt="image (2)" src="https://github.com/KUkingClass/reading-unit-testing/assets/63101979/d1bffb68-40cb-418b-8284-f31069ac0e13">

잘 설계된 API라면, 공개 API는 식별할 수 있는 동작과 일치하고 비공개 API는 구현 세부 사항과 일치한다.  
잘 설계되지 못한 API라면, 공개 API를 통해 구현 세부 사항을 클라이언트에게 노출하게 된다. 이는 리팩토링 내성을 약화시킬 수 있다.

### 구현 세부사항 유출의 예

```java
// 클라이언트
public class UserController {

		// userEntity의 이름 변경
    public void renameUser(Long userId, String newName) {

        UserEntity userFromDB = getUserFromDB(userId);

        String normalizedName = userFromDB.normalizedName(newName);
        userFromDB.setName(normalizedName);

        saveUserToDB(userFromDB);
    }
}

// 테스트 대상 
public class UserEntity {
    private String Name;
    public void setName(String name) {
        Name = name;
    }

    public String normalizedName(String name) {
        return name.length() > 50 ? name.substring(0, 50) : name;
    }
}
```
UserController 클라이언트는 UserEntity의 public API normalizedName(), setName() 2가지를 사용한다.
- setName()은 새로운 이름을 저장하는 메서드  
    → 클라이언트가 기대하는 동작을 한다.
- normalizedName()은 위의 목표와 직접적인 연관은 없다  
    → 세부 구현 사항이다.
  
<img width="600" alt="image (3)" src="https://github.com/KUkingClass/reading-unit-testing/assets/63101979/6d81aa39-b23c-44af-ad24-2168ca1d22df">  

따라서 위의 코드를 살펴보면 구현 세부사항 + 식별할 수 있는 동작이 모두 Public API로 공개된 상태가 된다.

```java
public class UserRepository {
    public void renameUser(Long userId, String newName) {
        UserEntity userFromDB = getUserFromDB(userId);
        userFromDB.setName(newName);
        saveUserToDB(userFromDB);
    }
}

public class UserEntity {
    private String Name;

	// 공개 API
    public void setName(String name) {
        this.Name = normalizedName(name);
    }

	// 비공개 API
    private String normalizedName(String name) {
        return name.length() > 50 ? name.substring(0, 50) : name;
    }
}
```
- normalizedName()은 구현 세부 사항 → 비공개 API로 변경  
- setName()은 이름을 바꾸길 기대하는 클라이언트의 작업을 돕기 때문에 식별할 수 있는 동작이다. → public API로 사용  
- setName()은 구현 세부 사항인 normalizedName() 메서드를 사용한다.

단일한 목표를 달성하고자 클래스에서 호출해야하는 연산의 수가 1보다 크면 유출하고 있을 가능성이 있다. 이상적으로는 단일 연산으로 개별 목표를 달성해야 한다.

### 잘 설계된 API 캡슐화
캡슐화는 분변성 위반이라고도 하는 모순을 방지하는 조치이다.
- 구현 세부 사항을 숨기면 클라이언트의 시야에서 클래스 내부를 가릴 수 있기 때문에 내부를 손상시킬 위험이 적다.
- 데이터와 연산을 결합하면 해당 연산이 클래스의 불변성을 위반하지 않도록 할 수 있다.
    - 클래스 내부에서 데이터를 구현세부사항과 관련된 연산과 결합해서 사용한다.

## 목과 테스트 취약성 관계

### 육각형 아키텍쳐
육각형 아키텍처는 도메인 / 애플리케이션 서비스 계층으로 나뉜다.  
<img width="600" alt="image (4)" src="https://github.com/KUkingClass/reading-unit-testing/assets/63101979/3784fe9b-ec1b-467c-9756-255615f5a86e">

어플리케이션 서비스 계층은 도메인 계층과 외부 환경과의 통신을 조정한다.  

어플리케이션 서비스에 대한 조정의 예시  
- 데이터 베이스를 조회하고 해당 데이터로 도메인 클래스 인스턴스 구체화
- 해당 인스턴스에 연산 호출
- 결과를 데이터베이스에 다시 저장

### 어플리케이션 간의 통신
<img width="600" alt="image (5)" src="https://github.com/KUkingClass/reading-unit-testing/assets/63101979/f352e140-f134-41d5-a544-89b7baae2a09">

외부 어플리케이션은 도메인 계층에 직접적으로 접근할 수는 없다. 반드시 어플리케이션 서비스 계층을 통해서 도메인 계층에 접근하도록 해야한다.

<img width="800" src="https://github.com/KUkingClass/reading-unit-testing/assets/63101979/2961ad24-dcfb-44e7-9035-f98745b48722"/>  

외부 클라이언트의 목표는 개별 도메인 클래스에서 하위 목표로 변환한다.  
→ 도메인 계층에서 식별할 수 있는 동작은 각각 구체적인 비즈니스 유즈케이스와 연관성이 있다.  
좋은 테스트라면, 어떤 테스트든 비즈니스 요구사항으로 거슬러 올라갈 수 있어야 한다.   

육각형 아키텍처의 주 목적은 다음과 같다.
- 도메인 계층과 어플리케이션 서비스 계층 간의 관심사 분리
- 어플리케이션 내부 통신
- 어플리케이션 간의 통신
  
각 계층의 API를 잘 설계하면 테스트도 육각형 구조를 갖기 시작한다.

### 시스템 내부 통신과 시스템 간 통신

어플리케이션의 시스템 통신은 시스템 내부 통신 / 시스템 간 통신으로 나누어진다.
<img width="600" alt="image (6)" src="https://github.com/KUkingClass/reading-unit-testing/assets/63101979/780b73f2-8b4a-4742-b455-57098b4c54a1">

- 시스템 내부 통신 : 구현 세부 사항. 검증 대상이 아니다.
    - ex) 도메인 클래스 간의 협력
    - 외부 클라이언트 입장에서 도메인 클래스간의 협력은 식별할 수 없는 동작이므로, 구현 세부 사항에 해당 → 검증하면 취약한 테스트가 된다.
- 시스템 간 통신 : 식별할 수 있는 동작. 검증 대상이 될 수 있다.
    - 목은 시스템 간 통신 패턴을 확인할때 사용하면 좋다.
