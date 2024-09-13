# CSV와 Json

게임에 영향을 주는 설정 데이터등은 어떻게 저장하는게 좋을까?

지금까지 배운 내용으로는 스크립트 내부에 작은 데이터베이스를 정의해놓고 사용했다.

하지만 여기에는 두가지 문제가 있다.

1. 소스코드를 읽을줄 알아야 한다.
2. 데이터가 수정될때마다 프로그램이 빌드되어야 한다.

따라서 이런식의 프로그램에 영향을 주고, 조정을 위해 자주 변하는 데이터가 있다면 외부 파일을 읽어오는 방식이 더욱 효율적일 것이다.

여기서는 외부의 데이터를 읽어오거나 저장하는 저장/관리 방식 대해 알아보도록 하자.

<br>

## CSV ( Comma separated values )

저장된 값들이 컴마(,)단위로 구분된 데이터들을 의미한다.
대표적인 csv데이터 구조로는 엑셀 파일을 들 수 있다.

반드시 엑셀뿐만 아니라 메모장으로 작성하더라도 다음과 같은 구조로 작성한 뒤, 확장자를 csv로 바꾸어도 정상적으로 작동한다.

```
1,2,3,4,5
6,7,8,9,10
```
[2행 5열로 이루어진 데이터 구조]
<br>
많은 기획자들은 엑셀파일로 벨런스를 조절하는 경우가 많으므로 해당 파일을 읽어오는 방법을 알아보자.

<br>

### 파일 형식 통일

당연한 이야기지만, 컴퓨터에서 xlxs파일을 읽는것는 쉽지 않다. 따라서 엑셀파일을 csv파일로 저장하고 다루기로 약속을 해야한다.

+) tab으로 구분한 파일인 .tsv파일을 사용할수도 있다.
[행의 요소에 컴마가 들어가는 경우가 있을 수 있으므로.]

<br>

### 구조 정의

우선은 csv의 기본적인 구조를 정의해놓아야한다.

해당 파일의 경로, 읽어온 데이터들을 저장하기 위한 2차원 리스트를 정의한 클래스를 하나 작성한다.

```cs
class CSV{

    public string fileName {  get; private set; }

    List< string > headIndex = null;
    List< List< string > > data = null;

    public CSV(string _fileName) {
        fileName = _fileName;
        data = new List<List<string>>();
    }

    public void SetHeadIndex(List<string> _headIndexes) {
        headIndex = _headIndexes;
    }

    public void AddDataRow(List<string> _dataRow) { 
        data.Add(_dataRow);
    }
    
}
```

첫번째 행은 보통 분류를 위한 행이므로 이를 저장할 행을 따로 나누어놓기로 한다.

<br>

### 유니티 내부 폴더의 경로

파일의 경로를 찾을 때, 고정된 스트링으로 한다면 플레이 환경마다 상이한 경로들에 대해 대응할 수 없을 것이다.

이런 상황을 위해서 유니티에서는 현재 게임이 실행중인 경로를 기반으로 데이터를 찾아올 수 있도록 특정한 폴더 경로를 얻을 수 있는 방법을 제공한다.

- **Application.dataPath** : 현재 프로젝트의 **Assets**폴더의 경로를 반환하며, 에디터에서 게임을 제작중일 때 에디터 내의 파일을 읽어올 때 사용한다. **빌드가 완료된 앱에서 열기는 힘들다**
- **Application.persistentDataPath** : 대상 기기별로 사용하는 사용자의 **개인 로컬 데이터 저장소**의 위치 경로를 가져온다. 즉, 플레이어의 데이터를 저장하거나 불러올 때 사용하기 유용하다.

<hr>

개발과 빌드된 상황을 구분하기 위해서 **#if**로 구분해놓는 방식이 일반적이다.

```cs
#if UNITY_EDITOR
    path = Application.dataPath;
#else
    path = Application.persistentDataPath;
#endif
```

<hr>

#### 경로의 병합

두가지 파일 경로를 더할때에는 그냥 문자열의 +등을 사용해도 괜찮지만 다음과 같은 방법으로 더한다면 자동으로 \를 추가해주게 된다.

`Path.Combine(string1, string2...)`

<br>

### 파일 읽어오기

파일의 구조는 정의해놓았으니, 해당 자료에 데이터를 저장해야 할 것이다.

따라서 파일을 읽어오는 역할을 하는 클래스를 새로 하나 정의한다.

```cs
public static class Load
{

    public static void LodeCSV(CSV _csvData) {
    
        string filePath;

        //저장경로는 환경에 따라 수정.
        filePath = Path.Combine(Application.dataPath, "DataTables",  _csvData.fileName + ".csv");

        //파일 존재 확인
        if (File.Exists(filePath)) {

            string csvFile = File.ReadAllText(filePath);

            string[] lines = csvFile.Split('\n');

            //마지막은 공백이므로 반복 제외
            for (int l = 0; l < lines.Length - 1; l++) {

                //각 행의 요소 분리
                string[] fields = lines[l].Split(',');
                List< string > datas = new List<string>();

                for (int f = 0; f < fields.Length; f++) {
                    datas.Add(fields[f]);
                }

                //데이터 리스트 추가
                if(l == 0)
                    _csvData.SetHeadIndex(datas);
                else
                    _csvData.AddDataRow(datas);

            }

        } else {

            Debug.Log("file no exist");

        }
    }
}
```

뭔가 길어졌지만 사실상 별건 없다.

1. 모든 문자열을 읽어온다.
2. \n을 기준으로 여러개의 문자열로 나눈다.
3. ,을 기준으로 여러개의 문자열로 나눈다.
4. 나눠진 데이터들을 행별로 구분하여 저장한다.

<br>

### 데이터의 저장

받아온것은 모두 string이므로 데이터를 사용하기 위해서는 맞는 타입의 데이터로 변환하는 등의 작업을 해주어야 한다.
[ 파일 내부에서 실수로 데이터 타입을 잘못 작성한 상황은 예외로 한다.]

데이터 저장을 위해 다음과 같은 구조체를 하나 정의하자.
```cs
struct weapon {
    public int idx;
    public string name;
    public int attack;
    public int dfe;
    public string description;
}
```
**여기서는 편의를 위해 구조체로 선언했지만, 데이터가 너무 많아짐에 따라 스택 영역이 줄어듦이 걱정된다면 클래스로 정의해도 괜찮다.**
<br>
 이제 각 종류별로 데이터를 형변환해주면 된다.

```cs
List<weapon> weapons = new List<weapon>();

foreach (var item in data) {

    weapon tmp = new weapon {
        idx = int.Parse(item[0]),
        name = item[1],
        attack = int.Parse(item[2]),
        dfe = int.Parse(item[3]),
        description = item[4]
    };

    weapons.Add(tmp);

}
```

큰 문제가 없다면, weapon타입들이 제대로 넘어갔을 것이다.

여기서는 index를 포함해서 List에 저장을 했지만, 이러한 고유 키 역할을 하는 index 열이 있는 테이블이라면 Dictionary형태로 저장하는 것이 여러 방면으로 효율적일 것이다.
<br>

### 예시로 사용한 csv

위의 설명에서 사용한 예시 csv파일은 다음과 같다.

```
index,name,atk,dfe,description
0,초보자의 단검,10,0,시작시 받는 가장 기초적인 무기
1,장검,30,0,전사가 쓰는 장검
2,활,25,20,궁수가 쓰는 활
3,지팡이,20,5,마법사가 쓰는 지팡이
```

<br>

## 직렬화

데이터를 전송/저장하기 위해서는 결국 데이터를 선형적으로 변환해야한다.

데이터를 주고받을때에는 아주 낮은단계까지 가보면 빛이나 전기신호에 의해 0 또는 1을 주고받기 때문에 결국 모든 데이터를 선형적으로 바꾸는 과정이 필요하다.

따라서 데이터를 직렬화한다는것은 단순히 이야기하면, 선형적인 구조가 아닌 데이터를 **텍스트화**시킨다고 볼 수 있다.

이렇게 저장된 텍스트(선형적 데이터)를 가져와서 역직렬화를 한다면 기존의 데이터구조를 얻어올 수 있다.

<br>

### 유니티 인스펙터와 직렬화

기본적으로 구조체와 클래스를 선언하면 인스펙터 상에서 접근하거나 수정할 수 없다.
당연한 이야기지만, 직렬화되어있지 않은 데이터구조이기때문에 후술할  json을 통한 데이터 전송/저장도 불가능하다.

따라서 해당 자료구조는 명시적으로 직렬화한 데이터 이라 라고 선언할 필요가 있는데, 해당 용도로 사용되는 어트리뷰트가

`[System.Serializable]`가 되겠다.

<br>

## Json ( JavaScript Object Notation )

json이란 엄밀히 말해서 자바스크립트에서 사용하던 데이터 구조인데, 사용하다보니 데이터 형태적으로 직렬화 구조로 사용하기 좋아서 유명해진 직렬화 자료구조이다.

<br>

### 기본 형식

클래스와 json의 형태를 비교해보면 다음과 같다.

```
[System.Serializable]
class Data{

    string name = "이름";
    int level = 1;
    float speed = 0.5f;

}

===================================
//json
("name" : "이름", "level" : 1, "speed" : 0.5)

```

보면 알겠지만 직관적으로 **"변수명" : 대응값**형태의 문자열 형식으로 반환해준다.

<br>

### json의 직렬화, 역직렬화

그렇다면 json을 만들고 json으로부터 데이터를 불러오는 법을 알아야 할 것이다.

사실 이 방법은 생각보다 간단하다.
`JsonUtility.ToJson([직렬화할 인스턴스]);`
`JsonUtility.FromJson<[역직렬화할 객체 타입]>([json 문자열]);`

ToJson으로 json 문자열을 생성한다.
문자열은 파일에 쓸 수 있으므로 여러가지 방법으로 파일에 써주면 된다.
ex)`File.WriteAllText(파일 경로,jsonString);`

반대로 읽어올때에는 저장했던 객체 타입, 저장했던 json문자열을 지정해주면 해당 객체 인스턴스로 변환하여 반환해준다.

<br>

### 직렬화와 보안성

보면 알겠지만, 민감한 정보를 json형태 그대로 저장하거나 통신을 이용하여 주고받는것은 위험하다.

정보가 있는 그대로 외부에 노출되기 때문이다.

따라서 민감한 정보는 json방식으로 다루지 않는것을 매우 추천하며, 보여지는것은 괜찮더라도 변조되면 안된다면 해시값 체크를 하는등의 처리가 필요할 것이다.

이러한 특성 때문 암호화 복호화 과정을 통해 외부자가 볼수 없도록 하는것이 정석으로 여겨지고 있다.

<br>