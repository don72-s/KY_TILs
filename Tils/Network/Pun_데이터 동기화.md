# 데이터 동기화

Fusion에서도 그랬듯이, 네트워크에서는 데이터 동기화가 매우 중요하다.
그렇다면 Pun은 어떤 방식으로 데이터를 동기화하는지 알아보자.

<br>

## 포톤 뷰

퓨전때에도 그랬지만, 씬에 존재하는 모든 오브젝트를 동기화시킬 필요는 없다.
따라서 동기화 대상이 되는 게임오브젝트들을 선별하여 동기화할 필요가 있다.

이를 구분하기 위해 부착되는 컴포넌트가 **포톤 뷰** 컴포넌트이다.

<br>

### 속성

포톤 뷰 컴포넌트는 다음과 같은 속성들을 가진다.

- **View ID** : 관리용 식별 ID
- **IsMine** : 소유자 확인
- **Observed** : 변화사항을 감지하고 적용하기 위한 속성
- **Ownership Transfer** : 소유자를 변경할 수 있는지에 대한 여부
- **Synchronization** : 동기화 방식 설정에 대한 속성

###### MonobehaviourPun

해당 오브젝트가 포톤 뷰 컴포넌트를 참조하여 동기화하기 위해서 대상 오브젝트의
스크립트 컴포넌트들은 **MonoBehaviourPun** 클래스를 상속하는것을 추천한다.


<br>

### 프리팹의 위치

포톤 Pun에서의 네트워크 오브젝트의 프리팹 생성과 관리를 위해서는 저장하는 위치가 정해져있어야 한다.
해당 폴더는 반드시 **Resources**라는 명칭을 가지고 있어야 하며, 네트워크 오브젝트 프리팹은 모두
해당 폴더에 들어가 있어야 한다.

<br>

물론 관리가 힘들어질것 같다면 내부에 폴더를 만들어도 괜찮다.
그때에는 Resources를 Root폴더로 취급하여 /를 이용해 경로를 지정하면 된다.

<br>

### 생성과 제거

프리팹을 만들었다면 이제 생성/삭제가 가능해야 하겠다.

<br>

이때에는 동기화를 위해 포톤 네트워크에게 만들겠다는것을 선언해야한다.

Pun에서 네트워크 프리팹을 생성하기 위해서는 다음과 같이 요청할 수 있다.
`PhotonNetwork.Instantiate([프리팹 이름], 위치, 회전)`

<br>

반대로 제거를 위해서는 다음과 같은 코드를 실행하면 된다.

`PhotonNetwork.Destroy([포톤 뷰]);`

<br>

기존에는 게임오브젝트 타입을 전달했으나, 제거를 위해서는 해당 오브젝트에
붙어있는 포톤 뷰 컴포넌트를 전달해야한다.

<br>

## 동기화

이제 만들거나 제거하는법은 알았으니, 실제로 동기화를 시키는 방법에 대해 알아보자.

<br>

### 변수 동기화

동기화의 기본은 변수 동기화라고 할 수 있겠다.
Pun에서는 동기화할 변수는 `IPunObservable`인터페이스로 구분된다.

<br>

우선 PhotonView가 있는 오브젝트에 새로운 스크립트를 부착하고
MonoBehaviourPun을 상속한 뒤, **IPunObservable**인터페이스를 상속한다.

<br>

해당 인터페이스는 다음과 같은 형식을 가진다.
`public void OnPhotonSerializeView(PhotonStream stream, PhotonMessageInfo info)`

<br>

여기서 값을 동기화하는 방식은 스트림을 이용한다.
즉, 선입선출방식으로 데이터를 전달하고 받아서 해석한다는 의미이다.

따라서, 데이터를 보내는 순서와 받는 순서를 일치시켜야 한다.

사용법의 예시는 다음과 같다.

```cs
public class PhotonController : MonoBehaviourPun, IPunObservable
{
    [SerializeField] int value1;
    [SerializeField] float value2;
    [SerializeField] bool value3;

    public void OnPhotonSerializeView(PhotonStream stream, PhotonMessageInfo info)
    {
        // 변수 데이터를 보내는 경우[ IsMine이 true인 경우 ]
        if (stream.IsWriting)
        {
            stream.SendNext(value1);
            stream.SendNext(value2);
            stream.SendNext(value3);
        }
        // 변수 데이터를 받는 경우[ IsMine이 false인 경우 ]
        else if (stream.IsReading)
        {
            value1 = (int)stream.ReceiveNext();
            value2 = (float)stream.ReceiveNext();
            value3 = (bool)stream.ReceiveNext();
        }
    }
}
```

<br>

`stream.SendNext(값타입 변수)`를 통해 스트림에 변수를 넣고
`stream.ReceiveNext()`를 통해 스트림에 올라온 변수를 받아올 수 있다.

<br>

##### 주의점

단, 해당 방식을 이용한 데이터 전송은 **값 형식** 데이터만 전송이 가능하다.

또한, 해당 프리팹의 소유권(IsMine)이 없을 경우, 값이 바뀌어도 변경사항을 전달하지 않는다.
하지만, 로컬 환경에서는 값이 변경된 상태이기 때문에 소유권이 없다면 변경 자체가 불가능하도록 조건을 달아두어야 한다.

<br>

### PhotonTransformView

캐릭터의 움직임 동기화는 대부분의 게임에서 필요로 한다.
따라서 포톤에서 구현되어있는 컴포넌트가 있는데 `PhotonTransformView`컴포넌트를
추가한다면 움직임과 회전을 지연보상을 포함한 동기화를 지연한다.

이것 외에도 RigidbotyView등도 제공한다.

<br>

## 소유권

퓨전에서도 나왔고 위의 동기화에서도 나온 이야기, 소유권에 대한 이야기다.

소유권은 대표적으로 아래와 같은 역할을 한다.

- 객체 동기화의 기준이 된다.
- 유효한 입력 판별의 기준이 된다.

<br>

### 기준

정보를 동기화하기 위해서는 당연히 참고할 기준이 필요하다.
기본적으로 해당 오브젝트를 생성한 클라이언트가 기준이 되므로
해당 클라이언트가 소유권을 가지게 된다.

<br>

### 입력 판별

캐릭터는 모두 같은 프리팹에서 인스턴스화되었으므로
씬에 존재하는 모든 인스턴스는 모두 입력을 받게 된다.

따라서 소유권이 없다면 해당 입력을 무시할 수 있도록 구분할 수 있는 기준이 되어준다.

<br>

해당 객체의 소유권은 **PhotonView**에 존재한다.
따라서 다음과 같은 코드를 작성해서 소유권을 판별할 수 있다.

`if(photonView.IsMine)` 소유권이 있다면 true반환

<br>

## 참조 데이터 동기화

값 형식의 동기화는 그저 값 자체를 전송하면 되므로 간단하다.
하지만 참조형식 데이터는 컴퓨터마다 저장되어있는 주소가 다르기 때문에 조금 다른 방식을 사용해야 한다.

<br>

### 포톤뷰와 ID

모든 동기화 오브젝트는 PhotonView를 가지고 있으며, 이를 이용해서 참조형식 데이터를 동기화할 수 있다.

<hr>
<br>

#### 보내는 경우

참조형 데이터를 동기화하기 위해서는 PhotonViewID가 있어야 한다고 했다.
따라서 해당 ID를 얻는 방법에 대해 알아본다.

1. PhotonView pv = collider.GetComponent<PhotonView>();<br>우선 참조형 데이터가 붙어있는 PhotonView 인스턴스를 가져온다.
2. int pd = pv.ViewID;<br>해당 PhotonView의 ID를 가져온다.

<br>

int형 변수를 얻었으니, stream을 이용하여 보내면 된다.

<hr>
<br>

#### 받는 경우

이제 스트림을 통해서 int형 ID변수를 받았을것이다.
이후 처리는 다음과 같다.

1. PhotonView target = PhotonView.Find(pd);<br>ID를 이용해 동일한 PhotonView를 찾아온다.
2. collider = target.GetComponent<Collider>();<br>대상이 되는 참조형 데이터를 가져온다.

<br>

해당 방식으로 가져온다면 같은 참조형 데이터를 사용할 수 있다.

<br>

## 함수 동기화

함수 동기화는 이전에도 보았던 RPC를 이용한다.
하지만 퓨전에서 사용했던 방식과는 약간 다르기때문에 다시 알아보도록 한다.

<br>

### RPC의 정의와 사용

퓨전과는 다르게 RPC함수의 어트리뷰트에는 단순하게 **PunRPC**키워드 하나만 들어간다.

```cs
[PunRPC]
void FireRPC(){ }
```

<br>

이후 해당 함수를 호출할때에는 PhotonView를 이용해야한다.
어느 포톤 뷰에 있는 RPC함수인지 구분할 방법이 필요하기 때문이다.

<br>

호출하는 방식은 다음과 같다.
`photonView.RPC("FireRPC", RpcTarget.All)`

첫번째 매개변수로는 string형식으로 RPC함수명을 작성한다.
다음으로는 해당 함수를 실행할 대상을 지정한다.

이렇게 원격으로 함수를 호출할 수 있으며, 3번째 이후의 매개변수로는 필요한 파라미터들을 보낼수도 있다.

<br>

### 타겟과 버퍼

잠긴 문을 여는 RPC함수를 호출했다고 해보자.
문을 연 후에 새로운 플레이어가 접속한다면 어떻게될까?

문이 안열린 상태로 있을것이다.
이것은 동기화의 취지와 맞지 않는 경우가 된다.

이럴때에는 rpc를 호출할 때 **Buffered**형식으로 호출해야 한다.
<br>

 Buffered가 붙은 타입의 RPC호출은 서버에서 기억하고있다가 해당 함수가 호출되지 않은
(예를들면 새로 들어온)클라이언트들에게 해당 함수들을 호출하게 만든다.

따라서 중요한 함수들은 기억해두었다가 반드시 호출시키는 방식의 Buffered 타겟 키워드가 있다.

<br>

### 타겟과 ViaServer

컴퓨터들의 성능과 네트워크속도에 따라서 서로 보내고 받는 순간이 어긋날수있다.

이는 RPC를 즉시 실행하기 때문이다.

이러한 현상을 보정하기 위해서 서버를 거쳐서 RPC를 실행시킬수 있다.

서버에 요청이 들어온 순서대로의 함수 실행을 보장하기 때문에 공정성을 가지게된다.

<br>

이럴때에는 타겟의 뒤에 **ViaServer**키워드를 붙여주면 된다.

<br>


## 생성과 동기화

`PhotonNetwork.Instantiate([프리팹 이름], 위치, 회전)`를 통해 네트워크 오브젝트를 만들었었다.

여기서 조금 더 이용할 수 있는 요소가 있는데, 위의 매개변수에서 해당 오브젝트를 생성할 때 매개변수를 전달해줄 수 있다.

예를들어 1,2,3이라는 배열을 생성하는 시점에서 포톤 뷰에 저장해놓을 수 있는데, 그 방법은 다음과 같다.

```cs
object[] initData = new {1, 2, 3};
PhotonNetwork.Instantiate([프리팹 이름], 위치, 회전, data : initData);
```

위와같이 object배열 형태로 Instantiate시에 저장해놓으면, 해당 값을 다음과 같이 꺼내서 사용할 수 있다.

```cs
public class PlayerController : MonoBehaviourPun, IPunObservable {

int v1;
int v2;
int v3;

	void Start(){

		v1 = (int)photonView.InstantiationData[0];
		v2 = (int)photonView.InstantiationData[1];
		v3 = (int)photonView.InstantiationData[2];

	}

}
```

<br>

위와같이 Instantiate시에 전달한 데이터를 가져와서 사용할 수 있다.

<br>


## 지연보상

네트워크의 특성상 모든 클라이언트가 완전히 동일한 타이밍에 동일한 행동을 가지는것은 불가능하다.

따라서 **사건이 보이는대로 일어나지 않는 현상**이 발생할 수 있다.

<br>

이는 게임을 부자연스럽게 만들고 몰입감을 떨어뜨리는 원인이 된다.

<br>

따라서 이런 현상을 완화시키기 위해 여러가지 완화책이 존재한다.
이번에는 이러한 기법인 **지연 보상**에 대하여 알아보자.

<br>

### 예상 대기

해당 방식은 RPC의 전달에 걸린 딜레이를 예상하여 그만큼 더 진행하거나 대기시간을 감소시키는 방식이다.

<br>

예상 대기의 실행 흐름은 다음과 같다.

<br>

1. RPC를 보내는 클라이언트에서 5초 후에 해당 함수를 실행한다고 가정한다.
2. RPC를 받는 입장에서는 해당 RPC를 받는데까지 걸린 시간 딜레이를 계산한다.
3. 수신측에서는 **5초 - 딜레이** 시간만큼 조금 더 보정해서 함수를 호출한다.

<br>

또는 총을 쏘는등의 대기가 없는 로직에서는 다음과 같이 작동한다.

1. RPC를 보내는 클라이언트에서 해당 함수를 실행한다고 가정한다.
2. RPC를 받는 입장에서는 해당 RPC를 받는데까지 걸린 시간 딜레이를 계산한다.
3. 수신측에서는 **딜레이 시간**만큼 총알의 생성 위치를 보정, 계산하여 생성한다.

<br>
<hr>

해당 방법을 위해서는 필요한 정보가 있다.

**RPC가 송신된 시간**의 정보가 필요하다.

<br>

이를 위해서 RPC함수를 정의할 때 RPC를 호출한 타이밍 등을 포함한 정보를 전달할 수 있다.
단순히 함수를 정의할 때 다음과같이 마지막에 매개변수를 전달하면 된다.

<br>

```cs
[PunRPC]
public void GameStart(PhotonMessageInfo info){}
```

<br>
<hr>

이제 해당 정보를 이용하면 딜레이시간(lag)을 찾아낼 수 있다.

`float lag = Mathf.Abs((float)(PhotonNetwork.Time - info.SentServerTime));`

<br>

이를 이용하여 작성한 예상 대기 코드는 다음과 같다.

```cs
public void RequestGameStart(){
	//본인을 포함한 모든 클라이언트들에게 전달.
	photonView.RPC("GameStart", RpcTarget.AllViaServer);
}

[PunRPC]
public void GameStart(PhotonMessageInfo info){
	float lag = Mathf.Abs((float)(PhotonNetwork.Time - info.SentServerTime));
	StartCoroutine(GameTimer(5f- lag));
}

IEnumerator GameTimer(float timer){
	yield return new WaitForSeconds(timer);
	Debug.Log("게임 스타트!");
}
```
<br>

위와 같은 방법은 대기시간이 있는 경우고, 총알을 쏘는 경우라면 다음과 같은 방식으로 코드를 작성할 수 있다.

<br>

```cs
public void RequestGameStart(){
	//본인을 포함한 모든 클라이언트들에게 전달.
	photonView.RPC("Shoot", RpcTarget.AllViaServer, pos, rot, speed);
}

[PunRPC]
public void Shoot(Vector3 position, Quaternion rotation, float speed, PhotonMessageInfo info){

	float lag = Mathf.Abs((float)(PhotonNetwork.Time - info.SentServerTime));
	position += lag * speed * (rotation * Vector3.forward);
	Instantiate(bulletPrefab, position, rotation);
}
```


<br>

### Pun2의 지연 보상

다행스럽게도 Pun2에는 어느정도 지연 보상이 적용된 PhotonView들이 작성되어있다.

rigidbody가 있는 객체는 **Photon Rigidbody View**
transform객체는 **Photon Transform View** 컴포넌트를 작성하면
어느정도 지연보상이 적용된 동기화 처리를 해준다.

<br>

다만, 가능하면 두개의 뷰는 하나만 사용하는것이 좋다.
물리와 transform은 서로 다른 라이프사이클에서 돌아가기 때문에
두 사이클 사이에서 보정치가 서로 충돌하여 물체가 덜덜 떨리는 현상이 생길것이다.

<br>

## 마스터 클라이언트와 권한 승계

호스트-클라이언트 구조의 네트워크에서는 방장이 모든 권한을 가지고 처리한다.
그리고 대부분의 로직도 방장이 중요한 처리를 하도록 구현되어있을 것이다.

<br>

그런데 방장이 나가거나 연결이 끊어진다면, 다른 클라이언트들은 어떻게될까?
저번에도 배웠듯, 방장의 권한 승계는 자동으로 이루어진다.

<br>

### 마스터 클라이언트 판별

승계까지는 자동으로 이루어진다는것을 알았으니, 방장만이 실행할 수 있는 코드를 작성해야한다.

크게 복잡할것없이 포톤에서 제공하는 속성을 이용해서 구분할 수 있다.

```cs
private void Update(){

	if (PhotonNetwork.IsMasterClient == false)
		return;

	//방장 이외에는 접근 불가.

}
```

<br>

### 룸 오브젝트

여기까지는 다 좋은데 하나의 문제가 있다. 대부분의 네트워크 오브젝트들은 방장이 생성하고 관리하는 상황이 된다.
이 경우에, 방장이 나간다면 방장 소유권의 모든 오브젝트들이 모두 사라질 것이다.

따라서 같이 사라지면 안되는 네트워크 오브젝트들의 소유권을 새로운 방장에게 옮겨주는것이 필요하다.

코드로 방장이 나가게 되면 권한을 옮기도록 작성할 수도 있겠으나, 포톤에서 이미 해당 기능을 구현해두었다.

<br>

방장이 나가더라도 사라지면 안되는 공용 네트워크 오브젝트를 **룸 오브젝트**라고 한다.

룸 오브젝트는 생성한 방장이 나가더라도 다음 방장에게 권한이 위임되며 사라지지 않는다.

<br>

네트워크 오브젝트를 룸 오브젝트로 만드는 방법은 다음과 같다.

`PhotonNetwork.InstantiateRoomObject([프리팹 경로], [위치], [회전]);`

조금 주의할점은, 해당 룸 오브젝트를 생성할 수 있는것은 **방장 클라이언트**뿐이라는것이다.

<br>

추가로, 시작부터 씬에 존재하던 네트워크 오브젝트 또한 자동으로 룸 오브젝트 취급이 된다.

<br>

### 시작된 작업의 승계

3초 주기로 아이템을 생성하는 코루틴을 작성했다고 가정하자.

게임이 시작된 순간, 마스터 클라이언트만 해당 코루틴을 실행할것이다.

그런데, 이때 방장이 나가게되면 생성 코루틴은 사라지게되고, 게임은 시작된 상황이니
해당 코루틴은 다시 돌아갈 방법이 없다.

따라서, 이러한 방식의 **시작된 경우 방장이 최초로 돌리는 로직** 특히 `코루틴`의 경우
다음 방장에게 넘겨서 계속해서 돌아가도록 해야할 것이다.

그러므로 다음과 같은 로직을 작성해둘 필요가 있다.

```cs
public class GameManager : MonoBehaviourPunCallbacks{

	void Start(){
	
		if(!PhotonNetwork.IsMasterClient)
			return;

		//방장이 처음 시작시킴.
		StartCoroutine(SpawnCo());

	}

        // 마스터 클라이언트 변경
	public override void OnMasterClientSwitched(Player newMasterClient){
        
		if (PhotonNetwork.LocalPlayer != newMasterClient)
			return;

		//새로운 방장이 다시 시작시킴.
		StartCoroutine(SpawnCo());	

	}
}
```


<br>

