# Kotlin interface
코틀린에서 인터페이스는 클래스 구조의 설계도 역할을 하며, 추상 멤버(Abstract members)와 구현된 멤버(Implementations)를 모두 포함할 수 있는 강력한 기능을 제공합니다. 자바 8 이후의 인터페이스와 유사하지만, 코틀린만의 고유한 특징들도 가지고 있습니다.

## 기본 정의 및 특징
코틀린 인터페이스는 interface 키워드를 사용하여 정의합니다. 가장 큰 특징은 메서드에 기본 구현을 포함할 수 있다는 점입니다.
```kotlin
interface MyInterface {
fun bar() // 추상 메서드 (구현 없음)

    fun foo() {
      // 본문(body)이 있는 메서드 (선택적 구현)
      println("Default Implementation")
    }
}
```
- bar(): 구현체가 없으므로, 이 인터페이스를 상속받는 클래스에서 반드시 오버라이드(재정의)해야 합니다.
- foo(): 본문이 있으므로, 상속받는 클래스에서 그대로 사용하거나 필요에 따라 오버라이드할 수 있습니다.

## Property
- 인터페이스 내부에 프로퍼티를 선언할 수 있지만, 제약 사항이 있습니다.
- 상태 저장 불가: 인터페이스의 프로퍼티는 값을 저장하는 Backing Field를 가질 수 없습니다.
- 추상 프로퍼티: 구현 클래스에서 반드시 값을 지정해야 합니다.
- 접근자 구현: get()을 통해 값을 계산하거나 반환하는 로직을 인터페이스 내에 정의할 수 있습니다.
```kotlin
interface MyInterface {
val prop: Int // 추상 프로퍼티 (구현 클래스에서 값 지정 필요)

    val propertyWithImplementation: String
        get() = "foo" // 접근자(getter)를 통한 구현 (상태 저장이 아님)

    fun foo() {
        print(prop) // 인터페이스 메서드에서 프로퍼티 접근 가능
    }
}

class Child : MyInterface {
// 추상 프로퍼티는 반드시 오버라이드하여 초기화해야 함
override val prop: Int = 29
}
```

## 인터페이스 상속
인터페이스는 다른 인터페이스를 상속받아 기능을 확장할 수 있습니다. 이때 하위 인터페이스에서 상위 인터페이스의 멤버를 미리 구현할 수 있습니다.
```kotlin
interface Named {
val name: String
}

interface Person : Named {
val firstName: String
val lastName: String

    // Named의 'name'을 Person 인터페이스에서 미리 구현함
    override val name: String 
        get() = "$firstName $lastName"
}

// Employee는 Person을 구현하지만, 'name'은 이미 구현되어 있으므로
// firstName과 lastName만 구현하면 됨
data class Employee(
override val firstName: String,
override val lastName: String,
val position: String
) : Person
```

## 오버라이딩 충돌 해결
- 코틀린 인터페이스는 다중 상속을 지원합니다. 
- 만약 구현하려는 여러 인터페이스에 동일한 이름과 시그니처를 가진 메서드가 있다면 충돌이 발생합니다. 이 경우, 구현 클래스에서 해당 메서드를 반드시 오버라이드하여 어떤 부모의 메서드를 호출할지(또는 어떻게 동작할지) 명시해야 합니다.
- super<인터페이스이름>.메서드명() 문법을 사용하여 특정 부모 인터페이스의 구현을 호출할 수 있습니다.

```kotlin
interface A {
fun foo() { print("A") }
fun bar() // 추상 메서드
}

interface B {
fun foo() { print("B") }
fun bar() { print("bar") } // 구현된 메서드
}

class C : A {
override fun bar() { print("bar") } // A의 bar 구현
}

class D : A, B {
// A와 B 모두 foo()를 가지고 있으므로 충돌 발생 -> 반드시 오버라이드 필요
override fun foo() {
super<A>.foo() // A의 foo 호출
super<B>.foo() // B의 foo 호출
}

    // A는 추상, B는 구현된 bar()를 가짐 -> D에서 명확히 재정의하거나 B의 것을 사용하도록 명시
    override fun bar() {
        super<B>.bar() // B의 bar 호출
    }
}
```

## abstract class와의 차이점
가장 중요한 점은 상태 저장 유무입니다.
- 인터페이스: 상태를 저장할 수 없습니다 (Backing Field 없음).
- 추상 클래스: 상태를 저장할 수 있습니다.

## 결론
- 유연성: 추상 메서드와 구현된 메서드(default method)를 모두 가질 수 있습니다.
- 상태 없음: 프로퍼티는 가질 수 있으나, 값을 저장하는 필드(Backing Field)는 가질 수 없습니다.
- 다중 구현: 클래스는 여러 인터페이스를 구현할 수 있으며, 이름 충돌 시 super<Type>으로 명시적 해결이 가능합니다.
- 계층 구조: 인터페이스끼리 상속이 가능하며, 이를 통해 재사용성 높은 설계를 할 수 있습니다.

참고자료 :
https://kotlinlang.org/docs/interfaces.html#resolving-overriding-conflicts
