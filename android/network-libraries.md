# 네트워크 라이브러리 분석

## **개요: 3단계 추상화 계층**

이 세 라이브러리를 이해하는 가장 좋은 방법은 다음과 같은 계층 구조로 생각하는 것입니다.

- **Retrofit (상위 계층 - 애플리케이션 레이어)**: 개발자가 가장 사용하기 쉬운 **선언적(Declarative) REST 클라이언트**. API 명세를 인터페이스로 정의하면 모든 네트워킹 코드를 자동으로 생성해 줍니다.
- **OkHttp (중간 계층 - 전송 레이어)**: Retrofit의 실제 엔진. HTTP 통신에 필요한 모든 강력하고 효율적인 기능을 제공하는 **HTTP 클라이언트**.
- **HttpURLConnection (하위 계층 - 시스템 레이어)**: 안드로이드 OS에 내장된 기본적인 **HTTP 통신 API**. 가장 원시적이고 저수준의 제어를 제공합니다.

> 핵심 관계: Retrofit은 내부적으로 OkHttp를 사용하여 통신합니다. OkHttp는 HttpURLConnection보다 더 현대적이고 강력한 기능을 제공하는 사실상의 표준 엔진입니다.
>

---

## **1. 각 라이브러리 분석: 장점과 단점**

### **A. HttpURLConnection**

안드로이드 OS에 기본적으로 내장된, 가장 원시적인 HTTP 통신 클래스입니다.

- **장점**:
    1. **내장 API**: 별도의 외부 라이브러리를 추가할 필요가 없어 앱 용량이 아주 약간 줄어듭니다.
    2. **저수준 제어**: HTTP 요청의 모든 과정을 (헤더 설정, 스트림 읽기/쓰기 등) 직접 제어해야 하므로, HTTP 프로토콜의 동작 원리를 학습하는 데 도움이 됩니다.
- **단점**:
    1. **과도한 상용구 코드(Boilerplate)**: 간단한 GET 요청 하나를 보내는 데도 스트림을 열고, 읽고, 닫고, 예외 처리를 하는 등 수십 줄의 코드가 필요합니다.
    2. **수동 파싱**: 서버로부터 받은 JSON/XML 응답을 문자열로 받은 뒤, JSONObject나 Gson 같은 라이브러리를 사용해 개발자가 직접 객체로 변환해야 합니다.
    3. **기능 부족**: 요청 재시도(Retry), 응답 캐싱(Caching), 연결 풀링(Connection Pooling)과 같은 현대적인 네트워킹 기능을 기본으로 제공하지 않습니다.
    4. **동기식 기본**: 기본적으로 동기식으로 동작하므로, 개발자가 직접 스레드를 생성하여 백그라운드에서 실행하지 않으면 ANR이 발생합니다.

### **B. OkHttp**

Square사에서 개발한, 현대적인 고성능 HTTP 클라이언트 라이브러리입니다. 안드로이드 네트워킹의 사실상 표준입니다.

- **장점**:
    1. **고성능 및 효율성**:
        - **연결 풀링 (Connection Pooling)**: 한 번 맺은 TCP 연결을 재사용하여 요청 지연 시간을 줄입니다.
        - **HTTP/2 지원**: 하나의 연결로 여러 요청을 동시에 처리하여 효율을 극대화합니다.
        - **GZIP 압축**: 요청/응답 본문을 압축하여 데이터 사용량을 줄입니다.
    2. **강력한 기능**:
        - **요청/응답 인터셉터 (Interceptors)**: 모든 요청과 응답을 가로채서 헤더 추가, 로깅, 에러 처리 등 공통 로직을 쉽게 구현할 수 있습니다.
        - **응답 캐싱 (Response Caching)**: HTTP 캐시 헤더를 자동으로 처리하여 불필요한 네트워크 요청을 줄여줍니다.
        - **자동 재시도 (Automatic Retries)**: 일시적인 네트워크 문제 발생 시 자동으로 요청을 재시도합니다.
    3. **간결한 API**: HttpURLConnection에 비해 Builder 패턴을 사용하여 요청을 만드는 과정이 훨씬 간결하고 직관적입니다.
- **단점**:
    1. **수동 파싱 필요**: HttpURLConnection과 마찬가지로, 응답으로 받은 JSON 문자열을 객체로 변환하는 작업은 여전히 개발자의 몫입니다.
    2. **REST API에는 다소 번거로움**: 여전히 URL을 직접 만들고, 요청 본문을 수동으로 구성해야 하므로, 수많은 REST API 엔드포인트를 다루기에는 코드가 반복적일 수 있습니다.

## **C. Retrofit**

OkHttp를 기반으로 Square사에서 만든, 타입-세이프(Type-safe) REST 클라이언트 라이브러리입니다.

- **장점**:
    1. **선언적 API 정의**: API 명세를 **코틀린 인터페이스(Interface)**와 **어노테이션(@GET, @POST, @Body 등)**으로 정의하면, Retrofit이 모든 구현 코드를 알아서 생성해 줍니다. 코드가 매우 깔끔하고 가독성이 극도로 높아집니다.
    2. **타입-세이프 (Type-safe)**: 컴파일 시점에 API 명세(메소드, 파라미터 등)가 올바른지 검사할 수 있습니다. URL 오타나 파라미터 누락 등을 런타임이 아닌 컴파일 타임에 잡을 수 있습니다.
    3. **자동 파싱 (Converter)**: Converter (예: GsonConverterFactory, MoshiConverterFactory)를 등록하면, JSON 응답을 지정된 데이터 클래스 객체로 **자동으로 변환**해 줍니다. 개발자는 더 이상 파싱 코드를 작성할 필요가 없습니다.
    4. **유연한 통합 (Call Adapter)**: Call Adapter를 통해 코루틴의 suspend 함수나 RxJava의 Observable과 완벽하게 통합됩니다. 비동기 처리가 매우 간결해집니다.
- **단점**:
    1. **REST API에 특화**: RESTful API 통신에는 최고지만, 단순 파일 다운로드나 WebSocket 통신과 같은 비(非)-RESTful 통신에는 OkHttp를 직접 사용하는 것이 더 적합할 수 있습니다.
    2. **약간의 학습 곡선**: 어노테이션, Converter, Call Adapter 등 Retrofit만의 고유한 개념들을 초기에 학습해야 합니다.

---

### **2. 공통점과 차이점 분석**

| 구분 | **HttpURLConnection** | **OkHttp** | **Retrofit**                    |
| --- | --- | --- |---------------------------------|
| **추상화 수준** | **최하위 (Low-level)** | **중간 (Mid-level)** | **최상위 (High-level)**            |
| **핵심 역할** | OS 기본 HTTP API | 고성능 HTTP 엔진 | 선언적 REST 클라이언트                  |
| **사용 편의성** | 매우 복잡, 상용구 코드 많음 | 간결 (빌더 패턴) | 매우 간결 (인터페이스+어노테이션)             |
| **JSON 파싱** | **수동** | **수동** | **자동 (Converter)**              |
| **비동기 처리** | 수동 (직접 스레드 관리) | enqueue를 통한 콜백 지원 | **완벽 통합 (코루틴 suspend, RxJava)** |
| **주요 기능** | 기본 기능만 제공 | **연결 풀링, 캐싱, 인터셉터 등** | OkHttp의 모든 기능 + 자동 파싱           |
| **관계** | 독립적 (OS 내장) | HttpURLConnection의 대체재 | **내부적으로 OkHttp를 사용**            |

### **결론 및 선택 가이드**

- **HttpURLConnection:** 현대 안드로이드 앱 개발에서는 **거의 사용하지 않습니다.** 아주 특별한 이유(외부 라이브러리 사용 불가 등)가 없거나, HTTP의 원리를 배우기 위한 학습 목적이 아니라면 선택할 이유가 없습니다.
- **OkHttp:** **강력한 제어가 필요할 때** 사용합니다. 모든 요청/응답을 인터셉터로 조작하거나, 복잡한 캐시 전략을 수립하거나, REST API가 아닌 일반적인 HTTP 통신(대용량 파일 다운로드 등)을 할 때 최고의 선택입니다.
- **Retrofit:** **95% 이상의 현대 안드로이드 앱에서 REST API 통신을 위한 표준 선택지**입니다. 개발자는 서버 API 명세에만 집중하면 되고, 모든 복잡한 과정은 Retrofit과 OkHttp가 알아서 처리해 줍니다. 생산성, 가독성, 안정성 모든 면에서 가장 뛰어납니다.

---

# OkHttp가 대용량 다운로드에 좋은 이유

## **1. 문제점: 대용량 파일을 '나이브'하게 다루면 생기는 일**

만약 1GB 크기의 동영상 파일을 다운로드한다고 가정해 보겠습니다. 가장 순진한 방법은 서버의 응답을 모두 받은 뒤, 그 데이터를 한 번에 파일로 저장하는 것입니다.

**나쁜 예제 :**

```kotlin
// 이 코드는 OutOfMemoryError를 유발합니다!
fun downloadFileTheWrongWay(client: OkHttpClient, url: String) {
    val request = Request.Builder().url(url).build()
    val response = client.newCall(request).execute()

    if (response.isSuccessful) {
        // response.body.bytes()는 1GB 응답 전체를 RAM의 byte 배열로 읽어들입니다.
        val fileBytes = response.body?.bytes() 
        
        // 앱이 여기서 죽습니다.
        File("path/to/save/video.mp4").writeBytes(fileBytes!!) 
    }
}
```

이 코드의 response.body?.bytes() 라인은 1GB의 데이터를 **RAM(메모리)에 한 번에** 불러오려고 시도합니다. 안드로이드 앱의 힙 메모리 제한(보통 256MB ~ 512MB)을 훨씬 초과하므로, 이 코드는 100% OutOfMemoryError를 발생시키며 앱을 강제 종료시킵니다.

## **2. 해결책: OkHttp의 스트리밍(Streaming) 방식**

OkHttp가 대용량 파일 다운로드에 강력한 이유는 바로 스트리밍(Streaming)을 네이티브하게 지원하기 때문입니다.

- **스트리밍이란?**: 데이터를 '거대한 물풍선'처럼 한 번에 통째로 받는 것이 아니라, '수도꼭지에서 흐르는 물'처럼 **작은 덩어리(Chunk)로 나누어 순차적으로** 읽고 쓰는 방식입니다.
- **OkHttp의 핵심**: response.body 객체는 응답 전체를 담고 있는 데이터 덩어리가 아니라, 서버로부터 데이터가 흘러 들어오는 '파이프라인(Pipe)'에 연결된 수도꼭지(ResponseBody)와 같습니다. 우리는 이 수도꼭지에서 물을 한 컵씩 받아서(버퍼에 읽기), 양동이에 붓는(파일에 쓰기) 작업을 반복할 수 있습니다.

이 방식을 사용하면, 1GB 파일이든 10GB 파일이든 상관없이, 앱은 항상 작은 컵(버퍼) 크기만큼의 **아주 적은 메모리만 사용**하게 됩니다.

---

## **3. 예제 코드: 스트리밍을 이용한 안전한 파일 다운로드**

이제 스트리밍 방식으로 대용량 파일을 다운로드하고, 사용자에게 진행 상황까지 알려주는 실용적인 예제 코드를 보여드리겠습니다.

```kotlin
import okhttp3.OkHttpClient
import okhttp3.Request
import java.io.File
import java.io.FileOutputStream
import java.io.IOException

// UI 업데이트를 위한 간단한 콜백 인터페이스
interface DownloadCallback {
    fun onProgress(progress: Int)
    fun onSuccess()
    fun onFailure(e: IOException)
}

fun downloadLargeFile(client: OkHttpClient, url: String, outputFile: File, callback: DownloadCallback) {
    val request = Request.Builder().url(url).build()

    // OkHttp의 Call 객체는 비동기 실행을 지원합니다.
    // 실제 앱에서는 Coroutine, RxJava 등을 사용해야 합니다. 
    // 여기서는 설명을 위해 execute()를 사용하지만, 반드시 백그라운드 스레드에서 호출해야 합니다.
    try {
        val response = client.newCall(request).execute()

        if (!response.isSuccessful) {
            throw IOException("다운로드 실패: ${response.code}")
        }

        val body = response.body ?: throw IOException("응답 본문이 비어있습니다.")
        
        // 1. 전체 파일 크기를 가져옵니다 (진행률 계산용)
        val totalFileSize = body.contentLength()
        var totalBytesRead = 0L

        // 2. 스트림을 안전하게 다루기 위해 .use 블록을 사용합니다.
        //    블록이 끝나면 스트림이 자동으로 닫힙니다.
        body.byteStream().use { inputStream ->
            FileOutputStream(outputFile).use { outputStream ->
                // 3. 데이터를 읽어들일 작은 '컵'(버퍼)을 준비합니다. (예: 8KB)
                val buffer = ByteArray(8 * 1024)
                var bytesRead: Int

                // 4. inputStream에서 데이터를 buffer로 읽고, 그만큼 outputStream에 씁니다.
                //    inputStream.read()는 파일의 끝에 도달하면 -1을 반환합니다.
                while (inputStream.read(buffer).also { bytesRead = it } != -1) {
                    outputStream.write(buffer, 0, bytesRead)
                    
                    // 5. 진행률을 계산하고 UI에 알립니다.
                    totalBytesRead += bytesRead
                    val progress = (totalBytesRead * 100 / totalFileSize).toInt()
                    callback.onProgress(progress)
                }
            }
        }
        callback.onSuccess()

    } catch (e: IOException) {
        e.printStackTrace()
        callback.onFailure(e)
    }
}
```

### **이 코드가 강력한 이유 (OkHttp의 장점)**

1. **메모리 효율성 (핵심)**: body.byteStream()을 통해 데이터의 '흐름' 자체를 가져옵니다. 10GB 파일이라도 메모리에는 항상 8KB 크기의 buffer만 존재하므로 OutOfMemoryError**가 절대 발생하지 않습니다.**
2. **진행률 추적의 용이성**: response.body.contentLength()를 통해 전체 파일 크기를 쉽게 얻을 수 있습니다. 스트림을 읽는 반복문 안에서 현재까지 읽은 바이트를 누적하면, 정확한 다운로드 진행률(%)을 계산하여 프로그레스 바(Progress Bar)를 업데이트할 수 있습니다.
3. **높은 수준의 제어**: 스트림을 직접 다루기 때문에, 다운로드 중간에 사용자가 '취소' 버튼을 누르면 반복문을 중단하고 스트림을 닫는 등 **중단/재개 로직**을 유연하게 구현할 수 있습니다.
4. **Okio 라이브러리를 통한 I/O 최적화**: OkHttp는 내부적으로 **Okio**라는 고성능 I/O 라이브러리를 사용합니다. ResponseBody의 source()를 사용하면 Okio의 BufferedSource를 직접 다룰 수 있는데, 이는 일반적인 InputStream보다 훨씬 더 효율적으로 버퍼를 관리하여 I/O 성능을 극대화합니다.

결론적으로, Retrofit이 API 명세를 우아하게 정의하는 '설계자'라면, OkHttp는 대용량 데이터를 메모리 문제 없이 효율적으로 운반하는 '강력한 운송 트럭'과 같습니다. 스트리밍을 통해 메모리 사용량을 최소화하고, 진행률 추적과 같은 정교한 제어를 가능하게 해주기 때문에 대용량 파일 다운로드에 OkHttp를 직접 사용하는 것은 선택이 아닌 필수입니다.

---

**OkHttp는 넷플릭스와 같은 서비스의 동영상 스트리밍과 다운로드 기능의 심장과도 같은 필수 기술입니다.**

다만, 두 시나리오에서 OkHttp가 사용되는 방식과 추상화 레벨은 완전히 다릅니다. 이 차이점을 이해하고 설명하는 것이 핵심입니다.

---

### **핵심 전제: 하나의** OkHttpClient **인스턴스 공유**

가장 중요한 아키텍처 원칙은 앱 전체에서 단 하나의 OkHttpClient 인스턴스를 만들어 공유하는 것입니다. Hilt와 같은 DI 라이브러리를 통해 싱글턴으로 관리하는 것이 이상적입니다.

**왜 공유해야 하는가?**

- **리소스 효율성**: 모든 네트워크 요청(API, 이미지, 비디오)이 동일한 연결 풀(Connection Pool)과 캐시를 공유하여 리소스를 극도로 아낄 수 있습니다.
- **중앙 집중식 제어**: 인증 토큰을 헤더에 추가하거나, 모든 네트워크 트래픽을 로깅하는 인터셉터(Interceptor)를 단 한 곳에서 관리할 수 있습니다. 이는 유지보수성과 안정성에 결정적인 영향을 미칩니다.

```kotlin
// 예시: Hilt를 사용한 OkHttpClient 모듈
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Provides
    @Singleton
    fun provideOkHttpClient(): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor(AuthInterceptor()) // 모든 요청에 인증 헤더 추가
            .addInterceptor(HttpLoggingInterceptor().apply { level = HttpLoggingInterceptor.Level.BODY })
            .connectTimeout(30, TimeUnit.SECONDS)
            .build()
    }
}
```

---

### **시나리오 1: 동영상 스트리밍 서비스 개발**

여기서 가장 중요한 함정은 "스트리밍을 위해 OkHttp 스트림을 직접 다루지 않는다"는 것입니다. 그렇게 하는 것은 바퀴를 재발명하는 것과 같습니다.

스트리밍의 복잡성(가변 비트레이트, 프로토콜 처리, 버퍼링 관리 등)은 **ExoPlayer**와 같은 전문 미디어 플레이어 라이브러리에 위임해야 합니다. 이때, OkHttp는 ExoPlayer가 사용하는 **네트워크 엔진**으로서 막후에서 활약하게 됩니다.

**OkHttp의 역할**: ExoPlayer가 "다음 비디오 조각(Chunk) 주세요"라고 요청할 때, 우리가 설정한 OkHttpClient를 통해 효율적으로 데이터를 가져와 전달하는 역할을 합니다.

### **실제 코드: ExoPlayer와 OkHttp 연동하기**

우리는 ExoPlayer가 기본 네트워크 스택 대신, 우리가 만든 공유 OkHttpClient를 사용하도록 설정만 해주면 됩니다.

```kotlin
// PlayerViewModel.kt

@HiltViewModel
class PlayerViewModel @Inject constructor(
    @ApplicationContext private val context: Context,
    private val okHttpClient: OkHttpClient // Hilt를 통해 공유 OkHttpClient 주입
) : ViewModel() {

    val exoPlayer: ExoPlayer

    init {
        // 1. ExoPlayer가 OkHttp를 사용하도록 DataSource.Factory를 설정합니다.
        val dataSourceFactory: DataSource.Factory = OkHttpDataSource.Factory(okHttpClient)

        // 2. 이 DataSourceFactory를 사용하여 MediaSource를 만듭니다.
        //    DASH, HLS 등 스트리밍 프로토콜에 맞는 MediaSource를 선택합니다.
        val mediaSourceFactory = DefaultMediaSourceFactory(dataSourceFactory)

        // 3. 커스텀 MediaSourceFactory로 ExoPlayer를 빌드합니다.
        exoPlayer = ExoPlayer.Builder(context)
            .setMediaSourceFactory(mediaSourceFactory)
            .build()

        // 4. 이제 ExoPlayer에 MediaItem을 설정하면, 모든 네트워크 요청은
        //    우리의 공유 OkHttpClient를 통해 이루어집니다.
        val mediaItem = MediaItem.fromUri("https://your-streaming-url/manifest.mpd")
        exoPlayer.setMediaItem(mediaItem)
        exoPlayer.prepare()
    }

    // ... onPause, onResume, onCleared 등 생명주기 관리 로직 ...
}
```

이 코드 한 줄 OkHttpDataSource.Factory(okHttpClient)이 바로 마법입니다. 이제 넷플릭스의 모든 비디오 스트림 데이터는 우리가 설정한 인증 인터셉터와 로깅 인터셉터를 통과하게 됩니다.

---

### **시나리오 2: 대용량 VOD 파일 다운로드 개발**

이 시나리오에서는 사용자가 오프라인 시청을 위해 2GB짜리 영화 한 편을 다운로드합니다. 여기서는 **OkHttp의 스트리밍 I/O 능력을 직접 활용**해야 합니다.

그리고 이처럼 오래 걸리고, 반드시 실행이 보장되어야 하는 작업은 **WorkManager**에게 위임하는 것이 현대 안드로이드 개발의 표준입니다.

**OkHttp의 역할**: WorkManager의 Worker 내부에서, OutOfMemoryError 없이 대용량 파일을 작은 조각으로 나누어 디스크에 쓰는 실제 I/O 작업을 수행합니다.

### **실제 코드: WorkManager와 OkHttp를 이용한 다운로드**

DownloadWorker.kt
```kotlin
@HiltWorker
class DownloadWorker @AssistedInject constructor(
    @Assisted appContext: Context,
    @Assisted workerParams: WorkerParameters,
    private val okHttpClient: OkHttpClient // Hilt를 통해 공유 OkHttpClient 주입
) : CoroutineWorker(appContext, workerParams) {

    override suspend fun doWork(): Result {
        val videoUrl = inputData.getString("VIDEO_URL") ?: return Result.failure()
        val outputFilePath = inputData.getString("OUTPUT_PATH") ?: return Result.failure()
        val outputFile = File(outputFilePath)

        val request = Request.Builder().url(videoUrl).build()

        try {
            val response = okHttpClient.newCall(request).execute()
            if (!response.isSuccessful) return Result.failure()

            val body = response.body ?: return Result.failure()
            val totalSize = body.contentLength()
            var downloadedSize = 0L

            body.byteStream().use { input ->
                FileOutputStream(outputFile).use { output ->
                    val buffer = ByteArray(8 * 1024)
                    var bytesRead: Int
                    while (input.read(buffer).also { bytesRead = it } != -1) {
                        // isStopped는 WorkManager가 작업을 취소했는지 확인
                        if (isStopped) return Result.failure()

                        output.write(buffer, 0, bytesRead)
                        downloadedSize += bytesRead
                        
                        // WorkManager의 내장 프로그레스 업데이트 기능 사용
                        val progress = (downloadedSize * 100 / totalSize).toInt()
                        setProgress(workDataOf("PROGRESS" to progress))
                    }
                }
            }

            return Result.success()

        } catch (e: Exception) {
            // 실패 시 재시도하도록 설정 가능
            return Result.retry()
        }
    }
}
```

DownloadViewModel.kt
```kotlin
class DownloadViewModel @Inject constructor(
    private val workManager: WorkManager
) : ViewModel() {

    fun startDownload(videoUrl: String, outputPath: String) {
        val inputData = workDataOf(
            "VIDEO_URL" to videoUrl,
            "OUTPUT_PATH" to outputPath
        )

        val constraints = Constraints.Builder()
            .setRequiredNetworkType(NetworkType.UNMETERED) // Wi-Fi에서만 다운로드
            .setRequiresStorageNotLow(true) // 저장 공간 충분할 때만
            .build()

        val downloadWorkRequest = OneTimeWorkRequestBuilder<DownloadWorker>()
            .setInputData(inputData)
            .setConstraints(constraints)
            .build()
        
        workManager.enqueue(downloadWorkRequest)
        // 이제 UI에서는 이 workRequest의 ID를 가지고 LiveData를 관찰하여 진행률을 표시할 수 있음
    }
}
```

이처럼 두 시나리오에서 OkHttp는 **각기 다른 추상화 레벨에서, 각기 다른 방식으로** 넷플릭스 서비스의 핵심 기능을 구현하는 데 반드시 필요한 기반 기술임을 알 수 있습니다.