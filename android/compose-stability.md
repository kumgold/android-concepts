# Compose Stability

## 1. @Stable의 3가지 조건

Compose가 어떤 객체를 "안정적이다"라고 믿으려면 다음 3가지 조건이 충족되어야 합니다.

1. **equals()의 일관성**: 두 인스턴스의 equals() 결과가 true라면, 두 인스턴스는 영원히 동일해야 합니다.
2. **변경 알림**: 객체의 공개 프로퍼티(public property)가 변경되면, Compose가 그 사실을 알 수 있어야 합니다. (예: MutableState 사용)
3. **재귀적 안정성**: 객체가 가지고 있는 모든 공개 프로퍼티 또한 안정적(Stable)이어야 합니다.

---

## 2. Case by Case 분석

### ① 기본 타입 (Primitives) & String

- **판정**: **@Stable (Stable)**
- **이유**: Int, Float, Boolean, String 등은 불변(Immutable)이기 때문에 값이 바뀌지 않음이 보장됩니다.

### ② 일반 Data Class (모든 필드가 val)

```kotlin
// ✅ Stable
data class User(
    val name: String, // Stable
    val age: Int      // Stable
)

```

- **판정**: **@Stable**
- **이유**: 모든 필드가 `val`이고, 각 필드의 타입(String, Int)도 Stable하기 때문입니다. Compose는 equals() 비교를 통해 값이 안 바뀌었으면 건너뜁니다.

### ③ Data Class (var 포함)

```kotlin
// ❌ Unstable
data class Counter(
    var count: Int // var는 언제든 바뀔 수 있음
)

```

- **판정**: **Unstable**
- **이유**: count가 바뀌어도 Compose는 알림을 받지 못합니다(State가 아니므로). 따라서 Compose는 "얘는 언제 바뀔지 몰라"라며 항상 다시 그립니다.

### ④ Data Class (State로 감싼 var)

```kotlin
// ✅ Stable
class Counter {
    var count by mutableStateOf(0) // State로 감쌌음
}

```

- **판정**: **@Stable**
- **이유**: `var`지만 MutableState를 사용했습니다. 값이 바뀌면 Compose에게 "나 변했어!"라고 알림을 주기 때문에 안정적인 것으로 간주합니다.

---

## 3. 가장 큰 문제: 컬렉션 (List, Set, Map)

안드로이드 개발자들이 가장 많이 실수하는 부분입니다.

### ❌ 일반 List (Interface)

```kotlin
data class Menu(
    val items: List<String> // ❌ Unstable
)

```

- **판정**: **Unstable**
- **이유**: Kotlin의 List는 **인터페이스**입니다.
    - 이 List의 구현체가 불변인 `listOf()`인지, 가변인 `ArrayList()`인지 컴파일러는 알 수 없습니다.
    - 내부 내용이 바뀌어도 Compose는 알 수 없으므로, **보수적으로 "불안정함"으로 판단**하고 무조건 리컴포지션을 수행합니다.

### ✅ 해결책 1: @Stable 어노테이션 사용 (Wrapper Class)

개발자가 컴파일러에게 "이거 내가 보장할게, 믿어줘!"라고 각서를 쓰는 방식입니다.

```kotlin
@Stable // "내가 보장할게!"
data class MenuState(
    val items: List<String>
)

@Composable
fun MenuList(state: MenuState) { ... } // items가 같으면 스킵됨

```

### ✅ 해결책 2: Kotlinx Immutable Collections (권장)

Kotlinx에서 제공하는 불변 컬렉션 라이브러리를 사용하면, 타입 자체가 불변임이 보장되므로 Stable로 인식됩니다.

```kotlin
// build.gradle에 의존성 추가 필요
// implementation "org.jetbrains.kotlinx:kotlinx-collections-immutable:x.x.x"

data class Menu(
    val items: ImmutableList<String> // ✅ Stable
)

```

---

## 4. @Immutable vs @Stable

두 어노테이션 모두 컴파일러에게 최적화를 허용하라는 신호지만, 의미가 약간 다릅니다.

- **@Immutable**:
    - "이 객체는 생성된 후에 **절대 변하지 않습니다.**"
    - 모든 프로퍼티가 `val`이어야 합니다.
    - 예: `data class User(val name: String)`
- **@Stable**:
    - "이 객체는 변할 수 있습니다. 하지만 **변한다면 반드시 Compose에게 알림을 줍니다.**"
    - 또는 "인터페이스(List)지만 실제로는 안 변한다고 약속합니다."
    - 예: `MutableState`를 가진 클래스, `Wrapper` 클래스.

---

## 5. 실제 코드 최적화 예시

### [Before] 최적화 실패 (List 사용)

```kotlin
// 데이터 모델 (Unstable)
data class UserListState(
    val users: List<String> // List는 Unstable
)

@Composable
fun UserList(state: UserListState) {
    // state 자체가 Unstable하므로,
    // 부모가 리컴포지션 될 때마다 여기도 무조건 다시 그려짐 (Jank 유발 가능)
    LazyColumn { ... }
}

```

### [After] 최적화 성공 (ImmutableList 사용)

```kotlin
// 데이터 모델 (Stable)
data class UserListState(
    val users: ImmutableList<String> // Stable
)

@Composable
fun UserList(state: UserListState) {
    // users 내용이 변하지 않았다면,
    // 부모가 리컴포지션 되어도 이 함수는 건너뜀 (Skip Recomposition)
    LazyColumn { ... }
}

```

---

## 요약

1. **Stable**한 상태여야 Compose가 변경 사항이 없을 때 그리기(Recomposition)를 건너뛸 수 있습니다.
2. **`val` + Primitive/String** 조합은 항상 Stable입니다.
3. `var`는 MutableState로 감싸야 Stable이 됩니다.
4. List, Map, Set은 기본적으로 **Unstable**입니다.
    - 해결책 A: kotlinx.collections.immutable 사용 (추천).
    - 해결책 B: @Stable이 붙은 Wrapper 클래스 사용.
    - 해결책 C: 파라미터에 @Stable 어노테이션 붙이기 (클래스 정의 가능한 경우).
