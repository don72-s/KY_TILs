# 오클루전 컬링

지난시간까지 사용했던 기본 ar 기능만을 이용한 경우, 대상이 가려지지 않는 부자연스러운 현상을 확인할 수 있다.

카메라 거리에따라서 해당 물체가 가려지도록 하려면 어떤 방식을 사용할 수 있을까?

<br>

## 컬링

컬링은 화면에 출력하지 않는 부분을 선별하여 랜더링하지 않을 부분을 골라내는 과정을 의미하지만, ar에서는 조금 더 현실감있게 사용하기 위해서 사용하게 된다.

<br>

### AR Occlusion Manager

공식적으로 제공하는 정석적인 컬링 방법이다.

1. AR Session Origin의 자식에 있는 AR Camera 오브젝트에 AR Occlusion Manager 컴포넌트를 추가해준다.
2. 각 항목별로 속성을 설정하여 컬링 수준을 조정한다.

다만 해당 오클루전의 퀄리티는 카메라 기기 성능에 따라 달라질 수 있다.

<br>

### 면에 의한 컬링

사람같이 면의 정의가 힘든 물체에 가려지는 경우에는 조금 자연스러운 컬링이 힘들다.

하지만, 편평한 면에 의해 가려지는 경우에는 랜더링 과정을 응용해서 자글거리는 현상을 없앨 수 있다.

1. 면을 랜더링하는 AR Plane의 머터리얼을 불투명한 머터리얼로 교체해준다.
2. 해당 머터리얼의 속성에 들어가서 Shader 종류를 VR -> SpatialMapping -> Occlusion으로 바꾸어준다.

해당 설정을 마친다면, 평면으로 인식된 표면의 랜더링이 현재 비추고있는 현실의 요소로 덧그리기때문에 조금 더 자연스러운 컬링이 가능하다.

<br>

# 디바이스 센서

우리는 자연스럽게 카메라를 사용했지만, 사실 스마트폰의 카메라는 아무나 접근할 수 없다.

자주 본 메세지인 권한 허용 메세지에서 승인을 받아야 접근이 가능하기 때문이다.

중요한 폴더 입출력 권한, gps 정보, 카메라 등등의 정보를 활용하기 위해서는 권한 요청이 필요하므로 이를 어떻게 요청할 수 있는지 알아보자.

<br>

## 권한 요청과 매니패스트

권한을 요청하는 방법은 우리가 이력서를 제출할때처럼 양식이 정해져 있다.

이러한 양식을 작성한 파일을 매니패스트 파일이라고 하며
해당 파일의 안에 요청할 권한 리스트를 작성하여 기기에 전달하는 방식으로 권한을 요청하게 된다.

<br>

### 매니패스트 파일의 생성

Project Settings -> Player -> Publishing Settings -> Custom Main Manifest 항목을 체크한다.

그렇다면 다음과 같은 경로에 xml파일이 하나 생길 것이다.
`Assets\Plugins\Android\`

해당 파일을 수정하여 원하는 권한을 요청할 수 있게 된다.

<br>

### 권한 요청서 작성

매니패스트 파일을 생성했으니 요청서를 작성해야 한다.
기본적으로 다음과 같은 양식을 따라야한다.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest
    xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:tools="http://schemas.android.com/tools">
  <!--이 영역 사이에 원하는-->
  <!-- <uses-permission android:name="android.permission.[요청 권한 종류]"/> -->
  <!--권한 요청서를 작성-->
    <application>
        <activity android:name="com.unity3d.player.UnityPlayerActivity"
                  android:theme="@style/UnityThemeSelector">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
            <meta-data android:name="unityplayer.UnityActivity" android:value="true" />
        </activity>
    </application>
</manifest>
```

<br>

여기서 요청할 수 있는 권한의 종류는 다음 페이지에서 확인할 수 있다.

[안드로이드 권한 목록](https://developer.android.com/reference/android/Manifest.permission)

<br>
### 실제 권한 요청

사직서를 작성했어도 제출하지 않는다면 퇴사처리되지 않는다.

즉, 권한 요청서를 작성했지만, 실제로 제출하지 않는다면 권한 허용 요구창도 나타나지 않는다.

즉, 권한 요청 서류를 제출하는 코드도 작성할 필요가 있다.

```cs
using System;
using UnityEngine;
using UnityEngine.Android;

public class PermissionRequester : MonoBehaviour {

    private void Start() {

        Request(Permission.FineLocation);

    }

    /// <summary>
    /// 권한 요청
    /// </summary>
    /// <param name="_targetPermission">요청할 권한의 종류 string</param>
    public void Request(string _targetPermission) {

        // 입력받은 종류의 권한이 이미 승인되어 있다면
        if (Permission.HasUserAuthorizedPermission(_targetPermission)) {

            Debug.Log("이미 허락된 권한입니다.");
            return;

        }


        // 권한 처리(승인/거절/강한 거절)시 반응(콜백)에 대한 정보를 담는 변수 선언
        PermissionCallbacks pCallback = new PermissionCallbacks();

        // 대응 함수를 호출한다. (자동으로 pCallback이 해제될것이므로 굳이 -=는 하지 않는다.)
        pCallback.PermissionGranted += OnSuccessed;
        pCallback.PermissionDenied += OnDenied;

        // Permission 구조체에서 권한을 요청한다.
        Permission.RequestUserPermission(_targetPermission, pCallback);

    }

    void OnSuccessed(string _str) {

        //요청 성공시 호출될 함수
        Debug.Log("success");

    }

    void OnDenied(string _str) {

        //요청 거부시 호출될 함수
        Debug.Log("denied");

    }

}

```

다음 코드는 시작하자마자 gps권한을 요청하는 코드다.

앱을 시작하자마자 바로 권한을 요청하여야 해당 권한을 이용하는 코드들이 정상적으로 동작할 수 있으므로 start쪽에 필요로하는 권한들을 요청하고 시작하는것이 좋다.

<br>

### 정보 이용하기

모처럼 GPS 권한을 허용받았으니 간단하게 GPS 정보를 확인해보자.

```cs
    Coroutine gpsCoroutine = null;

    public void GPSOn() {

        if (gpsCoroutine != null)
            return;

        gpsCoroutine = StartCoroutine(GPSCycle());

    }

    public void GPSOff() {

        if (gpsCoroutine == null)
            return;

        StopCoroutine(gpsCoroutine);
        gpsCoroutine = null;

    }

    IEnumerator GPSCycle() {

        WaitForSeconds delay = new WaitForSeconds(1);

        //gps 사용 확인
        if(!Input.location.isEnabledByUser)
            yield break;

        //gps 시작
        Input.location.Start();

        //gps 예열 대기
        for (int i = 0; i <= 20; i++) {

            if (Input.location.status == LocationServiceStatus.Initializing) {

                if (i == 20) {

                    Input.location.Stop();
                    gpsCoroutine = null;
                    yield break;

                }

                yield return delay;
            }

            if (Input.location.status == LocationServiceStatus.Failed) {
                Input.location.Stop();
                gpsCoroutine = null;
                yield break;
            }

            if (Input.location.status == LocationServiceStatus.Running)
                break;

        }

        //정보를 받아옴
        while (true) {

            LocationInfo info = Input.location.lastData;
            Debug.Log($"위도 : {info.latitude}, 경도 : {info.longitude}");
            yield return delay;

        }

    }
```

- gps를 매 프래임마다 호출하는것은 상당히 부담이 간다. 따라서 코루틴을 통해 주기를 늦춰서 호출하게 한다.
- gps는 기기 환경에 따라 사용을 설정해놓지 않았을수도 있고, 기능을 불러오는데 약간의 예열시간이 필요할수도 있다. 이를 상태를 통하여 확인할 수 있으므로 어느정도 여유시간을 두어야 한다.

<br>

### Android Log Cat

안드로이드 기기에서 테스트할 때에는 Debug.Log를 이용할 수 없어 불편할 것이다.

하지만 Android Log Cat이라는 패키지를 이용하면 유니티에서 로그를 볼 수 있게 되므로 개발을 용이하게 할 수 있다.

다만, 로그는 실행 앱별로 쌓이므로, 같은 앱이라도 가장 최근에 실행된 앱으로 항목을 옮기고 로그를 봐야한다.

<br>