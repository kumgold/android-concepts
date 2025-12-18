# Kotlin Flow

## 1. Flow, 왜 필요한가?
- 애플리케이션 개발에서 우리는 종종 데이터 스트림(Data Stream), 즉 시간이 지남에 따라 순차적으로 발생하는 데이터의 흐름을 다뤄야 합니다.
suspend함수가 단 한 번의 비동기 결과를 반환하는 데 최적화되어 있다면, Flow는 여러 개의 비동기 결과를 순차적으로 반환하는 데 사용됩니다.
- 과거 안드로이드에서는 RxJava가 데이터 스트림을 다루는 표준 라이브러리였습니다. 
- Flow는 코루틴을 기반으로 하여, RxJava의 강력한 기능들을 더 가볍고, 더 간결하며, 코틀린 언어에 더 자연스러운 방식으로 제공하기 위해 탄생했습니다.

## 2. Flow의 기본 구조: 생산자, 중개자, 소비자
Flow는 세 가지 구성 요소로 이루어집니다.
- <b>생산자 (Producer)</b>: flow { ... } 빌더 내에서 emit() 함수를 통해 데이터를 생성하고 방출합니다. 코루틴 기반이므로 이 과정에서 비동기 작업(네트워크 요청 등)을 안전하게 수행할 수 있습니다.
- <b>중개자 (Intermediary)</b>: map, filter, transform과 같은 연산자를 통해 방출되는 데이터를 가공하거나 변형합니다.
- <b>소비자 (Consumer)</b>: collect()와 같은 종단 연산자를 호출하여 데이터 스트림을 구독하고, 방출되는 값을 소비합니다.

<img src="/images/kotlin-flow-1-1.png"/>

## 3. Cold Flow vs. Hot Flow
이 부분은 Flow를 이해하는 가장 중요한 개념 중 하나이며, 조금 더 직관적인 비유를 추가해 보겠습니다.

- <b>Cold Flow</b>: YouTube 영상과 같습니다. 
  - <b>특징</b>: Collector가 재생 버튼(collect)을 누를 때마다, 영상(데이터 스트림)은 처음부터 새로 시작됩니다.
  - <b>관계</b>: 생산자와 소비자는 1:1 관계입니다. 각 소비자는 자신만의 독립적인 데이터 스트림을 가집니다. 
  - <b>예시</b>: flow { ... } 빌더, asFlow()로 생성된 모든 기본 Flow.
- <b>Hot Flow</b>: 실시간 TV 생방송(Live Stream)과 같습니다.
  - <b>특징</b>: Collector는 방송국(생산자)이 현재 송출하고 있는 데이터 스트림의 중간부터 참여합니다. 새로운 시청자가 들어온다고 해서 방송이 처음부터 다시 시작되지는 않습니다. 
  - <b>관계</b>: 생산자와 소비자는 1:N 관계입니다. 모든 소비자는 동일한 데이터 스트림을 공유합니다. 
  - <b>예시</b>: StateFlow, SharedFlow.

## 4. Flow의 핵심 원칙 2가지

### 1. Context Preservation (컨텍스트 보존)
Flow는 기본적으로 소비자(Collector)의 코루틴 컨텍스트(특히 Dispatcher)에서 실행됩니다. 이는 flow { ... } 빌더 내에서 UI를 직접 수정하는 코드를 넣으면 안 되는 이유입니다. 
하지만 생산자가 특정 스레드에서 동작해야 할 때가 있습니다. (예: 네트워크 요청은 Dispatchers.IO에서) 이때 withContext를 사용하면 안 되고, 반드시 flowOn() 연산자를 사용해야 합니다.
```kotlin
// ViewModel에서
fun getSomeData(): Flow<String> {
    return flow {
        // 이 블록은 flowOn에 의해 Dispatchers.IO에서 실행됨
        val data = apiService.fetchData() 
        emit(data)
    }
    .flowOn(Dispatchers.IO) // 생산자의 컨텍스트를 IO로 변경
}
// UI에서는 이 Flow를 Main 스레드에서 안전하게 collect 할 수 있음
```

### 2. Backpressure 처리
생산자가 데이터를 방출하는 속도가 소비자가 처리하는 속도보다 빠를 때 발생하는 문제를 Backpressure라고 합니다. 
Flow는 코루틴의 suspend 메커니즘을 통해 이를 자연스럽게 해결합니다. 소비자가 데이터를 처리하는 동안 생산자의 emit() 함수가 자동으로 일시 중단(suspend)되기 때문입니다. 
개발자는 복잡한 배압 전략을 고민할 필요 없이, Flow가 알아서 속도를 조절해 줍니다.

## 5. 안드로이드 UI에서 Flow 사용하기

- <b>문제점</b>: lifecycleScope.launch로 Flow를 수집하면, 화면이 백그라운드로 전환되어도(UI가 보이지 않아도) 데이터 수집이 계속되어 배터리와 리소스를 낭비합니다.
- <b>해결책</b>: repeatOnLifecycle(Lifecycle.State.STARTED) 블록 내에서 collect를 호출하는 것이 현재 Google이 가장 권장하는 표준 방식입니다.
  - <b>동작 원리</b>: Lifecycle이 STARTED 상태 이상일 때만 collect 코루틴을 실행하고, STOPPED 상태가 되면 코루틴을 자동으로 취소(cancel)합니다. 
  사용자가 다시 화면으로 돌아와 STARTED 상태가 되면, 코루틴을 새로 시작하여 데이터 수집을 재개합니다.
  - <b>flowWithLifecycle</b>: repeatOnLifecycle을 더 간결하게 표현한 연산자일 뿐, 내부 동작 원리는 동일합니다.

## 6. Flow의 진화: UI 상태 관리를 위한 StateFlow
- <b>문제점</b>: 화면 회전과 같은 Configuration Change가 발생하면 UI가 재생성됩니다. 
repeatOnLifecycle은 이전 코루틴을 취소하고 새로운 코루틴을 시작하기 때문에, Repository의 Cold Flow가 또 다시 호출되어 불필요한 네트워크 요청이 발생합니다.
- <b>해결책</b>: StateFlow를 사용하는 것입니다. StateFlow는 UI 상태를 저장하기 위해 특별히 설계된 Hot Flow입니다. 
  - 항상 값을 가집니다 (초기값 필요). 
  - .value 프로퍼티를 통해 최신 값을 동기적으로 읽을 수 있습니다. 
  - 새로운 구독자(Collector)에게는 가장 최신의 값을 즉시 보내줍니다(Replay=1).
- <b>stateIn 연산자</b>: Cold Flow를 StateFlow로 변환하는 가장 효율적이고 권장되는 방법입니다.
  - <b>SharingStarted.WhileSubscribed(5000)</b>: 이 설정이 바로 마법입니다.
  - <b>화면 회전 시</b>: UI가 파괴되고 재생성되는 시간은 매우 짧습니다(보통 5초 이내). 
  이 시간 동안 StateFlow는 구독자가 잠시 없어져도 Upstream Flow(Repository의 Cold Flow)를 살려둡니다. 새 UI가 다시 구독하면, 기존 Flow에 재연결되어 불필요한 재요청 없이 마지막 상태를 즉시 받아볼 수 있습니다.
  - <b>백그라운드 전환 시</b>: 사용자가 홈 버튼을 눌러 앱을 떠나면 5초의 타임아웃이 지나고, 
  StateFlow는 Upstream Flow를 자동으로 취소하여 리소스를 절약합니다.

## 7. 일회성 이벤트를 위한 SharedFlow
StateFlow는 상태를 다루는 데는 완벽하지만, 이벤트를 처리하는 데는 적합하지 않습니다. 
예를 들어, "저장 완료"라는 Toast 메시지를 StateFlow에 저장하면, 화면 회전 후 새로운 UI가 다시 구독할 때 이미 처리한 Toast 메시지가 또 표시되는 문제가 발생합니다.
<br>
이러한 일회성 이벤트를 처리하기 위한 Hot Flow가 바로 SharedFlow입니다. 
SharedFlow는 StateFlow보다 더 일반적인 Hot Flow로, 초기값이 없고, 구독자에게 얼마나 많은 과거의 이벤트를 재전송할지(replay) 등을 세밀하게 설정할 수 있습니다. 
이벤트 처리가 필요할 때는 SharedFlow를 사용하는 것이 올바른 아키텍처입니다.

## 결론
Kotlin Flow는 현대 안드로이드 개발에서 비동기 데이터 스트림을 다루는 핵심적인 도구입니다. 데이터 레이어에서는 Cold Flow를 사용하여 효율적인 데이터 파이프라인을 구축하고, ViewModel에서는 stateIn 연산자를 통해 이를 StateFlow로 변환하여 UI 상태를 관리합니다. 마지막으로, UI 레이어에서는 repeatOnLifecycle을 사용하여 생명주기를 인식하는 방식으로 안전하게 상태를 수집하는 것이 가장 견고하고 권장되는 아키텍처입니다. 일회성 이벤트 처리가 필요할 때는 SharedFlow를 추가로 활용할 수 있습니다.