# Kotlin Coroutine Basic Concepts

## 목차
- Android & Main Thread
- What is Coroutine?
  - Suspend Keyword
  - Structured concurrency
- CPS(Continuation Passing Style)

## Android & Main Thread
안드로이드의 View는 메인 스레드에서 업데이트가 된다. 그 이유는 서로 다른 스레드에서 View를 동시에 업데이트 하면 문제가 발생할 수 있기 때문이다.

안드로이드는 프레임 렌더링을 위해서 60fps(1초에 60 프레임 렌더링), 16ms(하나의 프레임을 렌더링하는데 걸리는 시간)에 달성해야 한다고 한다. 
이 이하로 떨어지게 되면 시각적으로 봤을때, 버벅거림이 발생하고 사용자에게 안 좋은 경험을 제공할 수 있기 때문이다.

<img src="/images/kotlin-coroutine-basic-concepts1.gif" alt=""><br>
출처 : 나무위키

<br>
스레드는 한 번에 하나의 일을 처리할 수 있다. 메인 스레드는 View 업데이트에 사용되어야 한다. 만약 View 업데이트가 아닌 다른 작업이 많아지거나 오래 걸린다면 메인스레드가 View 업데이트를 하지 못하게 된다.
결과적으로 View 업데이트에 딜레이가 생기고 사용자가 느꼈을때 앱이 끊기는 것처럼 보일 수 있게 된다.
<img src="/images/kotlin-coroutine-basic-concepts2.png" alt=""><br>

<br>
만약, 메인스레드가 어떤 무거운 연산에 의해서 다른 작업을 할 수 없는 차단된 상태가 유지되면 ANR이 발생하고 앱이 강제 종료된다.

<img src="/images/kotlin-coroutine-basic-concepts3.png" alt=""><br>
출처 : 안드로이드 공식 문서

<br>
그렇기 때문에 메인 스레드가 멈추지 않도록 복잡한 작업은 다른 Worker Thread에서 진행하도록 구현하는 것이 필요하다.
흔히 네트워크 요청이나 데이터베이스 접근 같은 로직은 메인 스레드가 아닌 Worker Thread에서 작동하는 것이 권장된다.

<br>

## What is Coroutine?
Coroutine은 일시 중단이 가능한 계산 인스턴스이다. 일시 중단이 가능하다는 말은 진행중인 **작업A**를 잠시 멈추고 다른 **작업B**를 작업한 후 다시 **작업A**를 수행할 수 있다는 뜻이다.
코루틴은 실행할 위치를 지정할 수 있는 Dispatcher를 제공하여 기본 스레드 외부에서 작업을 진행할 수 있다.

<img src="/images/kotlin-coroutine-basic-concepts4.png" alt=""><br>
출처 : 안드로이드 유튜브

<br>
왼쪽에 blocking 버전은 메인 스레드에서 fetchUser() 네트워크 요청을 호출하고 결과를 받기까지 메인 스레드가 다른 작업을 하지 못한다.
이 시간동안 사용자는 터치 이벤트나 다른 작업을 수행할 수 없게 된다.
<br><br>
반면에 coroutine 버전은 메인 스레드에서 fetchUser() 네트워크 요청을 호출하는데 메인이 아닌 다른 스레드에서 작업을 수행하도록 한다.
메인 스레드에서 fetchUser() 작업이 일시 중단 되는 것이다. 작업이 일시 중단 되었기 때문에 메인 스레드는 Free 상태가 되어 onDraw() 작업을 수행할 수 있다.

<br>

### Suspend Keyword
suspend가 붙은 함수는 일시 중단이 가능하다. 함수 내부에서 Coroutine을 호출할 수 있으며 다른 suspend 함수를 호출할 수 있다.
```
suspend fun doWorld() {
    println("Hello")
    delay(1000L) // Suspend 함수 호출
    println("World!")
}
```

### Structured concurrency
Coroutine은 특정 Scope 안에서만 실행될 수 있다. 이러한 Scope를 CoroutineScope라고 부른다. CoroutineScope는 Coroutine의 실행 범위를 생성한다.
CoroutineScope 내부에 여러 Coroutine을 호출할 수 있으며, 모든 하위 Coroutine이 완료될 때까지 외부 Scope를 완료할 수 없다.
만약 하위 Coroutine이 예외를 발생시켰다면, 하위 Coroutine을 호출한 최초 부모 Coroutine에 예외를 전달하여 해당 범위 안에 모든 Coroutine을 종료시킨다.

Structured concurrency는 Coroutine이 손실되거나 누출되지 않도록 보장한다. 또한, 모든 오류가 올바르게 전달되어 손실을 막는다.
```
fun main() = runBlocking { // CoroutineScope
    launch { doWorld() } // 하위 코루틴
    launch { doWorld() } // 하위 코루틴
}
```

## CPS(Continuation Passing Style)
Suspend 키워드가 붙어 있는 함수는 코틀린 컴파일러에 의해서 함수 내부에 마지막 파라미터로 Continuation이 추가된다.
Continuation은 Callback이다. CoroutineScope 안에서 suspend 함수가 호출된 시점에 실행 정보들을 Continuation 객체로 만들어 캐시에 저장하고
실행이 Resume 되었을때 저장한 실행 정보를 바탕으로 재개된다. 아래에 간단한 예제가 있다.


suspend 함수 안에서 또 다른 suspend 함수를 호출하는 함수가 있다. 이 함수를 CoroutineScope 안에서 호출한다고 가정하자.
```
suspend fun postItem(item: Item) {
    val token = requestToken() // suspend function
    val post = createPost(token, item) // suspend function
    processPost(post) // suspend function
}
```

만약 위 함수가 코루틴 컴파일러에 의해서 컴파일된 모습을 개념적으로 간단하게 표현한다면 아래와 유사한 형태를 갖는다.
(실제로는 Kotlin으로 작성되어 있지 않다.)
```
fun postItem(item: Item, cont: Continuation) {
    // 실행 순서 1 
    val stateMachine = object :  CoroutineImpl(cont) { 
        fun resumeWith(...) {
            postItem(null, this) // // 실행 순서 3
        } 
    }
    
    when (stateMachine.label) {
        0 -> { // // 실행 순서 2
            stateMachine.item = item
            stateMachine.label += 1
            requestToken(stateMachine)
        }
        1 -> { // 실행 순서 4
            val item = stateMachine.item
            val token = stateMachine.result
            stateMachine.label += 1
            createPost(token, item, stateMachine)
        }
        ...
    }
    ...
}
```

suspend 함수 안에서 실행되는 다른 suspension point는 label로 식별한다. 0부터 순서대로 suspend 함수를 실행과 동시에 COROUTINE_SUSPENDED 라는 특수한 값을 리턴한다.
이 값을 리턴하면 해당 스레드에서 실행된 함수는 suspend 상태가 되어 스레드는 다른 일을 할 수 있다.

suspend 함수가 중단되었을때 실행 정보는 Continuation 객체에 저장되어 있다.
이후 request가 완료되었다고 판단되었을때 Continuation(Callback)을 전달 받은 Impl 객체는 resumeWith를 실행한다.

resumeWith를 통해 재실행되었을때 label 값에 따라 다음 suspend 함수가 실행된다.
실행 정보는 Continuation에 저장되어 있기 때문에 처음부터 재실행 하는 것이 아닌 중단된 시점부터 코드를 실행하는 것이 가능하다.

<hr>
참고 자료 : <br>
https://youtu.be/BOHK_w09pVA?si=vm3ZjlCJ7xf7P5pm <br>
https://youtu.be/YrrUCSi72E8?si=cfnSXletyzdbHqG9 <br>
https://kotlinlang.org/docs/coroutines-basics.html#extract-function-refactoring <br>
