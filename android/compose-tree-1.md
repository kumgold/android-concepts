# Compose Tree - 1

**Compose Tree**는 Jetpack Compose가 작성한 코틀린 코드를 실제 사용자가 보는 화면으로 변환하기 위해 내부적으로 관리하는 **데이터 구조의 총체**입니다.

사실 Compose 내부에는 **하나가 아닌 3개의 병렬 트리**가 존재하며, 이들이 유기적으로 협력합니다. 이 구조를 이해하는 것이 성능 최적화와 커스텀 UI 제작의 핵심입니다.

---

## 1. 세 가지 트리 (The Three Trees)

Compose가 실행되면, 내부적으로 다음과 같은 세 가지 트리가 만들어집니다.

### ① Composition Tree - "무엇(What)"

- **정의:** 여러분이 작성한 @Composable 함수들의 호출 구조 그 자체입니다.
- **역할:** "어떤 컴포저블이 호출되었고, 어떤 데이터를 가지고 있는지"를 기록합니다.
- **특징:** 이 트리는 실제 UI 객체를 가지고 있지 않습니다. 대신 Slot Table이라는 메모리 구조에 정보(상태, 파라미터 등)를 저장합니다.
- **비유:** 건물의 **설계 도면**입니다.

### ② Layout Tree - "어디에(Where)"

- **정의:** UI 트리의 실제 뼈대입니다. LayoutNode라는 객체들로 구성됩니다.
- **역할:** 화면 배치(Measure & Place), 그리기(Draw), 히트 테스트(터치 감지)를 담당합니다.
- **특징:** XML 시절의 View 계층 구조와 가장 유사한 개념입니다. Composition Tree가 변경되면, 그 변경 사항이 Layout Tree에 반영됩니다.
- **비유:** 도면을 보고 실제로 지어진 **건물 뼈대**입니다.

### ③ Semantics Tree - "의미(Meaning)"

- **정의:** UI의 의미론적 정보를 담고 있는 트리입니다.
- **역할:**
    1. **접근성(Accessibility):** TalkBack 같은 스크린 리더가 이 트리를 읽어 시각 장애인에게 화면을 설명합니다.
    2. **테스트(Testing):** `composeTestRule.onNodeWithText(...)` 같은 테스트 코드가 요소를 찾을 때 이 트리를 사용합니다.
- **특징:** 모든 LayoutNode가 시맨틱 노드를 갖지는 않습니다. 의미 있는 정보만 병합(Merge)하여 관리합니다.
<details>
    <summary>Semantics Tree</summary>
Semantics Tree를 한마디로 정의하면 "화면의 '의미(Meaning)'를 설명하는 통역사"입니다. 컴퓨터나 시각 장애인에게는 화면의 픽셀 정보가 아무런 의미가 없습니다.

## 1. 비유: 그림 vs 설명

친구에게 카카오톡으로 **'저장 버튼'** 사진을 보냈다고 가정해 봅시다.

- **눈이 보이는 친구 (Layout Tree):**
    - 파란색 직사각형이 있고, 그 안에 흰색으로 "저장"이라는 글자가 써져 있네. (위치, 크기, 색상 정보)
- **앞이 안 보이는 친구 (Semantics Tree):**
    - 사진을 볼 수 없으니 설명이 필요합니다.
    - "이건 **버튼**이야. 누르면 **저장**이 돼. 지금 **활성화**되어 있어." (역할, 동작, 상태 정보)

**Semantics Tree**는 바로 이 '앞이 안 보이는 친구'를 위해 존재하는 **설명서**입니다.
    
---

## 2. 왜 별도의 트리가 필요한가요?

"그냥 Layout Tree를 그대로 쓰면 안 되나요?"라고 물을 수 있습니다. 하지만 Layout Tree는 너무 TMI(Too Much Information)입니다.

- **Layout Tree의 시선:**

> "여기 Box가 있고, 그 안에 Row가 있고, Spacer로 8dp 띄우고, Icon 그림을 그리고, 옆에 Text로 '저장'이라고 썼어."
>
- **Semantics Tree의 시선:**

> "이 덩어리 전체는 그냥 '저장 버튼'이야."
>

Layout Tree는 장식용 요소(Spacer, 배경 장식)까지 다 가지고 있지만, Semantics Tree는 **사용자와 상호작용하는 핵심 정보**만 추려서 간소화합니다. 그래서 별도의 트리가 필요합니다.
    
---

## 3. 누가 이 트리를 사용하나요? (핵심 고객)

이 설명서(Semantics Tree)를 읽는 주요 고객은 두 명입니다.

1. **접근성 서비스 (TalkBack 등):**
   - 시각 장애인이 화면을 터치하면, TalkBack은 Semantics Tree를 조회합니다.
   - "저장 버튼, 두 번 탭하여 실행"이라고 읽어줍니다.
2. **UI 테스트 프레임워크 (Compose Test Rule):**
   - 테스트 코드에서 `onNodeWithText("저장")`라고 찾을 때, 픽셀을 분석하는 게 아니라 이 Semantics Tree에서 "저장"이라는 속성을 가진 노드를 찾습니다.

---

## 4. 작동 원리: 병합(Merging) - 가장 중요한 개념

Compose에서 Semantics Tree를 이해할 때 가장 중요한 것이 "정보의 병합"입니다.

예를 들어 Button을 만들면 구조는 이렇습니다.

```kotlin
Button(onClick = { ... }) { // 부모
    Icon(...)               // 자식 1
    Spacer(...)             // 자식 2
    Text("저장")             // 자식 3
}
```

- **Layout Tree:** 부모, 자식1, 자식2, 자식3이 모두 각각 존재합니다. (총 4개 노드)
- **Semantics Tree:** **하나의 노드**로 합쳐집니다.

### **왜 합쳐질까요?**

사용자 입장에서는 아이콘을 누르든, 텍스트를 누르든 그냥 "버튼을 누른 것"입니다. 따라서 Compose는 Button같은 상위 컴포저블에 `mergeDescendants = true`라는 속성을 부여하여, 자식들의 의미(Text: "저장")를 부모(Button)로 빨아들입니다.

결과적으로 Semantics Tree에는 "텍스트가 '저장'인 버튼"이라는 하나의 노드만 남게 됩니다. TalkBack은 자잘한 아이콘이나 텍스트를 따로따로 읽지 않고, 한 번에 "저장 버튼"이라고 읽을 수 있게 됩니다.
    
---

## 5. 개발자가 직접 건드려야 할 때 (Modifier.semantics)

대부분의 기본 컴포저블(Button, TextField 등)은 이미 의미가 잘 부여되어 있습니다. 하지만 개발자가 커스텀 UI를 만들 때는 직접 의미를 부여해야 합니다.

**예시: 그냥 Box를 버튼처럼 쓰고 싶을 때**

```kotlin
Box(
    modifier = Modifier
        .clickable { /* 클릭 동작 */ }
        // Semantics 정보 추가
        .semantics {
            role = Role.Button         // "이건 버튼이야" 라고 알려줌
            contentDescription = "저장" // "이건 '저장' 기능을 해" 라고 알려줌
        }
) {
    // 내용물...
}
```

이렇게 Modifier.semantics를 사용하면 Semantics Tree에 정보를 심어줄 수 있습니다.
    
---

## 요약

1. **정의:** 화면의 '의미(역할, 텍스트, 상태, 액션)'만 요약해 놓은 트리입니다.
2. **용도:** 시각 장애인용 스크린 리더(TalkBack)와 **UI 테스트 코드**가 이 트리를 보고 화면을 파악합니다.
3. **특징:**
   - **간소화:** 장식용 요소(Spacer 등)는 무시합니다. 
   - **병합(Merging):** 버튼 내부의 아이콘+텍스트 처럼 쪼개진 요소들을 하나의 '클릭 가능한 덩어리'로 합쳐서 관리합니다.

</details>

---

## 2. 트리 생성 과정

코드가 어떻게 이 트리들로 변환되는지 '바닥' 레벨인 **Compose Runtime** 관점에서 보겠습니다.

### A. Slot Table과 Gap Buffer

Compose는 트리 구조를 저장하기 위해 객체 그래프(Linked List 등)를 쓰지 않습니다. 대신 **배열(Array)** 기반의 **Slot Table**을 사용합니다. 이것이 Compose가 빠른 이유입니다.

1. **선형 배열:** 컴포저블 함수가 실행되는 순서대로 배열에 데이터를 쭉 씁니다. (DFS: 깊이 우선 탐색 순서)
2. **Gap Buffer :** 텍스트 에디터가 커서를 이동하며 글자를 입력하는 원리와 같습니다.
    - 데이터 변경이 필요한 위치로 'Gap(빈 공간)'을 이동시킵니다.
    - 변경된 부분만 수정하고, 나머지 배열은 건드리지 않습니다.
    - 이 때문에 **리컴포지션이 일어날 때 전체 트리를 다시 만들지 않고, 변경된 부분만 O(1)에 가깝게 수정**할 수 있습니다.

### B. Applier

Slot Table(설계도)과 LayoutNode Tree(건물) 사이를 연결하는 일꾼입니다.

- **동작:** Composer가 Slot Table을 돌면서 변경 사항(Diff)을 감지하면, Applier에게 명령을 내립니다.
    - "노드 A의 텍스트 속성을 바꿔"
    - "노드 B 밑에 자식 노드 C를 추가해"
    - "노드 D를 삭제해"
- 이 과정을 통해 **Layout Tree**가 최신 상태로 유지됩니다.

---

## 3. LayoutNode의 구조와 Modifier

Layout Tree를 구성하는 **LayoutNode**를 자세히 보면 재미있는 사실이 있습니다.

### Modifier 체이닝의 비밀

우리가 흔히 쓰는 Modifier는 단순한 속성 설정이 아닙니다. LayoutNode를 감싸는 래퍼(Wrapper) 역할을 합니다.

```kotlin
Box(
    modifier = Modifier
        .padding(10.dp)   // 1번
        .background(Color.Red) // 2번
        .size(50.dp)      // 3번
)
```

이 코드는 내부적으로 LayoutNode에 **Modifier Chain**을 형성합니다.

- **실행 순서:** 레이아웃(크기 측정)은 **안에서 밖으로**, 그리기는 **밖에서 안으로** 등의 복잡한 순서를 가집니다.
- 그래서 padding을 먼저 쓰느냐 나중에 쓰느냐에 따라 결과가 완전히 달라지는 것입니다. 이는 트리의 노드 구조가 달라지기 때문입니다.

---

## 4. 렌더링 파이프라인과 트리의 상호작용

**Compose의 3단계**가 이 트리 위에서 실행됩니다.

1. **Composition 단계:**
    - Composition Tree (Slot Table)를 갱신합니다.
    - 함수를 실행하고 바뀐 데이터를 찾습니다.
2. **Layout 단계:**
    - Layout Tree를 순회합니다.
    - **Measurement:** 부모 노드가 자식에게 "너 크기 얼마면 돼?"(Constraints)라고 묻습니다.
    - **Placement:** 부모 노드가 자식의 위치(x, y)를 정해줍니다.
3. **Drawing 단계:**
    - Layout Tree의 노드들이 캔버스(Canvas)에 자신을 그립니다.

**최적화 팁:**

- 상태(State)를 읽는 시점이 1단계(Composition)라면 리컴포지션이 발생하지만, 2단계(Layout)나 3단계(Draw)에서만 읽는다면 리컴포지션을 건너뛰고 **레이아웃이나 드로잉만 다시 수행**합니다. 이것이 성능 최적화의 핵심입니다.

---

## 5. 개발자가 알아야 할 실무 포인트

이 이론이 실제 개발에 주는 교훈은 다음과 같습니다.

1. **Layout Inspector 활용:** 안드로이드 스튜디오의 Layout Inspector를 켜면 이 Layout Tree와 Semantics Tree를 눈으로 볼 수 있습니다. 디버깅의 필수 도구입니다.
2. **불필요한 래핑 줄이기:** `Box { Box { Box { ... } } }` 처럼 무의미하게 중첩하면 LayoutNode가 너무 많이 생성되어 Layout Tree가 깊어지고 성능이 떨어집니다.
3. **리컴포지션 최소화:** Slot Table의 비교 연산 비용을 줄이기 위해 key를 잘 사용하고, 가능한 한 상태 읽기를 뒤로 미루십시오(Lambda 사용 등).

---

## 요약

- **Compose Tree**는 사실 Composition(데이터/로직), Layout(화면 배치), Semantics(의미)라는 3개의 트리로 구성됩니다.
- 내부적으로는 Slot Table(배열 + Gap Buffer)이라는 효율적인 자료구조를 사용하여 변경 사항을 관리합니다.
- LayoutNode는 실제 화면을 그리는 단위이며, Modifier는 이 노드의 동작을 변형시키는 체인입니다.

이 구조를 이해하면 "왜 Modifier 순서가 중요한지", "왜 리컴포지션이 빠른지" 명확하게 알 수 있습니다.