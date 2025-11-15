# OpenGL

흔히 OpenGL이란, 그래픽과 이미지를 조작하는 하나의 API라고 볼 수도 있겠지만, OpenGL 자체는 API가 아니며, khronos사에서 개발하고 관리하는 하나의 사양이라고 한다.
OpenGL은 C++을 사용해서 개발되었다. 그렇기 때문에 C++에 대한 이해가 조금 필요하다.

## Core-profile vs Immediate mode

이전에는 OpenGL을 사용하면 Immediate mode(고정 파이프 라인이라고도 한다.)로 개발해야 했다. 
그래픽을 그리는데 사용하기 쉬웠지만, 대부분의 OpenGL 함수들은 라이브러리 안에 캡슐화되어 있었기 때문에 계산식을 조작하는데 어려움이 있었다. <br>
이러한 이유로 3.2 버전 이후로 Core-profile을 사용하도록 권장했다. 
Core-profile은 이전 방식보다 더 유연하고 효과적이지만, 배우는데 더 어렵다. 
기존 Immediate mode 방식은 일부 계산이 추상화 되어 있기 때문에 사용하기 쉬웠지만, OpenGL의 실제 동작 방식을 이해하기는 어려웠다. 
Core-profile은 OpenGL과 그래픽 프로그래밍 방식을 이해해야 하는 것이 어렵지만 더 많은 유연성과 효율성을 제공하며 그래픽 프로그래밍 방식에 대해 더 많이 이해할 수 있다. <br>

OpenGL 3.3 이후부터 핵심 메커니즘이 변경되는 것은 아니고 더 유용한 기능을 제공할뿐 기본적인 동작은 동일하다.
또한, 최신 버전은 최신 그래픽 카드에서만 동작하기 때문에 대부분 범용성을 고려하여 버전을 맞춰서 개발하는 편이다.

## Extensions
OpenGL의 가장 큰 특징 중 하나는 확장 기능이라고 한다. 그래픽 회사가 새로운 기술이나 렌더링 최적화를 내놓을 때마다 드라이버에 구현된 확장 기능을 통해 확인할 수 있다.
하드웨어가 이런 확장 기능을 제공하면 개발자는 확장 프로그램에서 제공하는 기능을 사용할 수 있다. 확장 기능을 효율적으로 사용할 수 있다. 결과적으로 그래픽 개발자는 확장 기능을 통해 새로운 렌더링 기법을
사용할 수 있게 되고 OpenGL의 새로운 버전에 기능이 추가될 때까지 기다리지 않아도 된다. (종종 매우 유용한 확장 기능은 새 버전에 추가되기도 한다.)
물론, 개발자는 분기문을 통해 확장 기능을 사용할 수 있는지 확인해야 한다.

## State machine
OpenGL은 하나의 거대한 State machine 으로 동작한다. State machine은 OpenGL이 어떻게 동작해야 하는지 정의한 변수의 모음이다.
OpenGL의 현재 상태를 정의하는 것을 Context라고 한다. 옵션을 조작하고 Buffer를 조작함으로서 상태를 변경할 수 있다. 그러면 Current-Context를 사용해서 렌더링한다. <br>
예를 들어, OpenGL에 삼각형 대신 선을 그리고 싶다고 말할때마다, OpenGL의 동작을 설명하는 일부 Context 변수를 변경하여 OpenGL의 상태를 변경하는 것이다. <br>

## Object
OpenGL의 객체는 OpenGL의 상태를 표현하는 옵션의 집합이다. OpenGL 객체는 C 형태로 작성되어 있다.
```C
struct object_name {
    float option1;
    int option2;
    char[] name;
};
```
```C
// The State of OpenGL
struct OpenGL_Context {
    ...
    object_name* object_Window_Target;
    ...
};
```
```C
// create object
unsigned int objectId = 0;
glGenObject(1, &objectId);
// bind/assign object to context
glBindObject(GL_WINDOW_TARGET, objectId);
// set options of object currently bound to GL_WINDOW_TARGET
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_WIDTH, 800);
glSetObjectOption(GL_WINDOW_TARGET, GL_OPTION_WINDOW_HEIGHT, 600);
// set context target back to default
glBindObject(GL_WINDOW_TARGET, 0);
```
OpenGL의 workflow를 보여주는 코드 조각이다.
1. 객체를 생성하고 객체에 대한 참조를 id로 저장한다. (glGenObject 안에서 실제 객체의 데이터를 저장한다.)
2. id를 사용하여 Context의 대상 위치에 Binding 한다. (여기서는 GL_WINDOW_TARGET이라는 Object에 위치한다.)
3. Window 옵션을 설정하고 id를 0으로 하여 바인드를 해제한다.
4. 설정한 옵션은 objectId에 의해 참조된 객체에 저장된다.

OpenGL의 객체를 사용하여 그리고 싶은 모델 데이터를 포함하는 객체를 생성하고(여러 옵션들을 설정해서 바인딩하고) 다수의 객체를 생성할 수 있다.
