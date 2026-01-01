# Scope Function

## Scope Function의 핵심 개념
Scope Function은 특정 객체의 컨텍스트(Context) 내에서 코드 블록을 실행하기 위한 함수입니다. 이 블록 안에서는 객체의 이름(변수명)을 반복해서 쓰지 않고도 그 객체의 속성이나 함수에 접근할 수 있습니다.

## 분류 기준: 2x2 매트릭스
이 5가지 함수는 딱 두 가지 기준만 기억하면 완벽하게 구분할 수 있습니다.

1. 수신 객체를 어떻게 가리키는가?
   - this: 람다 수신 객체 (Lambda Receiver). 객체의 멤버에 바로 접근 가능. 
   - it: 람다 인자 (Lambda Argument). it을 통해 접근하거나 이름을 바꿀 수 있음.

2. 무엇을 반환하는가?
   - 객체 자신 (Context Object): 체이닝하여 계속 객체를 조작할 때 유용.
   - 람다의 결과 (Lambda Result): 블록의 마지막 줄이 반환값이 됨.

|구분|반환값: 객체 자신|반환값: 람다 결과|
|---|---|---|
|참조: this|apply|run, with|
|참조: it|also|let|

## 각 함수의 특징과 사용 시나리오 (Best Practice)

### apply (this, 객체 자신)
- 느낌: "이 객체에다가(apply) 이것저것 설정을 적용해 줘."
- 사용처:객체 초기화. 객체를 생성하면서 프로퍼티를 설정할 때 가장 많이 씁니다.
```kotlin
val intent = Intent().apply {
    action = Intent.ACTION_VIEW // this.action 에서 this 생략
    data = Uri.parse("http://www.google.com")
}
```

### also (it, 객체 자신)
- 느낌: "이 객체로 작업하던 흐름은 유지하되,그리고 또한(also)이 작업도 좀 해 줘."
- 사용처: 부수 효과(Side-effect). 데이터의 유효성을 검사하거나, 디버깅 로그를 찍을 때 주로 씁니다. 객체의 속성을 변경하지 않고 그대로 넘길 때 좋습니다.
```kotlin
val numbers = mutableListOf("one", "two", "three")
numbers
    .also { println("새 항목 추가 전 리스트: $it") } // 로그 찍기
    .add("four")
```

### let (it, 람다 결과)
- 느낌: "이 객체를 ~하게(let)해 줘."
- 사용처:
  - Null 체크 후 작업 수행: ?.let { ... }패턴은 코틀린의 기본 패턴입니다. 
  - 변환 후 지역 변수 범위 제한: 객체를 다른 값으로 변환하고, 그 변환된 값을 특정 블록 안에서만 쓰고 싶을 때.
```kotlin
val str: String? = "Hello"
// str이 null이 아닐 때만 실행 (Safe Call)
str?.let {
    println("길이: ${it.length}")
} 
```

### run (this, 람다 결과)
- 느낌: "이 객체의 컨텍스트 안에서 그냥 실행(run)해 줘."
- 사용처:
  - 객체 초기화와 동시에 계산 결과가 필요할 때: apply와 비슷하지만, 초기화 후 객체 자체가 아니라 어떤 계산된 값이 필요할 때 씁니다.
  - 비확장 함수 run: 객체 없이 그냥 run { ... } 형태로 쓰면, 익명 함수처럼 블록 내의 코드를 실행하고 결과를 반환합니다.
```kotlin
val serviceResult = service.run {
    port = 8080
    start() // this.start()
    // 마지막 줄이 반환됨 (Boolean 등)
    isConnected 
}
```

### with (this, 람다 결과)
- 느낌: "이 객체와 함께(with) 작업을 수행해."
- 특징: 유일하게 확장 함수가 아닙니다. obj.with { }가 아니라 with(obj) { } 형태로 씁니다.
- 사용처: 반환값이 필요 없을 때, 객체의 여러 함수를 연속적으로 호출할 때 그룹화 용도로 씁니다. run과 거의 비슷하지만, "이 객체로 작업을 수행한다"는 뉘앙스가 더 강합니다.
```kotlin
with(binding) {
    nameTextView.text = "Kim"
    ageTextView.text = "20"
}
```

## 결론
Scope Function은 객체의 컨텍스트 내에서 코드를 실행하는 함수들입니다. 저는 주로 수신 객체 접근 방식(this vs it)과 반환값을 기준으로 상황에 맞춰 선택합니다.
- 객체 생성 및 초기화에는 this로 접근하고 객체 자신을 반환하는 apply를 사용합니다.
- Null 체크나 결과 변환이 필요할 때는 it으로 접근하는 let을 주로 사용합니다.
- 로깅 같은 부수 작업은 흐름을 끊지 않는 also를 사용합니다.
- 객체의 여러 속성을 연속해서 사용하거나 계산할 때는 run이나 with를 사용합니다.

이렇게 구분해서 사용하면 코드의 가독성을 높이고 의도를 명확하게 전달할 수 있습니다.
