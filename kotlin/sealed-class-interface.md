# Kotlin sealed class & interface 소개

Kotlin에서 sealed class는 제한된 클래스 계층 구조를 정의할 때 사용하는 강력한 도구입니다. 쉽게 말해, 부모 클래스를 상속받는 자식 클래스의 종류를 컴파일러가 모두 알고 있게 만드는 기능입니다.
결론적으로 이러한 특성 덕분에 개발자의 실수를 줄여주고, 코드의 안정성을 높여줍니다.

## 사용하는 이유

### Enum class의 한계
enum class는 상수 집합을 정의하는 데 유용하지만, 각 상수가 하나의 인스턴스만 가질 수 있다는 단점이 있습니다. 서로 다른 상태 값을 가진 객체를 개별적으로 생성할 수 없습니다.
```kotlin
// UI의 상태를 표현하려고 함 (로딩 중, 성공, 실패)
enum class UiState(val data: String?, val error: Exception?) {
// 로딩: 데이터도 없고 에러도 없음 -> 둘 다 null (낭비)
LOADING(null, null),

    // 성공: 데이터만 있으면 되는데 error 필드에도 null을 넣어야 함
    SUCCESS("불러온 데이터", null),
    
    // 실패: 에러만 있으면 되는데 data 필드에도 null을 넣어야 함
    ERROR(null, Exception("네트워크 오류"));
}
```

### abstract class의 한계
일반적인 상속 관계에서는 컴파일러가 부모 클래스를 상속받은 자식 클래스가 몇 개인지, 무엇인지 전부 알 수 없습니다. 아래 코드에서 else를 추가하여 에러를 없앨 수 있지만, 만약 개발자가 나중에 Jumping이라는 새로운 상태를 추가하고 when 문에 처리를 누락하더라도 컴파일러는 아무런 경고를 주지 않습니다. 이는 런타임 버그의 원인이 됩니다.
```kotlin
abstract class SomeState
class Idle : SomeState()
class Running : SomeState()

fun getStateMessage(state: SomeState): String {
    return when (state) {
    is Idle -> "Idle"
    is Running -> "Running"
    // 문제점: 컴파일러는 다른 자식이 더 있는지 모르기 때문에 반드시 else를 요구함
    else -> ""
    }
}
```

## Sealed class의 핵심 특징

- <b>컴파일 타임에 자식을 인지함</b>: Sealed Class의 모든 하위 클래스는 컴파일 시점에 결정됩니다. 즉, 이 모듈 외부에서는 새로운 하위 클래스를 정의할 수 없습니다. 컴파일 타임에 인지할 수 있는 이유는 하나의 파일 안에 모두 명시해야 하기 때문입니다. 이러한 제약 사항이 큰 이점을 갖습니다.
- <b>추상 클래스 성격</b>: 직접 인스턴스화할 수 없으며(abstract 특징), 추상 멤버를 가질 수 있습니다.
- <b>생성자 제한</b>: 생성자의 가시성은 protected(기본값) 또는 private만 가능합니다.

## When 식의 완전성
Sealed Class를 사용하면 when 식을 사용할 때 else 분기가 필요 없습니다. 컴파일러가 모든 경우의 수를 알고 있기 때문입니다.
```kotlin
sealed class SomeState

object Idle : SomeState()
class Running : SomeState()
object Jumping : SomeState()

fun getStateMessage(state: SomeState): String {
    // else 문이 없어도 에러가 발생하지 않음!
    // 만약 여기서 'Jumping' 처리를 누락하면 컴파일 에러가 발생하여 실수를 방지해줌
    return when (state) {
    is Idle -> "is Idle"
    is Running -> "is Running"
    is Jumping -> "is Jumping"
    }
}
```
이 기능 덕분에 새로운 상태(Class)가 추가되었을 때, 이를 처리하지 않은 모든 코드 부분에서 컴파일 에러가 발생하므로 유지보수가 매우 쉬워집니다.

## 메모리 최적화 : class vs object
Sealed Class의 하위 타입을 정의할 때, 상태(변수)가 필요한지 여부에 따라 class와 object를 구분해서 사용하는 것이 좋습니다.
- <b>상태가 있는 경우 (class 사용)</b>: 객체마다 다른 데이터를 가져야 한다면 class나 data class를 사용합니다.
- <b>상태가 없는 경우 (object 사용)</b>: 단순히 타입만 구분하면 된다면 object(싱글톤 패턴 적용)를 사용합니다.
```kotlin
sealed class SomeState

// [Bad]: 상태가 없는데 class로 선언하면, 매번 새로운 객체가 생성되어 메모리 낭비
class Idle : SomeState() // Warning 발생 가능

// [Good]: 상태가 없다면 object로 선언하여 메모리에 한 번만 올림 (싱글톤)
object Idle : SomeState()

// 상태(speed)가 필요하므로 class 사용은 적절함
class Running(val speed: Int) : SomeState()
```
상태가 없는 하위 클래스를 class로 정의하면 equals나 hashCode를 오버라이딩해야 하거나, 불필요한 객체 생성으로 메모리가 낭비될 수 있다는 경고(Warning)가 발생할 수 있습니다. (코틀린 업데이트로 object -> data object로 사용하는 경우도 많습니다.)

## 비교 : Sealed class & Enum class
| 특징                        | Enum class               | Sealed class                  |
|---------------------------|--------------------------|-------------------------------|
| 인스턴스 생성                   | 각 상수는 단 하나의 객체(Singleton) | 하위 클래스는 여러 인스턴스 생성 가능         |
| 상태                        | 모든 상수가 동일한 구조의 값만 가짐     | 하위 클래스마다 각각 다른 변수/구조를 가질 수 있음 |
| 상속 제어| 불가능| 가능 (계층 구조 형성 가능)              |
|용도|단순 상수 집합 (예: 요일, 색상)|복잡한 상태나 뷰의 유형 등 객체 구조화|

## Sealed interface
sealed interface는 sealed class와 마찬가지로 컴파일러가 모든 구현체를 알 수 있는 제한된 계층 구조를 제공합니다. when 문에서 else가 필요 없는 장점도 동일합니다.
하지만 클래스가 아닌 인터페이스이기 때문에 가질 수 있는 강력한 차별점이 있습니다.

### Sealed interface가 필요한 이유
인터페이스는 개수 제한 없이 구현할 수 있습니다. sealed interface를 사용하면 서로 다른 계층 구조에 속하는 객체를 만들 수 있습니다. 아래 코드에서 PurchaseButton은 ClickEvent로도 처리될 수 있고, AnalyticsLog로도 처리될 수 있습니다.
```kotlin
open class CommonButton

// class 대신 interface 사용
sealed interface ClickEvent

// [성공]
// CommonButton을 상속받으면서 동시에 ClickEvent 그룹의 일원이 됨
class LoginButton : CommonButton(), ClickEvent

// 심지어 여러 sealed interface를 동시에 구현할 수도 있음
sealed interface AnalyticsLog

class PurchaseButton : CommonButton(), ClickEvent, AnalyticsLog
```

### Enum class와 결합
enum class는 클래스이기 때문에 다른 sealed class를 상속받을 수 없습니다. 하지만 인터페이스는 구현할 수 있죠.
따라서 Enum을 Sealed 계층 구조의 일원으로 만들 수 있습니다.
```kotlin
sealed interface PaymentMethod

// data class를 이용한 방식
data class CreditCard(val number: String) : PaymentMethod

// enum class를 이용한 방식 (Sealed Class였으면 불가능)
enum class DigitalWallet : PaymentMethod {
    KAKAO_PAY, NAVER_PAY, APPLE_PAY
}

fun checkout(method: PaymentMethod) {
    when(method) {
    is CreditCard -> println("카드 결제: ${method.number}")
    // Enum 상수들을 한꺼번에 묶어서 처리 가능
    is DigitalWallet -> println("전자 지갑 결제: ${method.name}")
    }
}
```

## Sealed class & Sealed interface 비교

| 특징                                                  | Sealed class                                    | Sealed interface                      |
|-----------------------------------------------------|-------------------------------------------------|---------------------------------------|
| 상태 저장                                               | 가능 (생성자 파라미터 가질 수 있음)                           | 불가능 (프로퍼티는 가질 수 있으나 Backing Field 없음) |
| 상속/구현                                               | 단일 상속만 가능                                       | 다중 상속 가능                              |
| 생성자| 있음 (protected or private)                       | 없음                                    |
| Enum 지원| Enum이 상속받을 수 없음                                 | Enum이 구현 가능                           |
| 사용처| 공통 로직이나 변수가 필요한 계층 구조| 서로 다른 클래스들을 하나의 타입으로 묶을 때             |


## 결론
Sealed Class는 Enum의 타입 안전성과 추상 클래스의 유연함을 합친 기능입니다.
- 상태에 따라 다른 데이터를 가져야 하거나,
- when 문을 통해 모든 케이스를 안전하게 처리하고 싶다면

1. <b>Sealed Class를 쓰세요</b>:
   - 모든 자식들이 공통적으로 가져야 할 변수나 내부 로직이 부모 클래스에 정의되어야 할 때.
   - 계층 구조의 성격이 강할 때.
2. <b>Sealed Interface를 쓰세요</b>:
   - 공통 변수는 필요 없고, 단순히 타입 체크나 그룹화가 목적일 때.
   - 이미 다른 클래스를 상속받고 있는 클래스들을 묶어야 할 때 (다중 상속 필요).
   - enum class를 계층 구조에 포함시키고 싶을 때.

Sealed Class를 사용하는 것은 좋은 선택입니다. 이를 적절히 활용하면 개발자의 실수를 컴파일 단계에서 잡아내어 훨씬 견고한 애플리케이션을 만들 수 있습니다. 최근 코틀린 개발 트렌드에서는 상태(변수) 공유가 굳이 필요 없다면, 더 유연한 Sealed interface를 기본으로 사용하는 것을 권장하기도 합니다.

참고자료 : https://kotlinlang.org/docs/sealed-classes.html
