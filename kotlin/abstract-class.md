# Kotlin Abstract class
Kotlin에서 abstract class는 말 그대로 추상적인(미완성된) 클래스를 의미합니다. 스스로는 객체(인스턴스)를 만들 수 없으며, 오직 다른 클래스가 이를 상속받아 완성시킬 때만 의미를 가집니다. 마치, 건물을 짓기 위한 청사진(Blueprint)이나 템플릿과 같은 역할을 합니다.

## 특징
- 인스턴스화 불가: abstract class는 미완성 상태이므로 직접 객체를 생성할 수 없습니다.
- 멤버의 구성:
  - 추상 멤버 (Abstract Member): 구현부 없이 선언만 된 멤버. 상속받는 자식 클래스가 반드시 구현해야 합니다. (open 키워드 불필요)
  - 구현된 멤버 (Concrete Member): 이미 로직이 구현된 멤버. 자식 클래스가 그대로 쓰거나 필요시 오버라이드 할 수 있습니다.
- 상태(State) 저장: 인터페이스와 달리, 상태 값(Backing Field)을 가질 수 있습니다.

## 예제
아래 예제에서 getSize()나 vertex처럼 변하지 않는 공통 로직은 미리 만들어두고, width나 isSquare()처럼 구체적인 값이 필요한 부분만 자식에게 맡깁니다.
```kotlin
abstract class Rectangle {
// [구현된 멤버]
// 모든 사각형의 꼭짓점은 4개로 동일함 (상태 저장)
val vertex = 4

    // [추상 프로퍼티]
    // 가로, 세로 길이는 자식마다 다름 -> 구현 강제
    abstract val width: Int 
    abstract val height: Int 

    // [구현된 메서드]
    // 넓이를 구하는 공식은 가로*세로로 동일함 -> 미리 구현
    fun getSize(): Int = width * height 

    // [추상 메서드]
    // 정사각형인지 판별하는 로직은 상황에 따라 정의 필요 -> 구현 강제
    abstract fun isSquare(): Boolean
}

class MyRectangle : Rectangle() {
// 추상 멤버는 반드시 오버라이드하여 값을 채워야 함
override val width: Int = 10
override val height: Int = 20

    override fun isSquare(): Boolean = width == height
}
```

## 고급 기능
Kotlin에서는 독특하게도 상위 클래스에서 이미 구현된(open) 메서드를 하위 추상 클래스에서 다시 abstract로 선언하여 구현을 강제로 초기화할 수 있습니다.
```kotlin
open class Polygon {
    open fun draw() {
        println("기본 도형 그리기")
    }
}

abstract class WildShape : Polygon() {
// 부모인 Polygon에 draw() 구현체가 있지만,
// WildShape를 상속받는 자식들은 부모의 기본 draw()를 쓰지 말고
// 반드시 자신만의 draw()를 새로 만들도록 강제함
abstract override fun draw()
}
```

## Abstract class & Interface
| 구분                     | Abstract class     |Interface|
|------------------------|--------------------|---------|
|목적|밀접하게 관련된 클래스들의 공통 조상<br/>(is-A, 무엇인가?)|서로 다른 클래스들의 공통 행동 정의<br/>(can-do, 무엇을 할 수 있나?)|
|상태|가능(Backing Field 존재 가능). 공통 데이터 저장 가능|불가능. 상태를 저장할 수 없음|
|상속|단일 상속|다중 상속|
|유연성|낮음 (부모와 강한 결합)|높음 (필요한 기능만 Sticky하게 사용 가능)|

## 결론 및 사용 가이드
- Abstract Class를 사용하세요:
  - 관련된 클래스들 사이에 코드를 재사용하고 싶을 때.
  - 클래스들이 공통적인 상태(필드)를 가져야 할 때.
  - 부모-자식 관계가 명확할 때 (예: 사각형은 도형이다).
- Interface를 사용하세요:
  - 서로 관련 없는 클래스들이 동일한 동작을 해야 할 때.
  - 다중 상속을 통해 유연한 설계가 필요할 때.
  - 상태 저장 없이 행동(메서드)만 정의하고 싶을 때.

참고자료 :
https://kotlinlang.org/docs/classes.html#abstract-classes