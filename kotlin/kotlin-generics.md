## Kotlin Generics 핵심 요약 (면접 답변용)

Q : Kotlin의 제네릭에 대해 설명해주세요 <br>
A : 제네릭(Generics)은 컴파일 타임에 타입 안정성(Type Safety)을 보장하고, 코드의 재사용성을 높이기 위한 기능입니다. <br>
- 타입 안정성: 런타임에 발생할 수 있는 CastException을 컴파일 시점에 미리 막아줍니다. <br>
- 재사용성: 하나의 클래스나 함수가 다양한 데이터 타입을 처리할 수 있도록 일반화(Generalize)해줍니다. <br>
- 특히, Kotlin 제네릭의 가장 큰 특징은 가변성(Variance)을 다루는 방식입니다. Java의 와일드카드(? extends, ? super) 대신, in (반공변성)과 out (공변성) 키워드를 사용하여 더 직관적으로 선언 지점 변성(Declaration-site variance)을 지원한다는 점입니다. <br>


## 핵심 질문 & 모범 답안
면접관은 기본적인 정의를 넘어서, in/out의 의미와 제약 조건을 정확히 아는지 물어볼 확률이 높습니다.

### Q1. 공변성(ut)과 반공변성(in)의 차이를 설명해주세요.
- 답변 전략: 방향성과 읽기/쓰기 제약을 중심으로 설명하세요.
이 둘은 타입 간의 상속 관계가 제네릭 클래스 간의 관계에 어떻게 영향을 미치는지를 정의합니다.
- 공변성 (out):
  - 개념: Cat이 Animal의 자식일 때, Box\<Cat>도 Box\<Animal>의 자식으로 인정받는 것입니다. (상속 관계 유지)
  - 제약 (Producer): 클래스 내부에서 T 타입의 값을 생산(반환/읽기)만 할 수 있고, 소비(입력/쓰기)할 수는 없습니다.
  - 예시: List\<out E> (읽기 전용 리스트). List\<String>은 List\<Any>에 할당될 수 있습니다.
- 반공변성 (in):
  - 개념: 상속 관계가 뒤집힙니다. Cat이 Animal의 자식일 때, Box\<Animal>이 Box\<Cat>의 자식으로 인정받습니다. (상속 관계 역전)
  제약 (Consumer): 클래스 내부에서 T 타입의 값을 소비(입력/쓰기)만 할 수 있고, 생산(반환)할 수는 없습니다.
  - 예시: Comparable<in T>. Comparable\<Number>는 Comparable\<Int>에 할당될 수 있습니다. 숫자를 비교할 수 있는 놈은 정수도 비교할 수 있기 때문입니다.

### 공변성 (Covariance, out) - "읽기 전용"
상황: Cat은 Animal입니다. (Cat < Animal)
그렇다면, List\<Cat>은 List\<Animal>이라고 할 수 있을까요?
- 일반 제네릭 (무변성): NO. 서로 남남입니다.
- 공변성 (out): YES. List\<Cat>은 List\<Animal>의 하위 타입이 됩니다.

왜 out(생산)만 가능할까요? <br>
List\<Animal> 변수에 List\<Cat> 객체를 넣었다고 칩시다.
- 읽기 (Get): 리스트에서 꺼내면 Cat이 나옵니다. Cat은 Animal이니까 안전합니다. (OK)
- 쓰기 (Add): List\<Animal>이니까 Dog를 넣으려고 시도할 수 있습니다. 하지만 실제 객체는 List\<Cat>입니다. 고양이 리스트에 개를 넣으면 안 되죠! 위험합니다. (Fail)
-> 그래서 out을 붙이면 컴파일러가 "쓰기(입력) 금지!"를 때려버립니다. 
- 오직 읽기(생산)만 허용해서 타입 안정성을 지키는 것입니다.

코드 예제: 햄버거 가게 (생산자)
```kotlin
open class Burger
class CheeseBurger : Burger()
// out T: T를 생산(반환)만 하겠다. (읽기 전용 메뉴판)
interface BurgerProducer<out T> {
  fun produce(): T
  // fun consume(item: T) // 컴파일 에러! (입력 불가)
}

fun main() {
  val cheeseBurgerStore: BurgerProducer<CheeseBurger> = object : BurgerProducer<CheeseBurger> {
    override fun produce(): CheeseBurger = CheeseBurger()
  }
  // 공변성: CheeseBurger 생산자는 Burger 생산자라고 할 수 있다.
  // 왜? 치즈버거는 버거니까!
  val burgerStore: BurgerProducer<Burger> = cheeseBurgerStore
  val myBurger: Burger = burgerStore.produce() // 안전함!
}
```

### 반공변성 (Contravariance, in) - "쓰기 전용"
상황: Cat은 Animal입니다. <br>
그렇다면, Comparator\<Animal>(동물 비교기)은 Comparator\<Cat>(고양이 비교기)을 대신할 수 있을까요? <br>
- 반공변성 (in): YES. 상속 관계가 뒤집혀서 Comparator\<Animal>이 Comparator\<Cat>의 자식(하위 타입)처럼 동작합니다.

왜 역전될까요? <br>
우리는 고양이들을 정렬하기 위해 "비교하는 도구"가 필요합니다.
- "고양이 전용 비교기": 고양이끼리만 비교 가능합니다.
- "동물 전체 비교기": 고양이든 개든 다 비교할 수 있습니다.
- 따라서, "동물 비교기"는 "고양이 비교기"의 역할을 완벽하게 수행(대체)할 수 있습니다. 더 일반적인 놈이 더 구체적인 놈을 대체할 수 있는 상황, 이것이 반공변성입니다.
 
왜 in (소비)만 가능할까요? <br>
- 쓰기 (입력): 비교기(Comparator\<Animal>)에게 Cat 두 마리를 던져줍니다(in). Animal을 비교할 줄 아는 놈이니까 Cat 비교는 식은 죽 먹기입니다. 안전합니다. (OK)
- 읽기 (반환): 비교기가 내부적으로 가지고 있는 T를 꺼낸다고 칩시다. Comparator\<Cat> 타입 변수라면 Cat이 나오길 기대하겠죠? 하지만 실제 객체는 Comparator\<Animal>이라서 Dog나 Bird를 가지고 있을 수도 있습니다. 위험합니다. (Fail)
-> 그래서 in을 붙이면 컴파일러가 "읽기(반환) 금지!"를 때려버립니다.
코드 예제: 쓰레기통 (소비자)



```kotlin
open class Trash

class Plastic : Trash()
// in T: T를 소비(입력)만 하겠다. (쓰레기통)
interface TrashCan<in T> {
  fun throwAway(item: T)
  // fun recycle(): T // 컴파일 에러! (반환 불가)
}

fun main() {
  val generalTrashCan: TrashCan<Trash> = object : TrashCan<Trash> {
      override fun throwAway(item: Trash) {
          println("버려짐: $item")
      }
  }
  // 반공변성: "일반 쓰레기통"은 "플라스틱 쓰레기통"의 역할을 대신할 수 있다.
  // 왜? 일반 쓰레기통에는 플라스틱도 버릴 수 있으니까!
  val plasticTrashCan: TrashCan<Plastic> = generalTrashCan
  
  plasticTrashCan.throwAway(Plastic()) // 안전함!
}
```

## 요약: 면접용 멘트
- 공변성(out)은 List\<String>이 List\<Any>가 될 수 있게 해주는 것입니다. 데이터를 꺼내는(Produce) 것만 허용함으로써 타입 안정성을 보장합니다.
- 반공변성(in)은 Comparable\<Number>가 Comparable\<Int>를 대신할 수 있게 해주는 것입니다. 상위 타입 처리기가 하위 타입 처리기를 대체할 수 있다는 논리이며, 데이터를 넣는(Consume) 것만 허용합니다.
- 이 둘을 적절히 사용하면 라이브러리 설계 시 훨씬 유연하고 재사용 가능한 API를 만들 수 있습니다.

### Q. reified 키워드는 무엇이며, 언제 사용하나요?

이 질문은 제네릭의 한계(Type Erasure)와 코틀린만의 해결책을 묻는 킬러 문항입니다.  
- 타입 소거(Type Erasure) 문제를 해결하기 위해 사용합니다.
- JVM 제네릭은 런타임에 타입 정보가 지워집니다. 그래서 일반적인 제네릭 함수에서는 T::class.java처럼 타입 클래스에 접근하거나 is T 같은 타입 검사를 할 수 없습니다.
- 함수를 **inline**으로 선언하고 제네릭 타입 앞에 reified 키워드를 붙이면, 컴파일러가 함수의 바이트코드를 호출 지점에 복사해 넣으면서 실제 타입 정보도 함께 넣어줍니다. 이를 통해 런타임에도 제네릭 타입을 실체화된(Reified) 타입처럼 다룰 수 있게 됩니다.

 
실무 예시: Gson이나 Retrofit을 사용할 때 fromJson<User>(jsonString) 처럼 클래스 타입을 파라미터로 넘기지 않고 직관적으로 사용할 때 주로 활용합니다.
// reified 예시
```kotlin
inline fun <reified T> String.toObject(): T {
    return Gson().fromJson(this, T::class.java)
}
```

### Q. 제네릭 제약(Generic Constraints)은 어떻게 거나요? (where)
- 기본적으로 제네릭 T는 Any?로 간주되지만, 콜론(:)을 사용해 상한(Upper Bound)을 지정할 수 있습니다.
- 예를 들어 \<T : Number>라고 하면 Number와 그 자식들만 들어올 수 있습니다.
- 만약 두 개 이상의 제약 조건이 필요하다면 where 키워드를 사용합니다. 예를 들어 \<T> ... where T : CharSequence, T : Comparable\<T>와 같이 작성하면, 문자열이면서 비교 가능한 타입만 허용할 수 있습니다.

### 면접관에게 어필할 수 있는 한 문장 (Insight)
안드로이드 개발에서 제네릭은 단순히 유틸 함수를 만들 때뿐만 아니라, BaseActivity\<VB : ViewBinding>이나 BaseViewModel 같이 보일러플레이트 코드를 줄이는 기반 클래스(Base Class)를 설계할 때 핵심적인 역할을 한다고 생각합니다.
특히 reified와 inline 함수를 적절히 활용하면 API 호출이나 JSON 파싱 코드를 훨씬 안전하고 간결하게 만들 수 있습니다.