# 네트워크 동기화 기법(Pun2)

지난시간까지 Photon Fusion에 대해 알아봤다.
하지만 상당히 하이레벨로 구현되어있고, 나온지 얼마 안된 기술이라 상세한 프로그램 구현도 하기 힘들었고 자료도 얼마 없었다.

이번에는 조금 오래되었지만 매우 자주 사용되는 Photon Pun2에 대해 알아보도록 하자.

<br>

## Pun의 기능

Pun에서 제공하는 가장 큰 기능은 다음과 같다.

- **매치메이킹** : 멀티게임에서 방을 생성하고 찾아가거나 자동으로 매칭시켜주는 기능을 제공한다.
[빠른참가, 친구참가, 비밀방, 제한참가]

<br>

- **룸** : 매칭된 클라이언트들간의 데이터를 동기화시켜주는 기능을 제공한다.
[커스텀 프로퍼티, RPC, 변수 전달 동기화]

<br>

## 개발환경 구성

Fusion과 마찬가지로 Pun을 이용한 개발을 위해서는 우선 개발 환경을 구성할 필요가 있다.

1. 포톤 홈페이지로 가서 새 어플리케이션을 **Pun2**로 생성한다.
2. 유니티 에셋스토어로 이동하여 **Pun2 Free**를 다운받은 뒤, 프로젝트에서 임포트한다.
3. Setup창에 작성한 Pun애플리케이션 ID를 입력하여 연동을 완료한다.
4. (선택사항) ParralSync를 다운받는다.
<br>

## 포톤의 네트워크 구성

포톤의 매치매이킹의 구조는 클라이언트-서버 구조로 되어있다.

포톤 서버가 데디케이트 역할을 하게 되므로 클라이언트는 서버에 요청하게 되고, 서버는 이에 대한 반응을 하게 된다.

<br>
### 클라이언트의 요청

클라이언트측에서 포톤 서버에 요청을 보내기 위한 기능들은
모두 `PhotonNetwork`클래스에 정의되어있다.

함수만이 아닌, 현재 본인(클라이언트)의 상태를 반환받기 위한 속성들도 존재한다.

<br>

### 서버의 반응

서버측에서는 여러가지 일이 일어났음을 정보(매개변수)와 함께 전달해줄 것이다.

해당 반응은 콜백 방식으로 구현되어있으며, 대응되는 콜백 함수들을 Override하여 대응 행동을 구현할 수 있게 되어있다.

해당 콜백함수들은 `MonobehaviourPunCallbacks`클래스에 정의되어있으므로, 해당 클래스를 상속받은 뒤 대응 콜백함수를 재정의해서 사용할 수 있다.

```cs
public class NetworkManager : MonoBehaviourPunCallbacks
{
  public override void OnConnected() { }                          // 포톤 접속시 호출됨
  public override void OnConnectedToMaster() { }                  // 마스터 서버 접속시 호출됨
  public override void OnDisconnected(DisconnectCause cause) { }  // 접속 해제시 호출됨
  //...기타 등등...
}
```

<br>

### 세부 속성/함수

- **ClientState** : 클라이언트의 현재 상태를 Enum상태로 정의한다.
**PhotonNetwork.NetworkClientState**를 통해 서버로부터 상태를 받아올 수 있다.
- **PhotonNetwork.LocalPlayer.NickName** : 클라이언트의 닉네임을 설정한다.(중복 가능)
- **PhotonNetwork.ConnectUsingSettings()** : 설정된 네트워크 세팅으로 서버에 접속을 요청한다.
<br>

- **RoomOptions** : 방을 생성할 때 추가적인 옵션을 지정하기 위한 클래스

```cs
        RoomOptions roomOptions = new RoomOptions();
        roomOptions.IsVisible = true;           //방 공개 여부
        roomOptions.IsOpen = true;              //다른 플레이어 접속 가능 여부
        roomOptions.MaxPlayers = _result;       //최대 플레이어 접속자 수(0인 경우 제한인원 없음)
        roomOptions.PlayerTtl = 0;              //플레이어 응답없음 대기시간
        roomOptions.EmptyRoomTtl = 0;           //마지막 플레이어가 떠났을 때 방이 유지되는 시간
        roomOptions.CleanupCacheOnLeave = true; //방에서 떠났을 때 방 정보를 제거할지의 여부
        //roomOptions.CustomRoomProperties        //커스텀 데이터
```

<br>

- **PhotonNetwork.JoinRandomRoom()** : 랜덤으로 방에 접속한다.
`HashTable` 매개변수를 이용해서 특정 조건이 걸린 방들 중에서만 랜덤으로 접속하도록 설정할 수 있다.

<br>

- **OnRoomListUpdate(List< RoomInfo > roomList)** : 클라이언트가 로비에 있는 동안, 해당 콜백함수가 주기적으로 호출된다.
`List< RoomInfo >` : 로비에 있는 방들의 리스트 정보.
<br>
##### 주의사항
방 목록이 변경되었을 때, 모든 방 정보를 보내는것이 아니라 변경사항만을 보내준다.
`사라졌을때에는 사라진 방 목록을 보내줌.`

<br>

- **PhotonNetwork.PlayerList(Others)** : 방에 입장된 상태라면 해당 방에 있는 모든 플레이어(또는 나를 제외한)목록을 가져온다.

- **Player.GetPlayerNumber()** : 방에 접속해있는 사람들의 넘버링된 번호를 가져온다. 

해당 번호는 0번부터 부여하여 겹치지 않게 부여되며, 중앙의 사람이 빠져도 빈 숫자로 `당겨지지 않는다.`

따라서 유니티상에서 배열등을 사용하기 매우 유용하다.

##### 주의점

플레이어가 방에 들어가자마자 번호를 할당받는것이 아니다.
조금 딜레이가 있으므로 OnEnable등에서 사용하는것은 불가능하므로 따로 적용되는 이벤트를 이용할 필요가 있다.

- **PlayerNumbering.OnPlayerNumberingChanged** : 해당 정적 변수 이벤트에 등록한다면 현재 들어가있는 방의 구성원들 중 아무나 Numbering에 변화가 생긴다면 이벤트를 발생시킨다.<br><br>또한 플레이어가 중간에 나가는 경우에도 호출되므로 방에 대한 정보를 동기화하는데 유용하게 사용할 수 있다.

<br>

### PlayerNumbering

포톤에서 제공하는 컴포넌트중 하나인 `PlayerNumbering`컴포넌트가 있다.

해당 컴포넌트는 플레이어가 방(Room)에 입장되어있는 경우, 해당 방에 있는 클라이언트의 ID들을 직관적으로 띄워준다.

<br>

## 커스텀 프로퍼티

포톤의 플레이어와 게임 룸에는 기본적으로 여러가지 속성들이 존재한다.

하지만, 제공되는 정보들로는 부족할수도 있다. 팀 구성 정보, 비밀방과 비밀번호 등등...

따라서 필요에 따라 맵과 플레이어에 속성을 추가할 수 있는 능을 지원하는데 이것이 **커스텀 프로퍼티**이다.

<br>

### HashTable

커스텀 데이터를 지원한다고 해서 막 데이터를 주입할수는 없다.
커스텀 프로퍼티를 이용하려면 **HashTable**이라는 자료구조를 이용해야 하는데 기존 c#의 HashTable과 혼동하지 않는것이 중요하다.

<br>

### 프로퍼티 추가

프로퍼티의 추가는 다음과 같이 이루어진다.

```cs
Player localPlayer = PhotonNetwork.LocalPlayer;
PhotonHashTable hashTable = new PhotonHashTable();

hashTable.Add("Ready", false);
localPlayer.SetCustomProperties(hashTable);
```
 
1. 추가 또는 수정할 플레이어를 가져온다.
2. 해시테이블을 생성한다.
3. 추가,수정할 데이터를 해당 테이블에 저장한다.
4. 해당 플레이어의 **SetCustomProperties(해시테이블)** 함수를 호출한다.

<br>

- 값을 저장할 때 이미 저장된 같은 키의 값이 있다면 덮어쓰게 되므로 염두에 둘 필요가 있다.

<br>

### 프로퍼티 읽기

커스텀 프로퍼티를 저장했으니, 읽어오는 방법을 알아보자.

기본적으로 다음과같은 방식으로 읽게된다.

```cs
Player localPlayer = PhotonNetwork.LocalPlayer;
bool val = (bool)localPlayer.CustomProperties[키값];
```

<br>

1. 프로퍼티를 읽어올 플레이어를 가져온다.
2. 플레이어로부터 커스텀 프로퍼티를 가져온다.
3. 딕셔너리 방식으로 값을 가져온다.
4. 저장했던 자료형으로 언박싱한다.

<br>

당연스럽게도 유의할점들이 있다.

- 저장되지 않은 데이터의 키값을 전달하면 null이 반환된다.
- 올바른 언박싱이 아닌 경우에는 에러가 난다.

<br>

### 확장 메서드의 활용

이처럼 자주 사용하는데 길어지는 코드는 확장메소드를 응용할 수 있다.

커스텀메소드를 넣는것과 빼는것, 해당 동작들을 다음과 같이 확장메소드를 이용해서 간략화할 수 있다.

```cs
public static class CustomProperty {

    public const string READY = "Ready";

    static Hashtable customHashtable = new Hashtable();

    public static void SetReady(this Player _player, bool _isReady) {

        customHashtable.Clear();
        customHashtable.Add(READY, _isReady);
        _player.SetCustomProperties(customHashtable);

    }

    public static bool GetReady(this Player _player) {

        if (_player.CustomProperties.ContainsKey(READY)) {

            return (bool)_player.CustomProperties[READY];

        } else {

            Debug.LogError($"등록되지 않은 커스텀 변수를 찾으려고 했습니다. : {READY}");
            return false;

        }

    }

}
```

<br>

이렇게 작성해놓으면 본인의 커스텀 프로퍼티에 접근한다고 가정했을 때 다음과 같이 사용할 수 있다.

`PhotonNetwork.LocalPlayer.GetReady();`
`PhotonNetwork.LocalPlayer.SetReady(curReadyState);`


<br>

## 마스터 클라이언트

Fun의 경우 **Host - Client**구조를 사용하고 있기 때문에 방이 개설되면 호스트가 되는 방장이 배정되어야한다.

<br>

해당 클라이언트는 **마스터 클라이언트**로 지정되어 게임을 시작하거나 방의 상세 설정등을 할 수 있는 권한을 가질 필요가 있다.

<br>

### 배정과 자동 위임

기본적으로 룸에 처음 들어온 사람이 **마스터 클라이언트**를 배정받는다.
<br>
이후에 해당 플레이어가 방을 나간다면, 해당 권한을 다른 플레이어에게 자동으로 위임하게 되므로 마스터 클라이언트는 언제나 존재함을 보장할 수 있다.

<br>

### 방장의 위임

보통의 게임에서는 방장이 다른 플레이어에게 방장을 위임할 수 있다.

포톤에서는 다음과 같은 함수를 호출하여 마스터클라이언트를 위임할 수 있다.

`PhotonNetwork.SetMasterClient(대상 플레이어);`
<br>

당연하게도 이는 마스터 플레이어인 경우만 호출이 가능한 함수이며, 본인이 마스터플레이어인지 확인할 수 있는 속성도 다음과 같다.

`Player.IsMasterClient`

<br>

### 방장의 변화 콜백

방장 클라이언트 여부 프로퍼티는 기본 제공 프로퍼티이므로 
**OnPlayerPropertiesUpdate** 콜백을 호출하지 않는다.

<br>

하지만 방장의 변화에 따라 표시되는 UI정보를 수정할 필요가 있으므로 콜백 함수를 받을 필요가 있다.

<br>

이를 위해 포톤에서 제공하는 마스터 클라이언트 변화 콜백 함수를 사용할 수 있으며, 해당 함수는 다음과 같다.
`public override void OnMasterClientSwitched(Player newMasterClient)`

<br>

해당 함수를 오버라이드하면 방장이 바뀌는 순간 호출된다.

해당 상황은 **자동 위임, 지정 위임** 모두 해당된다.

<br>

## 장면의 전환

이제 모든 플레이어가 준비되었고 방장이 게임 시작 버튼을 누를 차례다.

다같이 장면을 전환하기 위해서는 포톤에서 제공하는 방법으로 씬을 전환할 필요가 있다.

<br>

기본적으로 다음과 같은 코드가 된다.
```cs
//방장 확인
if (!PhotonNetwork.LocalPlayer.IsMasterClient)
   return;

//모든 구성원들도 이동하도록 설정
PhotonNetwork.AutomaticallySyncScene = true;    // 장면 전환을 신청함.
PhotonNetwork.LoadLevel([씬 이름]);          
```
<br>

`PhotonNetwork.AutomaticallySyncScene = true`
해당 코드는 방장이 포톤서버에 장면전환을 요청한 경우, 방에 있는 다른 플레이어들도 해당 장면으로 전환하도록 설정하는 것이다.

**다만, 해당 설정은 모든 클라이언트가 해야하므로, start에서 각 클라이언트가 세팅하게 하는것이 좋다.**

<br>

`PhotonNetwork.LoadLevel([씬 이름])`
이후, 해당 함수를 호출하여 장면을 전환하면, 방에있는 구성원 모두 씬을 전환하게 된다.

<br>

##### 난입 금지

게임이 플레이에 들어가도 해당 룸은 계속 존재하는 상태이다.
만일 이 상태에서 플레이어가 방에 들어오게되면 바로 게임씬으로 전환될 것이다.

<br>

이러한 난입 기능을 제한하고싶다고 하면 다음과 같은 코드를 게임이 시작될 때 실행하도록 하면 된다.
`PhotonNetwork.CurrentRoom.IsOpen = false;`
<br>

다만, 주의점은 게임이 완료되고 대기실로 돌아올 때 해당 변수를 마스터 클라이언트가 **true**로 바꿔줄 필요가 있다.

<br>