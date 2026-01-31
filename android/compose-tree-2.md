# Compose Tree - 2

Compose가 화면을 그리는 과정은 마치 **건축**과 비슷합니다. 설계도(코드)가 있고, 실제로 건물을 올리는 시공(Layout), 그리고 건물의 용도를 설명하는 안내판(Semantics)이 유기적으로 연결되어 돌아갑니다.

이 세 가지 트리가 어떻게 **데이터를 주고받으며 최종적으로 사용자에게 도달하는지**, 그 유기적인 결합 흐름(Flow)을 중심으로 5단계로 나눌 수 있습니다.

---

## 1. 시작점: "코드에서 설계도까지" (Composition Tree)

**역할: 두뇌 (The Brain)**

모든 것은 개발자가 작성한 @Composable 함수에서 시작됩니다.

1. **실행 (Execution):** 런타임이 코드를 한 줄씩 실행합니다.
2. **기록 (Slot Table):** 실행된 함수, 매개변수, 내부 상태(State) 등을 메모리 상의 선형 배열인 **Slot Table**에 기록합니다. 이것이 바로 **Composition Tree**입니다.
3. **비교 (Diffing):** 만약 상태가 변해서 리컴포지션이 일어나면, 런타임은 이전 Slot Table과 현재 실행 결과를 비교합니다.
    - *"어? 텍스트가 '안녕'에서 '반가워'로 바뀌었네?"*
    - *"어? if문이 false가 돼서 Icon이 사라져야 하네?"*

**[결합 포인트]:** 이 변경 사항은 Applier라는 객체에게 전달됩니다. Applier는 설계도의 변경 사항을 실제 건물(Layout Tree)에 반영하는 현장 소장님입니다.

---

## 2. 형상화: "설계도를 보고 뼈대 세우기" (Layout Tree)

**역할: 신체 (The Body)**

Applier는 Composition Tree의 정보를 바탕으로 실제 화면을 구성할 객체인 LayoutNode를 조작하여 **Layout Tree**를 만듭니다.

1. **노드 생성:** 코드에 Box, Text 같은 UI 컴포저블이 있다면, 이에 대응하는 LayoutNode가 생성됩니다.
2. **트리 구성:** Box 안에 Text가 있다면, LayoutNode(Box)의 자식으로 LayoutNode(Text)가 붙습니다.
3. **Modifier 부착:** 여기서 **Modifier**가 결정적인 역할을 합니다.
    - Modifier.padding(10.dp) 같은 코드는 LayoutNode를 감싸는(Wrap) 형태로 적용됩니다.
    - 이 Modifier들은 LayoutNode가 자신을 측정(Measure)하고 배치(Place)하는 로직을 변형시킵니다.

**[결합 포인트]:** 이 Layout Tree는 안드로이드의 Canvas와 직접 연결됩니다. 하지만 그리기 전에 "의미"를 부여하는 과정이 병렬로 일어납니다.

---

## 3. 의미 부여: "뼈대에 설명서 붙이기" (Semantics Tree)

**역할: 목소리 (The Voice)**

Layout Tree가 만들어지는 동안, 동시에 **Semantics Tree**도 구성됩니다. 별도로 뚝딱 만들어지는 게 아니라, **Layout Tree에 기생하는 형태**로 존재합니다.

1. **데이터 추출:** LayoutNode들 중에서 Modifier.semantics나 클릭 이벤트(clickable)를 가진 노드들을 식별합니다.
2. **병합 (Merging):** 사용자에게는 버튼 내부의 Icon과 Text가 별개가 아니라 하나의 '버튼'입니다.
    - Compose는 `mergeDescendants = true` 속성을 가진 노드(예: Button)를 기준으로 하위 LayoutNode들의 의미 정보를 끌어올려 합칩니다.
3. **트리 생성:** 이렇게 병합되고 정제된 정보들만 모여서, 실제 UI 구조(Layout Tree)보다 훨씬 단순한 **Semantics Tree**가 완성됩니다.

**[결합 포인트]:** 이 트리는 화면을 그리는 데는 쓰이지 않지만, 접근성 서비스(TalkBack)나 **테스트 프레임워크**가 대기하고 있다가 이 정보를 가져갑니다.

---

## 4. 렌더링 파이프라인: "최종 산출물 만들기" (The Flow)

이제 세 트리가 준비되었으니, 한 프레임(Frame)을 그리기 위해 데이터가 어떻게 흐르는지 봅시다. 이 과정이 **Compose의 3단계**입니다.

1. **Composition 단계 (By Composition Tree):**
    - "무엇을 그릴까?"
    - 상태(State)가 바뀐 곳을 찾아내고, LayoutNode의 속성(색상, 텍스트 등)을 업데이트합니다.
2. **Layout 단계 (By Layout Tree):**
    - "어디에 그릴까?"
    - **측정(Measure):** 트리의 루트에서 말단으로 내려가며 "너 크기 얼마 필요해?"라고 묻고, 말단에서 다시 올라오며 크기가 결정됩니다.
    - **배치(Place):** 부모 노드가 자식 노드의 x, y 좌표를 지정합니다.
3. **Drawing 단계 (By Layout Tree):**
    - "어떻게 그릴까?"
    - 각 LayoutNode는 캔버스에 픽셀을 찍습니다.

**[최종 결과]:** 사용자의 눈에 UI가 보입니다.

---

## 5. 상호작용의 순환: "사용자가 화면을 터치하면?"

가장 중요한 **유기적 결합**은 사용자가 반응할 때 일어납니다.

1. **터치 발생 (Hit Test):**
    - 사용자가 화면을 탭합니다.
    - **Layout Tree**가 터치된 좌표에 있는 LayoutNode를 찾습니다.
2. **의미 전달 (Semantics):**
    - 해당 노드에 연결된 클릭 리스너(clickable)를 실행합니다. (이 정보는 Semantics Tree/Modifier에서 왔습니다.)
3. **상태 변경 (State Change):**
    - 클릭 이벤트로 인해 `count.value += 1` 같은 코드가 실행됩니다.
4. **스냅샷 감지 (Snapshot System):**
    - Compose의 스냅샷 시스템이 "어? count 값이 바뀌었네?"라고 감지합니다.
5. **재시작 (Recomposition):**
    - 이 신호가 다시 1번(Composition Tree)으로 전달됩니다.
    - Composition Tree는 변경된 부분만 다시 계산하고, Applier를 통해 **Layout Tree**를 수정하고, **Semantics Tree**도 텍스트가 바뀌었으니 업데이트됩니다.
    - 화면이 갱신됩니다.

---

## 요약: 하나의 유기체

이 세 가지 트리는 따로 노는 것이 아니라 **데이터의 변환 과정**입니다.

1. **Input:** 개발자의 코드 & 상태(State)

   ⬇️ 변환

2. **Composition Tree:** 논리적 구조와 데이터 (설계도)

   ⬇️ Applier가 반영

3. **Layout Tree:** 물리적 구조와 배치 (건물 뼈대)

   ⬇️ ↘️

4. **Drawing:** 픽셀 그리기 (시각) ➕ **Semantics Tree:** 의미 부여 (청각/기계)

   ⬇️

5. **Output:** 사용자 경험 (UX)

결국 **"상태(State)를 중심으로 세 개의 트리가 동기화되어 움직이는 것"**, 이것이 Compose가 화면을 그리는 본질적인 원리입니다.