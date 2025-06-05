# Understanding Android Compose 2

[Previous](./android-compose-1.md)

## Overview
앞서 Compose가 생긴 이유와 Compose와 상속, Recomposition 같은 개념에 대해서 작성했다.

<br><br>

## What is @Composable
먼저 Compose를 사용할 때 Annotation processor는 필요하지 않다. Compose는 Kotlin의 유형 검사 및 코드 생성 단계에서 코틀린 컴파일러 플러그인의 도움을 받아 동작한다. @Composable을 보면 분명 어노테이션이 붙어 있기 때문에 잘 이해가 되지 않지만, Composable annotation은 annotation보다는 키워드에 더 가깝다고 한다. (예 : suspend keyword)


suspend keyword가 붙은 함수를 보면, 다양한 방식으로 선언할 수 있다.
```
// function declaration
suspend fun MyFun() { … }
// lambda declaration
val myLambda = suspend { … }
// function type
fun MyFun(myParam: suspend () -> Unit) { … }
```

<br>

@Composable 함수도 마찬가지다.
```
// function declaration
@Composable fun MyFun() { … }
// lambda declaration
val myLambda = @Composable { … }
// function type
fun MyFun(myParam: @Composable () -> Unit) { … }
```

<br>

suspend와 마찬가지로 @Composable이 포함된 함수는 그렇지 않은 함수와 호환되지 않는다. suspend와 마찬가지로 타입이 변경된다.
```
// suspend
fun Example(a: () -> Unit, b: suspend () -> Unit) {
   a() // allowed
   b() // NOT allowed
}
suspend fun Example(a: () -> Unit, b: suspend () -> Unit) {
   a() // allowed
   b() // allowed
}
---------------------------------------------------------------
// @Composable
fun Example(a: () -> Unit, b: @Composable () -> Unit) {
   a() // allowed
   b() // NOT allowed
}
@Composable 
fun Example(a: () -> Unit, b: @Composable () -> Unit) {
   a() // allowed
   b() // allowed
}
```

@Composable 함수가 이렇게 동작하는 이유는 @Composable 함수를 처리하는 Calling Context Object라고 부르는 특정 Thread가 필요하기 때문이다.

<br><br>

## 실행 모델
Compose의 실행 모델은 Composer라고 부르는데, Composer의 구현은 Gap Buffer와 같은 데이터 구조가 포함되어 있다.
Gap Buffer는 일반적으로 텍스트 편집기에서 사용되며, 현재 Index(or Cursor)가 있는 Collection 데이터 구조를 나타낸다.
Flat Array 형태로 메모리에 구현되어 있고, 사용되지 않는 공간을 Gap이라고 부른다.

<img src="/images/android-compose-2_1.png" width="300">

Compose의 계층 구조가 실행되면 Gap Buffer에 데이터가 삽입된다.

<img src="/images/android-compose-2_2.png" width="300">

Compose의 계층 구조는 분명 재구성(Recomposition)이 발생한다. 이 때 Index(or Cursor)의 위치를 첫 번째 위치로 다시 설정한 다음, 처음과 같은 실행 과정을 거친다.
실행 과정을 거치는 동안 Gap Buffer 데이터 구조 안에 삽입된 데이터는 변경될 수도 있고 그렇지 않을 수도 있다.

<img src="/images/android-compose-2_3.png" width="300">

혹은 UI 계층 구조가 변경되어 새로운 데이터를 삽입해야 할 수도 있다. 이 때는 Gap을 현재 Index(or Cursor)로 이동시킨다.

<img src="/images/android-compose-2_4.png" width="300">

그 다음 데이터를 삽입한다.

<img src="/images/android-compose-2_5.png" width="300">

마지막으로 모든 데이터를 현재 위치로 순차적으로 이동시키고 변경 과정이 종료된다.
Gap Buffer의 작업은 Gap을 이동시키는 작업을 제외하고 다른 작업(예: get, insert, delete 등)의 작업 시간은 상수 시간에 이루어진다. Gap을 이동시키는 작업만 O(N) 작업 시간을 갖는다.
이러한 데이터 구조를 선택한 이유는 일반적으로 UI의 계층 구조가 변경되는 경우는 적다고 판단했기 때문이다. 값이 변하는 상황은 자주 있겠지만, UI 구조가 변하는 상황은 적기 때문에 적절한 Trade Off로 생각된다.



