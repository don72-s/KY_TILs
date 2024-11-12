# 컨트롤러 조작

VR 프로젝트의 필수 구성 사항과 기본적인 조작법 구현에 대해 알아보자.

<br>

## 필수요소

AR때와 마찬가지로 씬에는 반드시 존재해야하는 모브젝트들이 있다.

**XR Origin** : HMD의 위치를 카메라의 위치로 잡는 필수적인 구성요소이다.

Create > XR > XR Origin을 생성한 뒤, 기존에 존재하는 Main Camera를 삭제하면 된다.

**XR Interaction Manager** : 상호작용을 전체적으로 관리하는 오브젝트이며 컨트롤러에 해당하는 **Interactor**와 상호작용 대상이 되는 **Interactable** 오브젝트와의 상호작용을 중계하는 역할을 한다.

`XR Origin을 추가하면 자동으로 생성된다.`

**Locomotion System** : 플레이어의 이동, 회전 , 순간이동등의 모든 이동 관련 행위를 관리하는 오브젝트이다.

Create > XR > Locomotion System을 생성하면 된다.

<hr>

다만, 위의 오브젝트들은 설정이 되어있지 않으므로 설정되어있는 요소들을 가져와 사용할 수 있는 방법이 있다.

XR Toolkit Asset을 다운받을 때 Starter Assets를 다운받게 된다면 이미 설정이 완료된 프리팹을 가져와서 사용할 수 있다.

Starter Assets > Prefabs > XR Interaction Setup 프리팹을 사용하면 위의 3가지 필수 오브젝트들을 기본 설정이 완료된 채로 사용할 수 있다.

<br>


## VR에서의 UI

지금까지의 UI는 화면에 출력하는 방식을 사용했다.

하지만 VR화면에서는 버튼이 계속 눈앞에 붙어있어봤자 불편하기만 하고 사용도 못할것이다.

따라서 VR에서 UI를 사용하기 위해서는 Canvas모드를 WorldSpace로 바꾸어준 뒤 배치하면 된다.

<br>

다만 그것만으로는 VR컨트롤러의 Ray와 상호작용하지 않는다.

기존의 Event System은 마우스/키보드 입력으로 되어있으므로 EventSystem에 XR UI InputModule컴포넌트가 있어야 한다.

마지막으로 UI Canvas가 컨트롤러 입력을 인식할 수 있도록 Canvas 오브젝트에 **Tracked Device Graphic Raycaster** 컴포넌트를 추가해 주면 UI가 컨트롤러의 Ray 상호작용을 인식할 수 있게 된다.


<br>
## 텔레포트

VR 조작을 통한 텔레포트 이동을 위해서는 텔레포트가 가능한 지역을 지정해야 한다.

이를 위해서는 영역을 지정으로 하는 스크립트가 부착되어 있는 곳으로만 텔레포트가 가능하다.

- **Teleportation Area** : 해당 스크립트가 부착되어있는 오브젝트의 영역 중, 지정한 곳으로 순간이동한다.

- **Teleportation Anchor** : 해당 영역으로 텔레포트하면, 지정된 위치로 이동시킨다.

`위의 스크립트들을 부착시킨 후, Interaction LayerMask를 [Teleport] 로 지정해주어야 한다.`

<br>

## 조작법의 전환

세세하게 바꾸기까지는 힘들지만, 오른손으로 이동하도록, 왼손으로 회전하도록 바꾸는 등의 조작은 할 수 있다.

Left(Right) Controller 오브젝트의 **Controller Manager** 컴포넌트의 Locomotion Settings항목의 속성을 조작하여 조작방식을 바꿀 수 있다.

아무것도 체크가 되어있지 않다면 **회전/순간이동**
Smooth Motion Enabled가 체크되었다면 **이동**
Smooth Turn Enabled가 체크되었다면 **부드러운 회전/순간이동**이 된다.

코드상에서 수정하려면 ActionBasedControllerManager 컴포넌트를 가져와서 
`smoothMotionEnabled, smoothTurnEnabled`
의 속성을 수정하는것으로 런타임 도중에도 조작법을 변경할 수 있다.

<br>

## 오브젝터와의 상호작용

오브젝트와의 상호작용을 위해서는 상호작용을 하는 **Interactor**, 상호작용을 기다리는 **Interactable** 두가지 종류로 이루어진다.

<br>

### Interactor

interactor는 단적으로 컨드롤러를 들고있는 손을 의미한다.

상호작용할 수 있는 종류는 다음과 같다.

**Lay Interactor** : lay를 쏘아서 해당 위치에 있는 interactabe 물체와 상호작용을 일으킨다.

**Poke Interactor** : 손가락으로 쿡 쿡 찌르는듯한 상호작용을 일으킨다.

**Direct Interactor** : 손이 interactable에 직접적으로 닿았을 때 상호작용을 일으킨다.

**Socket Interactor** : 거치대 역할을 하는 interactor이다.
GrabInteractable 오브젝트가 접근하게 되면 해당 거치대에 고정시키는 역할을 한다.

<br>

### Interactable

interactable 컴포넌트를 가지고 있는 오브젝트는 상호작용이 일어났을 때 일어나는 반응을 정의한다.

**Grab Intractable** : 상호작용이 일어나면 잡히는 반응을 보이게 된다. 물체를 들어올리거나 던지는 행동이 가능하다.

> Grab 상호작용은 잡은 물체가 벽을 통과하는 현상이 생긴다.
> 해당 현상이 싫다면 **Movement Type**을 Velocity Tracking으로 설정하면 된다.

**Climb Interactable** : 상호작용이 일어나면 해당 물체를 기준으로 움직이게 된다. 사다리를 잡고 팔을 당기거나 폈을 때의 이동을 상상하면 된다.

**Simple Interactable** : 상속용 Interactable이다. 상호작용이 일어났을 때 일어날 행동을 커스텀하여 구현하기 위한 스크립트이다.

<br>

### 상호작용 이벤트

모든 Interactable 스크립트들은 여러가지 상황에 따른 이벤트들을 가지고 있다. 따라서 상호작용이 발생했다면 발생할 행동들을 등록하여 더욱 세세한 상호작용을 구현할 수 있다.

<br>

- **Active** : 트리거 버튼(검지)을 기준으로 push되었을 때 발생한다.

- **Select** : 홀드버튼(중지)을 기준으로 push되었을 때 발생한다.

<br>