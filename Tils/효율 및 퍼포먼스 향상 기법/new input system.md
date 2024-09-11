# Input System

사람들이 키보드만을 이용하게 된다면 별다른 문제가 없겠지만 이제는 크로스플랫폼 개발이 원활하게 이루어지고있다.

매우 자주 사용되는 입력장치인 **키보드마우스/콘솔패드**등을 이용한 Input Manager가 구현되어있으나, vr입력, 체감형 컨트롤러 등의 정형화되지 않았거나 최신화된 입력 장치들에는 대응하기가 힘들다.

이러한 문제점을 어떻게 개선하게 되었을까?

<br>

## Input System

여러가지 입력기기 또는 입력 방식에 대응하기 위해서 input system이 개발되었으며, 유니티 패키지 다운로드를 통해 import할 수 있다.

+) 만일 input system이 작동하지 않거나 input manager를 동시에 사용하고 싶다면 다음과 같이 설정하도록 하자.

Project Setting -> Player -> other settings -> configuration -> Active InputHandling 항목을 개발 상황에 따라 설정해주면 된다.

<br>

### input system의 사용

input system을 사용하기 위해서는 전용 컴포넌트가 필요하다.

따라서 우선 **Player Input**컴포넌트를 추가해주면 된다.

컴포넌트가 생성된 뒤에, 기본 입력 scriptable obj를 추가할 필요가 있다. 따라서 create 항목의 **Input Action** 오브젝트를 하나 생성한 뒤 연결해주면 된다.

<br>

### 입력의 설정

Input Action타입의 파일이 생성되면 이제 대응되는 입력들을 정의해줄 필요가 있다.

희망에 따른 입력 분류를 먼저 actions에 생성한 뒤, actionType을 지정해주어야 한다.

- value : 값으로 입력을 받겠다.
- button : 버튼의 눌림으로 입력을 받겠다.
- pass through : 버튼의 눌림, 떨어짐 입력을 받겠다.

<hr>

큰 입력 항목을 추가했다면 받아들일 입력을 정해야 한다.

이를 키 바인딩이라고 하며, 키에 대응하는 입력을 바인딩시키면 된다.

만일 vector2와같은 입력값인데 키보드로 입력을 원한다면 composite그룹을 생성한 뒤, 대응값을 바인딩시키면 된다.


<br>
### 코드에서 입력받기

이제 코드에서 입력을 확인할 필요가 있다.

우선 GetComponent로 **PlayerInput** 컴포넌트를 가져온다.

```cs
input.actions["actions 이름"].ReadValue<입력 타입>();
input.actions["actions 이름"].IsPressed();
input.actions["actions 이름"].WasPressedThisFrame(); // GetKeyDown
input.actions["actions 이름"].WasReleasedThisFrame(); // GetKeyUp
```

등을 이용하여 입력값을 가져올수 있다.

<br>

### 가상 조이스틱 생성

input system을 임포트했다면 가상 조이스틱이 구현된 스크립트를 제공한다.

ui 이미지같은 오브젝트에 **OnScreenStick**컴포넌트를 붙여준 뒤, control path를 지정해준다면 입력을 제공할 수 있다.

`살짝 버벅이는것 같음. 해결되지 않으면 따로 가상스틱을 구현할 것.`

<hr>

또한 **OnScreenButton**컴포넌트를 붙여주게 된다면, 버튼 입력 또한 매핑시켜줄 수 있다.

onclick을 이용할때와 차이점은, keyDown, keyUp의 상황으로써도 사용할 수 있다는 점이다.

<br>