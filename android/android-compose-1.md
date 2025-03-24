# Understanding Android Compose

## Overview
새로운 안드로이드 UI Toolkit Jetpack Compose에 대해서 이해를 돕고자 한다. 이 글은 Compose 디자인 방식의 이유와 Compose에서 작성되는 코드들, API 형성과 같은 이야기를 다루는 게시글을 번역한 글이다.

## Compose의 해결 과제
관심사의 분리는 잘 알려진 소프트웨어 개발 원칙 중 하나이다. 안드로이드 개발자로써 자주 듣는 이야기지만, 실제로 잘 이루어지는이 알아보는 것은 어려운 일이기도 하다. 이 원리가 잘 적용 되었는지 판단하는 가장 쉬운 조건은 ‘결합도’와 ‘응집력’이다.  
서비스를 개발할 때 여러 단위로 구성된 모듈을 만든다. 결합력은 서로 다른 모듈에서 객체 사이의 종속성이며, 어떤 모듈이 다른 모듈 코드에 영향을 끼치는 정도를 나타낸다. 응집력은 모듈 내의 객체들의 관계이며, 모듈 내의 객체들이 얼마나 잘 그룹화되어 있는지를 나타낸다.  
유지 관리가 용이한 서비스는 응집력(Cohesion)은 높고 결합력(Coupling)은 최소화한 소프트웨어를 의미한다.

<img src="/images/android-compose-1_1.png">

결합력이 높다는 것은 하나의 모듈에서 코드를 수정 했을 떄, 다른 모듈에 영향을 미친다는 것을 의미한다. 결합이 암묵적일수록 관련 없어 보이는 코드에 영향을 미치며 예상하지 못 한 문제점들이 생길 수 있다.  
관심사의 분리는 앱이 성장함에 따라 코드가 쉽게 유지되고 확장될 수 있도록 서로 관련된 코드를 최대한 그룹화 하는 것을 의미한다.  

<br>

예를 들어, ViewModel과 XML 레이아웃이 있다.

<img src="/images/android-compose-1_2.png">

ViewModel은 레이아웃에 많은 데이터를 제공한다. layout은 Listener를 활용하면서 ViewModel에 접근할 수 있다. 그렇기 때문에 둘 사이에 수 많은 의존성이 존재할 수 있다. 대부분의 앱은 동적으로 사용자와 상호작용하기 때문에 둘 사이의 종속성이 레이아웃 XML에 의해 정적으로 동작하는지 혹은, 프로그램 수명 주기 안에서 제대로 동작하는지와 같은 것들을 계속 확인해야 한다. 그렇지 않으면 NullPointerException이 발생할 수 있기 때문이다. (findViewById(), viewBinding에서 이런 경우를 종종 볼 수 있다.)  
ViewModel은 Kotlin으로 작성되지만, 레이아웃은 XML로 작성되어 명확한 분리선이 존재한다. 그러나 이 둘은 아주 긴밀하게 연결되어 있다.

<br><br>

만약 이 둘을 같은 Kotlin 언어로 개발한다면 어떨까? 그렇게 되면 같은 언어로 작성되기 때문에 이전에 문제가 됐던 암시적인 종속성(findViewById(), viewBinding)으로 인한 문제가 사라지고 마찬가지로 작성되었던 보일러 플레이트 코드를 줄일 수 있을 것이다. 이러한 작업이 분리선을 제거하여 관심사의 분리를 훼손한다고 보여질 수도 있지만, 앱을 개발하면서 UI 관련된 로직은 무조건 존재할 수 밖에 없다. 이걸 프레임워크에서 바꿀 수는 없다. 다만, 이러한 로직을 조금 더 잘 분리하기 위해서 프레임워크 차원에서 도구를 제공할 수 있으며 이러한 도구가 Composable 함수이다. 지금까지 Kotlin으로 개발하면서 다양한 로직을 함수로 나누어 개발했을 개발자들이 이제 UI도 함수로 나누어서 개발하는 도구가 Composable 함수이다. 이 도구를 활용하여 더 깨끗한 코드를 작성하여 리팩토링을 용이하게 할 수 있다.

## Composable function
```
@Composable
fun App(appData: AppData) {
  val derivedData = compute(cappData)
  Header()
  if (appData.isOwner) {
    EditButton()
  }
  Body {
    for (item in derivedData.items) {
      Item(item)
    }
  }
}
```
위 예제에서 Composable 함수는 불변 함수이며, 오직 AppData 객체에 의해서 값을 변경한다. 함수 f(x)와 같이 x값이 같으면 항상 같은 값이 나오는 멱등성을 갖는다. 이러한 특징은 데이터의 일관성과 안정성을 보장한다. 또한 Kotlin 언어 사용으로 인하여 Composable 함수 계층 안에 if문, for문과 같은 언어적 특징을 활용하여 계층 구조를 표현할 수 있다는 장점이 있다.

## 선언형 UI
선언형 프로그래밍은 현시점 유행하는 패러다임이다. 선언형 프로그래밍과 반대되는 명령형 프로그래밍과 비교하여 이야기하는데 예를 들어 보자.

<img src="/images/android-compose-1_3.png">

이메일 앱을 만든다고 상상했을 때, 메시지가 없으면 빈 봉투, 메시지가 있으면 편지지, 메시지가 100개 이상이면 불 타는 모습을 렌더링한다. 명령형 프로그래밍으로 이런 앱을 구현하려면 최악의 경우 아래 코드와 같은 함수로 구현해야 할 수도 있다.  
```
fun updateCount(count: Int) {
  if (count > 0 && !hasBadge()) {
    addBadge()
  } else if (count == 0 && hasBadge()) {
    removeBadge()
  }
  if (count > 99 && !hasFire()) {
    addFire()
    setBadgeText("99+")
  } else if (count <= 99 && hasFire()) {
    removeFire()
  }
  if (count > 0 && !hasPaper()) {
   addPaper()
  } else if (count == 0 && hasPaper()) {
   removePaper()
  }
  if (count <= 99) {
    setBadgeText("$count")
  }
}
```
<br><br>

반면에 선언형으로 구현했을 땐 아래와 같이 간단하게 구현할 수 있다.
```
@Composable
fun BadgedEnvelope(count: Int) {
  Envelope(fire=count > 99, paper=count > 0) {
    if (count > 0) {
      Badge(text="$count")
    }
  }
}
```
이게 선언형의 특징이다. 해당 상태에 대한 코드를 기술하지만, 어떻게 전환되어야 하는지에 대한 기술은 하지 않는다. 즉, 이전 상태에 대해서는 고려할 필요가 없고 현재 상태에 대해서만 정의를 하면 된다. 프레임워크가 현재 상태에서 다른 상태로 이동하는 것을 제어하기 때문에 더 이상 여기에 대해서 생각할 필요가 없어진다.

## Composition vs Inheritance

소프트웨어 개발에서 Composition은 단순한 코드의 여러 Unit이 모여서 더 복잡한 코드 Unit을 구현하는 방법이다. 객체지향 프로그래밍에서 가장 일반적인 형태의 Composition은 상속이다. 반면에, Jetpack Compose에서는 클래스가 아닌 함수만을 가지고 구현하는데, 이러한 방법에도 몇 가지 장점이 있다.  

<br>

예를 들어, 여러가지 Input 값을 추가한다고 가정한다면, 상속을 통해 구현한 코드는 아래 코드 조각처럼 보인다.
```
class Input : View() { /* ... */ }
class ValidatedInput : Input() { /* ... */ }
class DateInput : ValidatedInput() { /* ... */ }
class DateRangeInput : ??? { /* ... */ }
```
View는 기본 클래스이며, Input()은 여러 하위 클래스들을 갖고 있다. 만약 DateInput이 날짜의 유효성을 검사하는 클래스일 때, 시작 날짜와 종료 날짜의 유효성을 검사하는 클래스가 필요하다면 DateInput을 두 번 상속할 수 없기 때문에 또 다른 Input 하위 클래스가 필요하게 된다.  

<br><br>

반면에, Composable 함수로 구현하면 상속과는 조금 다른 모양이 된다.
```
@Composable
fun <T> Input(value: T, onChange: (T) -> Unit) { 
  /* ... */
}
```
Composable 함수로 기본 Input을 구현하고 유효성 검사를 위해 기본 Input을 다른 Composable 함수로 감싸서 Input 함수를 꾸밀 수 있다.
```
@Composable
fun ValidatedInput(value: T, onChange: (T) -> Unit, isValid: Boolean) { 
  InputDecoration(color=if(isValid) blue else red) {
    Input(value, onChange)
  }
}
```
첫 번째 예제처럼 날짜의 유효성을 검사하는 DateInput을 Composable 함수로 구현한다면, 아래 코드 조각과 같은 모양으로 구현할 수 있다.
```
@Composable
fun DateInput(value: DateTime, onChange: (DateTime) -> Unit) { 
  ValidatedInput(
    value,
    onChange = { ... onChange(...) },
    isValid = isValidDate(value)
  )
}
```

이제 시작 날짜와 종료 날짜의 유효성을 검증하는 Composable 함수를 쉽게 구현할 수 있게 되었다. 상속은 두 번 상속하는 것이 불가능하지만 함수의 호출은 두 번 호출하는 것이 가능하기 때문에 다음 코드 조각처럼 예상 상황을 쉽게 해결할 수 있다.
```
@Composable
fun DateRangeInput(value: DateRange, onChange: (DateRange) -> Unit) { 
  DateInput(value=value.start, ...)
  DateInput(value=value.end, ...)
}
```
결론적으로 Compose는 상속을 사용하지 않기 때문에 상속으로 구현할 때 어려운 상황을 쉽게 해결할 수 있다.

<br>

기존 Inheritance의 또 다른 문제는 Decoration을 추상화하는 과정에서 발생할 수 있다.
```
class FancyBox : View() { /* ... */ }
class Story : View() { /* ... */ }
class EditForm : FormView() { /* ... */ }
class FancyStory : ??? { /* ... */ }
class FancyEditForm : ??? { /* ... */ }
```
이를테면, FancyBox와 같이 다른 View를 꾸며 주는 View가 있다고 가정했을 때(EditForm이라는 View 또한 존재한다.), FancyStory, FancyEditForm은 어떤 클래스를 상속 받아야 하는지 불분명 하다.  

<br>

이와 반대로, Compose는 이런 문제를 쉽게 처리할 수 있다.
```
@Composable
fun FancyBox(children: @Composable () -> Unit) {
  Box(fancy) { children() }
}
@Composable fun Story(…) { /* ... */ }
@Composable fun EditForm(...) { /* ... */ }
@Composable fun FancyStory(...) {
  FancyBox { Story(…) }
}
@Composable fun FancyEditForm(...) {
  FancyBox { EditForm(...) }
}
```
Composable 함수는 람다 형식 매개변수로 children 객체를 삽입할 수 있기 때문에 children 객체를 또 다른 Composable 함수로 감싸줌으로써 위와 같은 문제를 쉽게 해결할 수 있다. Compose는 이런 방식으로 설계되었다.

## Recomposition
Recomposition은 Composable 함수 다시 호출하여 업데이트 한다는 의미이다. Recomposition은 Composable 함수의 계층 구조를 따라서 업데이트가 필요한 부분만 재호출하는 방식으로 진행된다.
```
fun bind(liveMsgs: LiveData<MessageData>) {
  liveMsgs.observe(this) { msgs ->
    updateBody(msgs)
  }
}
```
LiveData를 이용한 개발 방법은 LiveData가 업데이트 되는 경우 람다가 호출되고 람다 블록 안에 함수가 호출됐다.

<br>

```
@Composable
fun Messages(liveMsgs: LiveData<MessageData>) {
 val msgs by liveMsgs.observeAsState()
 for (msg in msgs) {
   Message(msg)
 }
}
```
State와 함께 Compose를 사용한 개발 방법에서는 LiveData를 obsereAsState() 함수를 통해서 State와 매핑시킨다. 매핑된 State는 LiveData가 변경될 때마다 업데이트 되고, 해당 State와 연결된 Composable 함수에 Recomposition이 발생하여 항상 최신 값을 유지할 수 있다.

<br><br>

참고 자료 : <br>
https://medium.com/androiddevelopers/understanding-jetpack-compose-part-1-of-2-ca316fe39050