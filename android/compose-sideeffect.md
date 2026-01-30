# Compose 생명주기와 SideEffect

## 1. Composable의 생명주기 (The Lifecycle)
기존 안드로이드의 Activity/Fragment 생명주기(onCreate, onResume, onDestroy 등)가 있었습니다. 
Compose의 생명주기는 훨씬 단순하지만, **Recomposition**때문에 더 빈번하게 발생합니다.

Composable 함수는 **Composition(UI 트리)** 내에서 다음 3가지 상태를 가집니다.

<img src="/images/compose-sideeffect.png">

1. **Enter the Composition (진입):**
    - Composable이 UI 트리에 처음 추가될 때입니다.
    - 이때 초기화 로직이나 최초의 API 호출이 주로 실행됩니다.
2. **Recomposition (갱신):**
    - 함수의 입력값(매개변수)이나 내부 상태(State)가 바뀌어서 다시 실행될 때입니다.
    - **중요:** 리컴포지션은 1초에 수십 번 일어날 수도 있고, 아예 건너뛸 수도 있습니다. 따라서 **이 단계에서 무거운 작업이나 로그 출력을 직접 하면 절대 안 됩니다.**
3. **Leave the Composition (이탈):**
    - Composable이 UI 트리에서 제거될 때입니다.
    - 이때 리소스 해제가 이루어져야 합니다.

---

## 2. 사이드 이펙트(Side Effect)란?
- "Composable 함수의 범위(Scope) 밖에서 발생하는 모든 상태 변경"을 말합니다.
- **나쁜 예시 (절대 금지):**

```kotlin
@Composable
fun MyScreen(viewModel: MyViewModel) {
    // [위험!] 리컴포지션 될 때마다 API 호출됨 -> 서버 폭주
    viewModel.fetchData() 
    
    // [위험!] 리컴포지션 될 때마다 로그 찍힘
    Log.d("Tag", "Rendered") 
        
    Text("Hello")
}
```

Compose는 UI를 그리는 것이 주 목적이므로, 위와 같은 작업은 **Effect API**라는 안전 장치 안에서 실행해야 합니다.

---

## 3. 주요 Effect Handlers
Compose는 상황에 맞는 다양한 도구를 제공합니다. 이를 정확히 구분해서 쓰는 것이 핵심입니다.

### A. 비동기 작업 실행: LaunchedEffect
- **역할:** Composable 내에서 **코루틴**을 실행합니다.
- **동작:**
    - **Enter:** Composable이 처음 나타날 때 블록 내부의 코루틴을 실행합니다.
    - **Recomposition:** key값이 바뀌면 기존 코루틴을 취소하고 다시 실행합니다.
    - **Leave:** Composable이 사라지면 코루틴을 자동으로 취소합니다.
- **사용 예시:** 화면 진입 시 스낵바 보여주기, 데이터 로딩.

```kotlin
LaunchedEffect(Unit) { // Unit은 "키가 변하지 않음"을 의미 -> 처음에 딱 한 번만 실행
    viewModel.loadData()
}
    
LaunchedEffect(userId) { // userId가 바뀔 때마다 다시 로드
    viewModel.loadUserData(userId)
}
```

### B. 콜백에서 코루틴 실행: rememberCoroutineScope
- **역할:** Composable 함수 밖(예: 버튼 클릭 리스너)에서 코루틴을 시작해야 할 때 CoroutineScope를 제공합니다.
- **주의:** Composable의 생명주기에 종속된 스코프입니다. 화면이 사라지면 여기서 실행한 작업도 취소됩니다.
- **사용 예시:** 버튼 클릭 시 스크롤 이동, 스낵바 표시.

```kotlin
val scope = rememberCoroutineScope()
    
Button(onClick = {
    // onClick은 일반 콜백이므로 suspend 함수를 바로 못 씀
    scope.launch {
        snackbarHostState.showSnackbar("Hello")
    }
}) { Text("Click") }
```

### C. 정리(Cleanup)가 필요할 때: DisposableEffect
- **역할:** 리스너 등록/해제와 같이 **시작과 끝이 명확한 작업**을 처리합니다.
- **동작:**
    - **Enter:** 효과가 실행됩니다.
    - **Leave (or Key change):** onDispose 블록이 실행되어 정리를 수행합니다.
- **사용 예시:** 생명주기 옵저버 등록, 센서 리스너, 브로드캐스트 리시버, 백버튼 콜백.

```kotlin
DisposableEffect(lifecycleOwner) {
    val observer = LifecycleEventObserver { _, event ->
        if (event == Lifecycle.Event.ON_START) { ... }
    }
    lifecycleOwner.lifecycle.addObserver(observer)
    
    // Composable이 사라질 때 반드시 실행됨
    onDispose {
        lifecycleOwner.lifecycle.removeObserver(observer)
    }
}
```

### D. Compose 상태 -> 비-Compose로 전달: SideEffect
- **역할:** 리컴포지션이 **성공적으로 완료된 직후**에 실행됩니다.
- **사용 예시:** Compose의 상태를 외부 시스템(예: 시스템 상태바 색상 변경, 로깅)과 동기화할 때.

```kotlin
// 리컴포지션 때마다 실행되지만, UI가 성공적으로 그려진 후에만 실행됨이 보장됨
SideEffect {
    systemUiController.setStatusBarColor(color)
}
```

---

## 4. 고급: 놓치기 쉬운 함정들
### 1) rememberUpdatedState (클로저 문제 해결사)
오래 걸리는 작업(예: 3초 후 로그 출력) 중에 참조하는 변수가 바뀌면 어떻게 될까요?

- **상황:** LaunchedEffect는 재시작되지 않길 원하는데, 내부에서 참조하는 값은 최신이어야 할 때.
- **해결:** 값은 바뀌어도 효과(Effect)를 재시작하지 않으면서 최신 값을 참조하게 해줍니다.

```kotlin
@Composable
fun LandingScreen(onTimeout: () -> Unit) {
    // onTimeout 함수는 리컴포지션 때마다 바뀔 수 있음 (새로운 인스턴스)
    // 하지만 LaunchedEffect(Unit)은 다시 돌지 않음.
    // 따라서 그냥 onTimeout()을 부르면 '과거의' 함수를 부르게 됨.
        
    val currentOnTimeout by rememberUpdatedState(onTimeout)
    
    LaunchedEffect(Unit) {
        delay(3000)
        currentOnTimeout() // 항상 최신 onTimeout 함수가 호출됨
    }
}
```

### 2) produceState (비동기 데이터 -> State 변환)
Flow, LiveData 등을 Compose의 State로 변환해주는 헬퍼입니다. 내부적으로 LaunchedEffect + mutableStateOf를 사용합니다.

```kotlin
val uiState by produceState(initialValue = Result.Loading, key1 = url) { 
	value = Result.Loading 
	val data = api.fetch(url) 
	value = Result.Success(data) 
}
```

### 3) derivedStateOf (리컴포지션 최적화)

상태가 **너무 자주 바뀔 때** 사용합니다. 입력은 자주 바뀌지만, 결과는 드물게 바뀔 때 유용합니다.

- **예시:** 스크롤 위치(scrollState.value)는 픽셀 단위로 계속 바뀌지만, "버튼을 보여줄지 말지(showButton)"는 특정 임계값 넘을 때만 바뀝니다.
```kotlin
// 나쁜 예: 스크롤 할 때마다 리컴포지션 발생
val showButton = scrollState.firstVisibleItemIndex > 0 
    
// 좋은 예: 0보다 큰지 여부가 '바뀔 때만' 리컴포지션 발생
val showButton by remember {
    derivedStateOf { scrollState.firstVisibleItemIndex > 0 }
}
```

---

## 요약: 언제 무엇을 써야 할까요?

1. **화면 켜지자마자 API 호출?** -> LaunchedEffect(Unit)
2. **검색어 바뀔 때마다 API 호출?** -> LaunchedEffect(keyword)
3. **버튼 눌렀을 때 스낵바?** -> rememberCoroutineScope + .launch
4. **리스너/옵저버 등록 & 해제?** -> DisposableEffect
5. **타이머 도는데 최신 값 참조?** -> rememberUpdatedState
6. **스크롤 위치에 따른 UI 변경 최적화?** -> derivedStateOf