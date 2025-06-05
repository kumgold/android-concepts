# Kotlin object keyword

## Overview
object 키워드는 다양한 상황에서 사용되는데, 별도의 생성자 호출 없이 단 하나의 인스턴스만 생성할 수 있고, 이 인스턴스를 통해 멤버에 접근할 수 있다.

<br><br>

## Object Declaration (객체 선언)
- 싱글톤 패턴을 구현하는 방법이다.
- 단 하나의 인스턴스를 생성하며, 프로그램 전체에서 이 인스턴스를 공유한다.
```
object MySingleton {
    fun doSomething() {
        println("Singleton 객체의 함수 호출")
    }
}

fun main() {
    MySingleton.doSomething()
}
```

<br><br>

## Object Expression (객체 표현식)
익명 클래스의 객체를 만든다. 이러한 객체는 일회성으로 사용할 때 유용하다. 처음부터 정의할 수도 있고, 기존 클래스를 상속 받거나 인터페이스를 구현할 수 있다.

### 슈퍼 타입이 없는 object 객체
```
val helloWorld = object {
    val hello = "Hello"
    val world = "World"
    // object expressions extend Any, so `override` is required on `toString()`
    override fun toString() = "$hello $world"
}
```

### 슈퍼 타입이 있는 object 객체 (상속 or 인터페이스 구현)
```
window.addMouseListener(object : MouseAdapter() {
    override fun mouseClicked(e: MouseEvent) { /*...*/ }
    override fun mouseEntered(e: MouseEvent) { /*...*/ }
})
```

<br><br>

## Companion object
클래스 안에서 object를 구현하려면 companion object 키워드를 사용해서 구현할 수 있다.
Java의 static 초기화와 의미가 같으며, 해당 클래스가 로드되면 companion object 블록이 초기화된다.
```
// 일반적인 companion object
class MyClass {
    companion object {
        fun create(): MyClass = MyClass()
    }
}
// 슈퍼 타입이 있는 Named companion object
class MyClass {
    companion object : Factory<MyClass> {
        override fun create(): MyClass = MyClass()
    }
}
// 접근 방식
val instance = MyClass.create()
```

<br><br>

## Data object
object로 생성된 객체는 유용하지만, 한 가지 아쉬운 점이 있다면 디버깅을 위해 로그를 찍었을 때 해당 객체의 주소값이 표현된다.
```
object MyObject
fun main() {
    println(MyObject) // MyObject@1f32e575
}
```

data object 키워드는 위의 단점을 보완한 것으로 toString(), equals(), hashCode()와 같은 함수들을 자동으로 생성하고 구현해준다.
```
data object MyDataObject {
    val x: Int = 3
}
fun main() {
    println(MyDataObject) // MyDataObject
}
```

### data class, data object의 차이점
- data object는 copy() 함수가 없다. 싱글톤이기 때문에 필요 없기 때문이다.
- componentN() 함수가 없다. data object는 이런 속성을 갖고 있지 않기 때문이다.
