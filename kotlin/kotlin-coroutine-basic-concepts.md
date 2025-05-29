# Kotlin Coroutine Basic Concepts

## 목차
- Android & Main Thread
- What is Coroutine?
- Suspend Keyword

## Android & Main Thread
안드로이드의 View는 메인 스레드에서 업데이트가 된다. 그 이유는 서로 다른 스레드에서 View를 동시에 업데이트 하면 문제가 발생할 수 있기 때문이다.

안드로이드는 프레임 렌더링을 위해서 60fps(1초에 60 프레임 렌더링), 16ms(하나의 프레임을 렌더링하는데 걸리는 시간)에 달성해야 한다고 한다. 
이 이하로 떨어지게 되면 시각적으로 봤을때, 버벅거림이 발생하고 사용자에게 안 좋은 경험을 제공할 수 있기 때문이다.

<img src="/images/kotlin-coroutine-basic-concepts1.gif" alt=""><br>
출처 : 나무위키

<br><br>

스레드는 한 번에 하나의 일을 처리할 수 있다. 메인 스레드는 View 업데이트에 사용되어야 한다. 만약 View 업데이트가 아닌 다른 작업이 많아지거나 오래 걸린다면 메인스레드가 View 업데이트를 하지 못하게 된다.
결과적으로 View 업데이트에 딜레이가 생기고 사용자가 느꼈을때 앱이 끊기는 것처럼 보일 수 있게 된다.
<img src="/images/kotlin-coroutine-basic-concepts2.png" alt=""><br>

<br><br>

만약, 메인스레드가 어떤 무거운 연산에 의해서 다른 작업을 할 수 없는 차단된 상태가 유지되면 ANR이 발생하고 앱이 강제 종료된다.

<img src="/images/kotlin-coroutine-basic-concepts3.png" alt=""><br>
출처 : 안드로이드 공식 문서

<br><br>

그렇기 때문에 메인 스레드가 멈추지 않도록 복잡한 작업은 다른 Worker Thread에서 진행하도록 구현하는 것이 필요하다.
흔히 네트워크 요청이나 데이터베이스 접근 같은 로직은 메인 스레드가 아닌 Worker Thread에서 작동하는 것이 권장된다.


## What is Coroutine?
Coroutine은 일시 중단이 가능한 계산 인스턴스이다. 일시 중단이 가능하다는 말은 진행중인 **작업A**를 잠시 멈추고 다른 **작업B**를 마치고 다시 **작업A**를 수행할 수 있다는 뜻이다.
코루틴은 실행할 위치를 지정할 수 있는 Dispatcher를 제공하여 기본 스레드 외부에서 작업을 진행할 수 있다.

<img src="/images/kotlin-coroutine-basic-concepts4.png" alt=""><br>
출처 : 안드로이드 유튜브

<br><br>

왼쪽에 blocking 버전은 메인 스레드에서 fetchUser() 네트워크 요청을 호출하고 결과를 받기까지 메인 스레드가 다른 작업을 하지 못한다.
이 시간동안 사용자는 터치 이벤트나 다른 작업을 수행할 수 없게 된다.
<br><br>
반면에 coroutine 버전은 메인 스레드에서 fetchUser() 네트워크 요청을 호출하는데 메인이 아닌 다른 스레드에서 작업을 수행하도록 한다.
메인 스레드에서 fetchUser() 작업이 일시 중단 되는 것이다. 작업이 일시 중단 되었기 때문에 메인 스레드는 Free 상태가 되어 onDraw() 작업을 수행할 수 있다.


## Suspend Keyword




<hr>
참고 자료 : <br>
https://youtu.be/BOHK_w09pVA?si=vm3ZjlCJ7xf7P5pm