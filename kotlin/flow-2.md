# **Flow의 세 가지 유형: 한눈에 보는 비교**

| 구분 | Flow **(Cold Flow)** | StateFlow **(Hot Flow)** | SharedFlow **(Hot Flow)** |
| --- | --- | --- | --- |
| **핵심 역할** | **데이터 생산의 '설계도'** | **UI 상태(State)의 '게시판'** | **일회성 이벤트(Event)의 '방송'** |
| **비유** | YouTube 영상 (재생 누를 때마다 시작) | 실시간 뉴스 속보 화면 (언제 켜도 최신 헤드라인) | 긴급 재난 문자 (그 순간에만 수신) |
| **특징** | collect될 때마다 새로 시작 | 항상 최신 값 1개를 가짐 (상태) | 값 없이 시작 가능 (이벤트) |
| **초기값** | 필요 없음 | **필수** | 필요 없음 |
| **주요 사용처** | **Repository** (Data Layer) | **ViewModel** (UI State 노출) | **ViewModel** (UI Event 전달) |

---

# **실생활 예제: 실시간 뉴스 앱**

## **1단계: 데이터 생산자 -** Flow **(in Repository)**

Repository의 역할은 데이터를 가져오는 것입니다. '최신 뉴스 목록'을 1초마다 가져오는 가상의 API를 Flow로 구현합니다.

- **역할**: "최신 뉴스를 가져오는 방법"에 대한 설계도(Flow)를 정의합니다. ViewModel이 이 설계도를 달라고 요청하면 그때서야 실제 작업을 시작합니다.

```kotlin
// Data Layer: NewsRepository.kt

class NewsRepository @Inject constructor(private val newsApi: NewsApi) {

    /**
     * 1초마다 최신 뉴스 목록을 가져오는 '설계도(Flow)'를 반환합니다.
     * 이 함수는 호출 즉시 아무것도 하지 않습니다.
     * 누군가 이 Flow를 collect() 해야만 내부 코드가 실행됩니다. (Cold)
     */
    fun getLatestNewsStream(): Flow<List<Article>> = flow {
        while (true) {
            val latestNews = newsApi.fetchLatestNews() // suspend fun (API 호출)
            emit(latestNews) // 데이터를 스트림에 방출
            delay(1000) // 1초 대기
        }
    }.flowOn(Dispatchers.IO) // 이 flow 블록은 IO 스레드에서 실행됨
}
```

## **2단계: UI 상태 관리자 -** StateFlow **(in ViewModel)**

ViewModel은 Repository로부터 데이터 스트림을 받아, UI가 소비할 수 있는 UI 상태(StateFlow)로 변환합니다.

- **역할**: UI의 현재 상태를 담는 **게시판(**StateFlow**)** 역할을 합니다. 화면 회전이 일어나도 이 게시판의 내용은 그대로 유지됩니다.

```kotlin
// UI Layer: NewsViewModel.kt

// UI가 알아야 할 모든 정보를 담는 '상태' 클래스
data class NewsUiState(
    val articles: List<Article> = emptyList(),
    val isLoading: Boolean = true,
    val errorMessage: String? = null
)

@HiltViewModel
class NewsViewModel @Inject constructor(
    private val newsRepository: NewsRepository
) : ViewModel() {

    // 외부에 노출할 UI 상태 게시판 (StateFlow)
    val uiState: StateFlow<NewsUiState> = newsRepository.getLatestNewsStream()
        .map<List<Article>, NewsUiState> { articles -> // Cold Flow를 NewsUiState로 변환
            NewsUiState(articles = articles, isLoading = false)
        }
        .onStart { emit(NewsUiState(isLoading = true)) } // 스트림 시작 시 로딩 상태 방출
        .catch { throwable -> // 에러 처리
            emit(NewsUiState(isLoading = false, errorMessage = throwable.message)) 
        }
        .stateIn(
            scope = viewModelScope,
            started = SharingStarted.WhileSubscribed(5000), // 화면이 보일 때만 구독
            initialValue = NewsUiState(isLoading = true) // 초기 상태
        )
}
```

## **3단계: 일회성 이벤트 방송 -** SharedFlow **(in ViewModel)**

사용자가 '새로고침' 버튼을 눌렀을 때, "새로고침 완료!"라는 Toast 메시지를 **단 한 번만** 보여주고 싶습니다.

- **역할**: 일회성 이벤트를 위한 방송 채널(SharedFlow)입니다. 이 채널은 이벤트가 발생한 그 순간에만 방송을 내보냅니다.

```kotlin
// UI Layer: NewsViewModel.kt (추가된 코드)

@HiltViewModel
class NewsViewModel @Inject constructor(...) : ViewModel() {
    
    // ... 기존 StateFlow 코드 ...

    // 일회성 이벤트를 위한 비공개 방송 채널 (MutableSharedFlow)
    private val _events = MutableSharedFlow<String>()
    
    // 외부에 노출할 공개 방송 채널 (SharedFlow)
    val events: SharedFlow<String> = _events.asSharedFlow()

    fun onRefreshClicked() {
        // ... 새로고침 로직 수행 ...
        
        viewModelScope.launch {
            // 이벤트가 발생하면 방송 채널에 메시지를 '송출(emit)'
            _events.emit("새로고침 완료!")
        }
    }
}
```

## **4단계: 소비자 - UI (in Composable Screen)**

이제 UI는 ViewModel의 StateFlow(상태 게시판)와 SharedFlow(이벤트 방송)를 구독하여 화면을 그립니다.

```kotlin
// UI Layer: NewsScreen.kt

@Composable
fun NewsScreen(
    viewModel: NewsViewModel = hiltViewModel()
) {
    // StateFlow를 구독하여 UI 상태를 얻음 (상태가 바뀔 때마다 리컴포지션)
    val uiState by viewModel.uiState.collectAsStateWithLifecycle()
    val context = LocalContext.current

    // SharedFlow를 구독하여 일회성 이벤트를 처리
    // LaunchedEffect는 컴포지션이 시작될 때 단 한 번 실행되는 코루틴을 만듦
    LaunchedEffect(key1 = true) {
        viewModel.events.collect { eventMessage ->
            // 이벤트가 발생하면 Toast를 띄움
            Toast.makeText(context, eventMessage, Toast.LENGTH_SHORT).show()
        }
    }

    // 얻어온 uiState를 기반으로 화면을 그림
    Column {
        Button(onClick = { viewModel.onRefreshClicked() }) {
            Text("새로고침")
        }
        
        if (uiState.isLoading) {
            CircularProgressIndicator()
        } else if (uiState.errorMessage != null) {
            Text("에러: ${uiState.errorMessage}")
        } else {
            LazyColumn {
                items(uiState.articles) { article ->
                    Text(text = article.title)
                }
            }
        }
    }
}
```

## **정리: 공통점과 차이점 심화**

- **공통점**:
    - 세 가지 모두 **코루틴 기반의 비동기 데이터 스트림**입니다.
    - map, filter, catch 등 동일한 중간 연산자를 대부분 공유합니다.
- **결정적 차이점**:
    - **Cold vs. Hot**: Flow는 구독자가 생길 때마다 1:1로 새로운 데이터 스트림을 생성하는 **Cold Flow**입니다. StateFlow와 SharedFlow는 하나의 데이터 스트림을 모든 구독자가 공유하는 **Hot Flow**입니다.
    - **상태(State) vs. 이벤트(Event)**: StateFlow는 '현재 상태'를 나타내는 데 특화되어 있습니다. 항상 값을 가지며, 새로운 구독자에게 최신 값을 즉시 보내줍니다. SharedFlow는 '일회성 사건'을 전달하는 데 특화되어 있습니다. 값을 가지지 않을 수 있으며, 기본적으로 새로운 구독자에게 과거의 이벤트를 보내주지 않습니다 (replay = 0). 이것이 화면 회전 후에도 Toast가 다시 뜨지 않는 이유입니다.