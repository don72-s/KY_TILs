# Shared(P2P) Network 구성 [ Fusion ]

멀티게임을 만들기 위해서는 네트워크 기반 지식과 구현 능력이 필요하다.

하지만 요즘은 해당 역할을 대신하는 플랫폼들이 많이 존재한다.

여기서는 포톤 엔진을 사용해서 네트워크를 구성하는법을 알아보도록 하자

<br>

## 개발 환경 구성

멀티환경을 구축하고 외부 플랫폼을 사용하는 이상, 요구되는 개발 환경을 먼저 구성해야 할 필요가 있다.

포톤의 fusion을 사용하려면 어떤 환경을 구성해야하는지 자세히 알아보자.

<br>

### 개발 환경 구성 방법

다음과 같은 순서로 개발 환경을 구현한다.

1. 포톤 사이트에 들어가서 새로운 애플리케이션을 생성한다.
2. fusion을 사용할 것이므로 해당 항목으로 설정하고, sdk를 입력한다.
3. 유니티 프로젝트를 생성하고 Edit > Project Settings > Editor > Asset Serialization 항목이 Force Text인지 확인한다.
4. 유니티 패키지 매니저를 연다.
5. add package from git을 이용하여 **com.unity.nuget.mono-cecil@1.10.2**패키지를 추가한다.
6. fusion사용에 필요한 sdk를 유니티 asset store에서 다운받는다 [ photon fusion ]
7. Tools > Fusion > Fusion Hub에 포톤에서 생성한 애플리케이션 ID를 입력한다.

필요에 따라 voice server, chat server를 추가한다.

<br>

### 테스트를 위한 환경 구성

멀티게임의 환경을 디버그하기 위해서는 여러개의 빌드된 파일이 필요하다.

하지만 parralSync 에디터를 이용해 조금 더 편하게 테스트해볼 수 있다.

1. **https://github.com/VeriorPies/ParrelSync**에 가서 프로젝트를 다운받는다.
2. **ParralSync**폴더를 유니티 프로젝트에 넣는다.
3. 위에 ParralSync탭을 이용해 같은 프로젝트를 복제하여 테스트한다.

<br>
### 서버 연결 테스트

기본 작업이 완료되었으니, 정상적으로 동작하는지 테스트해볼 필요가 있다.

하이어라키뷰로 들어가서 Create > Fusion > Scene > Setup Network In The Scene 항목을 선택한다.

그러면 Prototype Network Start와 Prototype Runner 두개의 항목이 추가된다.

- **Prototype Network Start** : 네트워크 연결을 쉽게 할 수 있도록 도와주는 인터페이스
- **Prototype Runner** : 네트워크를 관리하는 핵심 오브젝트, 각 상황별 이벤트를 제공하여 콜백 함수를 연결할 수 있다.

<br>

## 네트워크와 동기화

멀티 게임을 위해서는 각 클라이언트별로 같은 종류의 오브젝트가 생성되고, 동기화되어 관리해야 한다.

한쪽에서 상호작용한 영향을 받은 정보를 다른 모든 클라이언트들이 공유하고 적용해야 하기 때문이다.

어떤 방식으로 해당 구성 요소들을 만들 수 있을지 알아보자.

<br>

### SimulationBehaviour

포톤 퓨전에서 제공하는 네트워크 기능을 사용하기 위해서 기존 monobehaviour의 기능을 확장한 SimulationBehaviour을 상속해야 한다.

해당 클래스를 상속받기 위해서는 **Fusion** 네임스페이스를 using해야 한다.

```cs
using Fusion;
public class TestClass : SimulationBehaviour{
    
}
```

<br>

### 이벤트 핸들링

서버측에서 일어나는 이벤트들을 기반으로 행동을 구현해야 하는 경우가 있다.

새로운 플레이어가 입장했을 때, 해당 플레이어의 객체를 만든다던지, 플레이어가 퇴장했을 때 해당 플레이어를 제거한다는 등의 동작이 필요하다.

따라서 해당 이벤트가 발생한 경우의 콜백이 필요한데, 이를 위해서 자동으로 호출되는 인터페이스들이 존재한다.

- **IPlayerJoined** : 플레이어가 참가했을 때 호출되는 인터페이스 구현
- **IPlayerLeft** : 플레이어가 퇴장했을 때 호출되는 인터페이스

<br>

또한, 접속한 플레이어가 본인 플레이어인지, 리모트 플레이어인지 분류할 방법이 필요하다.

적어도 shared 네트워크 항목에서는 본인이 자신을 한번만 생성하면 되므로, 다음과 같은 방식으로 코드를 작성하면 된다.

또한 네트워크에서 공유하기 위한 오브젝트는 Instantiate 방식이 아닌
`Runner.Spawn(prefab);`방식으로 생성해야 한다.

`Runner를 이용하기 위해서는 Network Runner 컴포넌트가 있는 오브젝트에 추가해야한다!`

```cs
GameObject playerPrefab;

public void PlayerJoined(PlayerRef player){

    if(player != Runner.LocalPlayer)
        return;
        
    Runner.Spawn(playerPrefab);
}
```

<br>

여기서 플레이어 프리팹에도 컴포넌트 지정이 필요하다.

Runner.Spawn에서 만들 수 있도록 하려면 해당 프리팹에는
`NetworkObject`컴포넌트가 붙어있어야 한다.

또한, 해당 오브젝트의 transform정보를 동기화하기 위해서는
`NetworkTransform`컴포넌트를 붙여야 한다.

두 컴포넌트가 붙은 오브젝트는 네트워크 환경에서 위치가 동기화되게 된다.

<br>

## 소유권과 제어권

내 플레이어가 있다고 가정하자.
위치를 동기화시키려고 하는데, 다른 클라이언트에서 내 캐릭터를 이동시키려고 하면 어떻게 될까?

판단을 할 수 없을 것이다.

따라서 네트워크 오브젝트를 제어하기 위해서는 소유권이 있어야 하며, 소유권은 해당 오브젝트를 만든(Spawn) 클라이언트가 가지게 된다.

해당 오브젝트의 소유권자가 누구인지는 런타임 시에
해당 네트워크 오브젝트의 **StateAuthority** 속성에서 확인할 수 있다.

<br>

### 룸 오브젝트

생성된 오브젝트는 생성한 클라이언트가 소유권을 가진다고 했다.

그렇다면 처음부터 씬에 존재하던 네트워크 오브젝트는 누가 소유권을 가질까?

우선 shared 즉, p2p환경에서는 처음 들어온 클라이언트가 해당 오브젝트의 소유권을 가지며, 나가게 된다면 해당 오브젝트도 제거된다.

그러므로 p2p 방식이라면 방장을 위임하는 동작이 구현되어 있어야 한다.

<br>

## 오브젝트의 제어

오브젝트 생성이 완료되었다.
그렇다면 이제 플레이어를 움직이게 만들어야 하겠다.

다만 네트워크 환경이다보니까 몇가지 더 고려해야 할 점이 있으므로 조금 더 자세히 살펴보도록 하자.

<br>

### NetworkBehaviour

이전에 network transform 컴포넌트가 있었다.
해당 컴포넌트는 NetworkBehaviour를 상속한 상태인데, 이는 Network Object컴포넌트가 있는 오브젝트에 포함된다면, 해당 컴포넌트는 네트워크간의 데이터 동기화가 됨을 의미한다.

따라서 동기화시키고싶은 컴포넌트가 있다면 해당 컴포넌트는 **Network Behaviour** 상속시킬 필요가 있다.

<br>

### FixedUpdateNetwork

기존의 Update 함수는 사용자의 컴퓨터 환경에 따라 매 프래임 호출되는 함수였다.

하지만, 네트워크 환경에서는 그 속도로 갱신하는게 사실상 불가능하다.

따라서 네트워크 동기화를 위한 전용 Update함수가 필요하다.

<br>

이를 위해서 Fusion에서 제공하는 **FixedUpdateNetwork**함수를 오버라이드 하여 사용하면, 네트워크 통신 주기마다 일정하게 호출시킬 수 있다.

따라서 `두 함수에 들어가야 할 내용을 구분하고 분리하여 구현하는 능력이 필요하다.`

가장 기본적으로는 클라이언트의 입력을 받는 **Input**계열은 **Update**, 프레임 상황에 따라 달라지는 연산이며 동기화해야하는 경우에는 **FixedUpdateNetwork**에 구현할 필요가 있겠다.

<br>

### deltaTime, Runner.deltaTime

캐릭터의 이동에는 클라이언트 성능과 관계없이 일관적으로 동작하도록 **Time.deltaTime**을 통하여 보정했었다.

하지만, 네트워크에서 이동처리는 **FixedUpdateNetwork**를 통하여 이동시키게 될 것이다.

하지만 **Time.deltaTime**은 Update의 보정 수치이므로 그대로 적용할 수 없다.

이를 위해서 Fusion에서 따로 제공하는 보정값이 있는데 이게 바로
`Runner.DeltaTime`이다.

따라서 **FixedUpdateNetwork**에서의 보정값은 Time.deltaTime이 아니라 **Runner.DeltaTime**을 대신 사용하면 되겠다.

<br>

### 제어권의 확인

여러명이 접속한 플레이어 프리팹에는 모두 컨트롤러 스크립트가 붙어있을것이다. 그렇다면, 모든 플레이어 프리팹이 입력을 받고 이동할것임을 의미한다.

보통의 경우에는 본인의 캐릭터만 움직이게 해야 할 것이다.

이를 위해 소유권을 가진 객체만 입력을 받도록 처리할 필요가 있다.

이를 위해서 `HasStateAuthority`속성을 사용할 수 있다.

해당 객체의 소유권을 가지고있다면 **true**를 반환하는 속성이기 때문에, 이를 통해 제어해도 되는 오브젝트인지 판단할 수 있다.

<br>

## network, simulation

두가지 종류의 Behaviour를 상속했었는데, 두 클래스를 상속하는 상황을 간단하게 정리하고 넘어가자.

- **NetworkBehaviour** : 캐릭터의 동작, 이동 관련 동기화가 필요할 때 NetworkObject에 붙어서 동작.
- **SimulationBehaviour** : Runner에 붙어서 다른 캐릭터의 입장, 퇴장 등등의 상태 위주의 이벤트 구독, 관리가 필요할 때 상속.

<br>

## 캐릭터별 카메라 제공

멀티게임에서는 여러명의 플레이어가 존재하는 만큼 하나의 카메라를 공유하는 방식은 특수한 경우가 아니라면 사용할 수 없다.

각 클라이언트별로 본인의 캐릭터에 포커스를 둔 카메라가 존재해야 한다는 뜻이다.

즉, 카메라는 `네트워크로 동기화를 할 필요가 없다`라는 결론이 나오는 것이다.

<hr>

따라서 단순히 주시할 대상을 본인의 캐릭터로 설정하면 된다. 

<br>

### Spawned

NetworkBehaviour는 해당 오브젝트가 Spawn되었을 때 콜백으로 호출되는 함수를 virtual로 제공한다.

해당 함수가 Spawned이다.

해당 함수는 `awake/start`와는 다르게 네트워크 오브젝트가 준비 된 뒤에 호출되는 함수이므로 네트워크 객체라면 초기화를 해당 함수를 사용해야한다.

예를들어 **HasStateAuthority**같은 경우에는 네트워크 연결이 완료되지 않은 경우 호출할 수 없으므로 **Awake/Start**에서 사용하면 `안된다.`

따라서 캐릭터 컨트롤러에서 Spawned를 오버라이드하여 카메라의 타겟을 지정하게 해주면 카메라 등록이 완료된다.

```cs
public override void Spawned() {

    if (HasStateAuthority) {
        Camera.main.GetComponent<CameraController>().target = transform;
    }

}
```

<br>