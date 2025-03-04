# Android Compose Lifecycle

## Overview
Compose의 Lifecycle은 Activity, Fragment와 다르게 매우 간단하다.  
- Initial Composition : 초기 Composable UI 생성
- Re-Composition : State 변경을 감지하면 해당 Composable 함수 재호출 (State가 변경되지 않았다면 Re-Composition을 최대한 건너 뛴다.)
- Dispose : Composable 함수를 Compose Tree에서 삭제

<img src="/images/android-compose-lifecycle1.png">

## Compose phase
Composable 함수는 세 가지 단계를 거쳐 렌더링된다.
- **Composition(What to show)** : 표시할 UI이다. Compose는 Composable 함수를 실행하고 UI Description을 만든다.
- **Layout(Where to place it)** : UI를 배치할 위치이다. Layout 단계는 측정과 배치 두 단계로 구성된다. Layout Tree에 있는 각 노드의 요소 및 하위 요소를 2D 좌표로 측정하고 배치한다.
- **Drawing(How to render it)** : UI를 렌더링한다. UI 요소가 기기 화면인 캔버스에 그려진다.

<img src="/images/android-compose-lifecycle2.webp">

### Composition
Composition 단계에서 Composable 함수를 실행하여 UI를 나타내는 Tree 구조를 출력한다. Tree 구조는 Node로 구성되어 있으며,
다음 단계를 위해 필요한 모든 정보를 갖고 있다. 각 Composable 함수가 UI Tree 안에 Node와 1:1로 매핑된다. (if문 등이 포함되면 다른 트리가 그려질 수도 있다.)

<img src="/images/android-compose-lifecycle3.gif">

### Layout
Layout 단계에서 Tree의 각 요소가 Child Composable 함수를 측정하고 Composable 함수를 사용 가능한 2D 공간에 배치한다.
2D 공간에 각 Node의 크기와 위치를 결정하기 위해서 필요한 모든 정보가 포함되어 있다. 다음과 같은 3단계 알고리즘을 사용하여 Tree를 순회한다.
- 하위 요소 측정 : Node에 하위 요소가 있는 경우 측정한다.
- 자체 크기 결정 : 측정 값을 기준으로 하위 Node의 자체 크기를 결정한다.
- 하위 요소 배치 : 각 하위 Node는 Node 자체 위치를 기준으로 배치된다.

각 Node는 먼저 하위 요소의 크기를 측정하고 하위 요소 측정 값을 사용하여 자체 크기를 결정한다. 이후에 하위 요소를 배치한다.
각 Node는 한 번만 방문하며, Layout 단계가 끝나면 Node에 할당된 너비와 높이, 그려야 하는 x, y 좌표 정보가 포함된다.

<img src="/images/android-compose-lifecycle4.gif">

### Drawing
Drawing 단계에서 UI Tree에 각 Node를 화면 캔버스에 그린다. 앞 단계에서 너비, 높이, x, y 좌표 정보를 알게 되었으므로
Tree는 위에서 아래로 전위 순회하며 차례대로 Composable 함수를 화면에 그린다.

<img src="/images/android-compose-lifecycle5.gif">

<br>
참고 자료 : <br>
https://medium.com/androiddevelopers/compose-phases-7fe6630ea037 <br>
https://developer.android.com/develop/ui/compose/lifecycle?hl=ko <br>
https://developer.android.com/develop/ui/compose/phases?hl=ko <br>
