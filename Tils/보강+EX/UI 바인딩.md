# 보강 ex


## UI 관리

UI를 그리고 설계하는데 효율적으로 관리하는 방법을 알아보자.

<br>

### UI 레이아웃

UI를 배치할 때에 규칙에 따라 배치해야 하는 경우, UI 레이아웃 컴포넌트를 이용할 수 있다.

- **Vertical/Horizontal Layout Group** : 자식 오브젝트들을 (세로/가로) 로 일정한 간격들을 유지하도록 배치해준다.
- **Grid Layout Group** : 자식 오브젝트들을 바둑판 형식으로 일정한 간격으로 배치해준다.
- **ContentSizeFitter** : 텍스트와같은 크기가 유동적으로 변해야 하는 오브젝트들의 크기를 자동적으로 조절해주는 역할을 한다.

<br>

### UI 바인딩

UI는 생각보다 많은 오브젝트들이 계층형태를 이루게 되므로 찾아가는데 생각보다 시간이 걸린다.

그런데, 내부의 버튼에 연결된 함수의 이름등이 수정된다면 연결이 풀리게 된다.

이러한 상황을 효율적으로 관리하기 위한 기법이다.

***

우선 UI의 최상단 부모역할을 하는 오브젝트가 자식 UI를 관리하는 느낌의 구조가 된다.

따라서 부모 UI가 가질 Script를 하나 작성한다.

```cs
public class BaseUI : MonoBehaviour{

    //탐색용 딕셔너리
    Dictionary<string, GameObject> gameObjectDic;
    Dictionary<(string, System.Type), Component> componentDic;

    protected virtual void Awake(){
        Bind();
    }
    
    protected void Bind(){
        Transform[] trans = GetComponentsInChildren<Transform>(true);
        gameObjectDic = new Dictionary<string, GameObject>(trans.Length << 2);//딕셔녀리 여유공간 포함 할당.    
        componentDic = new Dictionary<(string, System.Type), Component>();

        foreach(Transform _child in trans){
        if(!gameObjectDic.TryAdd(_child.gameObject.name, _child.gameObject)){
            Debug.Log($"이름이 겹침! : {_child.gameObject.name}");
            }
        }        

    }
    
    //UI오브젝트 이름으로 찾아오기
    public GameObject GetUI(in string _name){
        gameObjectDic.TryGetValue(_name, out GameObject ret);
        return ret;
    }
    
    //이름이 _name인 UI오브젝트에 있는 T타입 컴포넌트 가져오기
    public T GetUI(in string _name) where T : Component{
        
        (string, Type) key = (_name, typeof(T))";
    
        if(!componentDic.ContainsKey(key)){
        
            gameObjectDic.TryGetValue(_name, out GameObject obj);
            if(obj == null) return null;

            T comp = obj.GetComponent<T>();
            if(comp == null) return null;
            
            componentDic.TryAdd(key, comp);            
        }
        
        return componentDic[key] as T;

    }
    
}
```

다만 딕셔너리로 관리하기 때문에, 자식 오브젝트들 중 이름이 겹치는 UI오브젝트가 있으면 안된다.

<br>

이렇게 완성되었다면, BaseUI클래스를 상속받아 UI별로 다루도록 할 수도 있다.

따라서 해당 클래스를 상속받아서 이용하면 되겠다.

<br>

### 포인터 핸들링

사용자와 상호작용을 위해서 Button UI를 많이 사용한다.

그런데 버튼이 아니더라도 다른 UI가 마우스(터치)와 상호작용하는 방법이 존재한다.

`IPointer`류의 인터페이스를 구현하면 된다.

- **IPointerClickHandler** : 대상이 눌렸을 때 호출
- **IPointerEnverHandler** : 대상의 영역에 마우스가 들어오면 호출
- **IPointerExitHandler** : 대상의 영역에서 마우스가 나갈 때 호출
...등등 여러가지 종류의 인터페이스들이 존재한다.

<br>

필요에 따라 인터페이스를 상속하고 대응하는 함수를 구현하면 해당 이벤트가 발생 시 구현된 함수를 호출하게 된다.

<br>

### 3D와 포인터 핸들링

IPointer 인터페이스들은 기본적으로 2D에서 동작한다.

동작하는 이유는 Canvas에서 raycast를 통하여 IPointer가 구현된 UI 오브젝트에 알림을 보내기 때문이다.

그렇다면 해당 역할을 하는 친구가 있다면 3D에서도 사용할 수 있지 않을까?

<hr>

해당 역할을 해주는 컴포넌트가 **Physics Raycaster**이다.

해당 컴포넌트를 MainCamera에 달아준다면 2D에서와 같이 IPointer 인터페이스들을 이용할 수 있다.

다만, 주의해야할 점은 매개변수에서 delta등을 이용할 때이다.

해당 매개변수는 화면에서의 변화량을 반환하므로, 2D화면에서 움직이듯이 X와 Y값만 변경될 것이다.

따라서 다음과 같은 보정이 필요하다.

```cs
Vector3 pos;

public void OnBeginDrag(PointerEventData eventData){
    pos = transform.position;
}

public void OnDrag(PointerEventData eventData){
    //평평한 plane을 이용, vector는 표면의 normal벡터
    //pos는 기준 위치
    Plane plane = new Plane(Vector3.up, pos);
    //카메라상 마우스 위치에서 ray를 발사.
    Ray ray = Camera.main.ScreenPointToRay(Input.mousePosition);
    //plane으로 ray를 발사했을 때 부딛히는지 여부
    if(plane.Raycast(ray, out float distance)){
        //해당 위치로 이동시킴
        transform.position = ray.GetPoint(distance);
    }
    
}
```

<br>

## UI 바인딩과 포인터 핸들링

IPointer, IDrag등을 이용하기 위해서는 각 오브젝트에 부착해야 한다. 그렇다면 UI바인딩을 한 이유가 없어진다.

따라서 조금 다른 방식으로 인터페이스를 관리할 필요가 있다.

<br>

### 어뎁터 클래스 작성

IPointer, IDrag를 확인하기 위한 코드를 우선 작성할 필요가 있다.

해당 코드는 어뎁터의 역할을 하게 된다.

```cs
public class EventReceiver : MonoBehaviour
    ,IPointerClickHandler
    ,IPointerEnterHandler
    ,IPointerUpHandler
    ,IDragHandler
    //...이하 생략{

        public event UnityAction<PointerEventData> OnClicked;
        public event UnityAction<PointerEventData> OnEntered;
        public event UnityAction<PointerEventData> OnUped;
        public event UnityAction<PointerEventData> OnDraged;
        //...이하 생략

        public void OnPointerClick(PointerEventData eventData) {
            OnClicked?.Invoke(eventData);
        }
        public void OnPointerEnter(PointerEventData eventData) {
            OnEntered?.Invoke(eventData);
        }
        public void OnPointerUp(PointerEventData eventData) {
            OnUped?.Invoke(eventData);
        }
        public void OnDrag(PointerEventData eventData) {
            OnDraged?.Invoke(eventData);
        }
        //...이하 생략

    }
```

해당 스크립트가 붙는 오브젝트는 마우스 클릭과 드래그등의 이벤트를 받을 수 있게 된다.

<br>

### 바인딩 스크립트 수정

대응 이벤트를 동적으로 추가, 관리하기 위해서 다음과 같은 방식으로 이벤트 추가, 제거하는 코드를 작성할 수 있다.

```cs
enum EventType{CLICK, ENTER, UP, DRAG, .....}

public void AddEvent(in string _name, EventType _type, UnityAction<PointerEventData> _callback){

    gameObjectDic.TryGetValue(_name, out gameObject gameObject);
    if(gameObject == null) return;
    
    //있으면 가져오고 없으면 추가
    EventReceiver receiver = gameObject.GetOrAddComponent<EventReceiver>();

    switch(_type){
    
        case EventType.CLICK:
            receiver.OnClicked += _callback;
            break;
        case EventType.ENTER:
            receiver.OnEntered += _callback;
            break;
        case EventType.UP:
            receiver.OnUped+= _callback;
            break;
        case EventType.DRAG:
            receiver.OnDraged+= _callback;
            break;
            //...이하 생략

    }

}

public void RemoveEvent(in string _name, EventType _type, UnityAction<PointerEventData> _callback){

    gameObjectDic.TryGetValue(_name, out gameObject gameObject);
    if(gameObject == null) return;
    
    //있으면 가져오고 없으면 추가
    EventReceiver receiver = gameObject.GetOrAddComponent<EventReceiver>();

    switch(_type){
    
        case EventType.CLICK:
            receiver.OnClicked -= _callback;
            break;
        case EventType.ENTER:
            receiver.OnEntered -= _callback;
            break;
        case EventType.UP:
            receiver.OnUped-= _callback;
            break;
        case EventType.DRAG:
            receiver.OnDraged-= _callback;
            break;
            //...이하 생략

    }

}


```

다음과 같이 코드를 작성해놓는다면 클릭, 드래그등의 대응 이벤트를 코드에서 효율적으로 추가할 수 있다.

<br>

