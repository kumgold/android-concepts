# Flow와 LiveData의 차이점

## LiveData
관찰 가능한 데이터 홀더 클래스(Data holder class)이다. 
데이터의 변화를 관찰하고 데이터가 변경되면 관찰자에게 알리도록 설계되었다. 관찰 가능한 일반 클래스와 다르게 수명 주기를 인식한다.
Activity, Fragment, Service 등 앱 구성요소의 수명 주기를 인식할 수 있고, 활성 상태인 앱 구성 요소 관찰자만 업데이트 한다.


## Kotlin Flow
비동기식으로 계산할 수 있는 데이터 스트림이다. 여러 값을 순차적으로 내보낼 수 있으며, 비동기적으로 생성하고 사용한다.
기본적으로 스레드에 안전하게 설계되어 있으며, Android가 아닌 Kotlin Library이다.

## 비교
| 항목      | LiveData                        | Flow                                   |
|---------|---------------------------------|----------------------------------------|
| 기본 개념   | 데이터 홀더 (Data holder)            | 비동기 데이터 스트림 (Asynchronous Data stream) |
| 생명주기 인식 | 자동으로 인식 (내장 기능)                 | 수동으로 처리 필요                             |
| 스레딩     | 기본적으로 메인 스레드에서 동작. 스레드 전환 번거로움. | flowOn() 연산자로 쉽게 스레드 전환 가능.            |
| 연산자     | 제한적 (map, switchMap 등)          | 풍부함 (map, filter, zip, combine 등)      |
| 종속성     | Android Architecture Components | Kotlin Coroutines                      |
| 주 사용처   | UI Layer                        | 모든 Layer                               |
| 에러 핸들링  | 내장된 기능은 없다.                     | retry(), catch()와 같은 강력한 도구를 제공한다.     |

