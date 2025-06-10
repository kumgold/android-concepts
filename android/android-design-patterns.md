# Android Design Patterns

## MVC (Model-View-Controller)

<img src="/images/android-design-pattern-1.png" width="400px">

안드로이드에서 Controller 역할은 Activity(or Fragment)가 담당하게 된다. 
단점은 안드로이드에서 Activity(or Fragment)는 View까지 담당하기 때문에 Controller, View 코드에 커지고 유지보수가 어려워진다.

1. 모든 입력(Input)은 Controller로 전달된다.
2. Controller는 입력에 따라 필요한 Model을 업데이트한다.
3. Model의 업데이트 결과는 View를 선택한다. (Controller : View → 1 : N)
4. View를 업데이트 하기 위해서는
   - View가 Model을 직접 이용하여 업데이트
   - Model에서 View에 Notification을 전달하여 업데이트

<br/>

### 장점
- 단순하다. Activity에 Controller, View 코드를 모두 작성하기 때문에 가장 단순한 패턴이라고 할 수 있다.

### 단점
- 앱이 커질수록 유지보수가 어려워진다. 가장 단순한 코드에서 오는 단점.
- Controller-View가 결합되어 있어서 테스트 코드 작성이 어렵다.

<br/><br/>

## MVVM (Model-View-ViewModel)
<img src="/images/android-design-pattern-3.png" width="400px">

MVVM 패턴은 Observer를 적극 활용하여 객체의 변경이 일어날때마다 UI를 갱신한다. 의존성을 줄여 안드로이드에 더욱 맞춤화된 패턴이다.

1. 모든 입력(Input)은 View로 전달된다.
2. ViewModel은 Input에서 발생하는 Logic을 처리하여 데이터를 변경하거나 Model에 알린다.
3. Model에서 데이터가 변경되면 ViewModel에 알린다.
4. View는 ViewModel의 해당 데이터를 직접 참조하지 않고, 알림을 설정하여 ViewModel에서 데이터가 변경되었을 때 해당 알림을 받도록 설정한다. 알림을 받으면 View를 업데이트할 수 있다.

<br/>

### 장점
- View(UI Logic)와 Model(Business Logic) 사이의 의존성이 없다.
- ViewModel과 View 사이의 의존성이 없다.
- 중복되는 코드를 모듈화할 수 있어 재사용에 용이하다.

### 단점
- ViewModel의 설계가 어렵다.

<br/><br/>

## MIV (Model-Intent-View)
<img src="/images/android-design-pattern-4.png" width="400px">

MVI의 Intent는 사용자의 의도를 나타낸다. 사용자 Input에 대한 가정을 Intent로 나타내며 Intent는 사용자의 행동을 Observe한다. 즉, Intent는 앱의 상태를 바꾸려는 의도이다.

1. 모든 입력(Input)은 Intent를 나타낸다.
2. Intent, 앱의 상태가 변경되면 Model은 해당 데이터를 변경한다.
3. 변경된 데이터에 의한 결과물 View가 출력된다.
4. 출력된 View에 다른 Intent가 적용되면 이 사이클이 반복된다.

MVI의 데이터는 단방향이다. 앱은 상태에 초점을 맞추고 상태에 따라 데이터가 변하고 View가 변하는 하나의 사이클이 반복적으로 일어난다. 이론적으로 MVI는 순수 함수와 가장 닮은 모습으로 어떤 상태에 따른 결과물이 항상 같을 수 있음을 의미한다. 그렇기 때문에 불변성으로 인하여 View를 예측하기 쉽고 디버깅이 쉽다.

<img src="/images/android-design-pattern-5.png" width="400px">

그러나, 어떤 실행을 위 사이클에 담을 수 없는 경우가 발생한다. 이를 SideEffect라고 한다. 보통 멀티 스레드 환경이나 특정 이벤트 발생과 같은 일은 이러한 사이클에 담을 수 없다.

<img src="/images/android-design-pattern-6.png" width="400px">

<br/>

### 장점
- 데이터가 한 방향으로 순환한다. 이는 예측 가능하게 하고 디버깅을 쉽게 만들 수 있다.
- 개발자가 상태를 미리 정의하기 때문에 상태에 따른 충돌이 일어나지 않는다.

- ### 단점
- 다른 패턴들과 다르게 그 의미가 명확하지 않아 진입 장벽이 높은 편이다.
- 작은 변경도 Intent를 통해서 관리해야 한다.
- 복잡한 화면이나 SideEffect를 같이 처리하는 경우 파일이 많아지고, 개발이 어렵다.