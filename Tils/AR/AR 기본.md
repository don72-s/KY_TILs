# AR 개요와 기본개념

AR은 증강현실을 뜻하며 현실공간에 가상의 물체를 혼합시켜 랜더링하는 기술을 뜻한다.

유니티에서 AR프로그램을 개발하기 위해서는 AR Core라는 SDK가 필요하다.

<br>

# AR 개발 환경구축

유니티에서 AR앱 개발을 위해서는 몇가지 세팅이 필요하다.

1. 안드로이드 빌드 환경 추가
2. 패키지 매니저에서 AR Foundation, ARCore XR Plugin 인스톨
3. project settings - XR Plug-in management에서 AR Core 체크
4. project settings - Player - 안드로이드 - Other Settings의 Auto Graphics API 를 체크 해제, API목록에서 Vulkan을 제거
5. project settings - Player - 안드로이드 - Other Settings의 하단의 Identification - Minimum API Level을 [android 7.0 Nugat] 이상의 버전으로 세팅(최소 요구사항)
6. project settings - Player - 안드로이드 - Other Settings의 하단의 Configuration - Scripting Backend를 IL2CPP로 설정, Target Architectures의 ARM64를 체크

<hr>

이후 빌드 테스트할 안드로이드 기기의 개발자모드를 활성화하여 USB 디버깅이 가능하도록 설정해준다.

<br>

# 유니티 월드의 AR 환경

AR환경은 기존 유니티 게임 환경보다 개방적이다.
다르게 말하자면 사용자의 행동을 제한하기가 어려워진다는 뜻이다.

AR서비스의 안정적인 제공을 위해서 어떻게 행동에 제약을 줄 수 있을지 생각해보도록 하자.

<br>

## AR Session

대상 디바이스와 통신하여 여러가지 정보들을 수집하는 역할을 한다.

카메라의 정보, 자이로센서, gps값 등의 정보를 이용할 수 있도록 도와준다.

<br>


## AR Session Origin

AR Session Origin에 의해 수집된 데이터들을 기반으로 유니티 환경을 실제 환경에 매칭시켜주는 역할을 한다.

즉, 정보를 받아와서 유니티월드와 실제 월드를 동기화 시키는 역할을 하게 된다.

또한, 해당 오브젝트는 AR Camera를 자식으로 가지게 되므로 main camera의 역할을 대체하게 되므로 기존 카메라는 지워줄 필요가 있다.

<hr>

위의 두가지 항목은 AR 프로젝트에서 필수적으로 필요한 컴포넌트들로써 반드시 존재해야 한다.

두 항목은 AR 관련 패키지를 다운로드했다면, 하이어라키뷰에서 Create -> XR 항목을 통해 생성할 수 있다.

<br>

# AR 환경의 오브젝트 제어

AR 환경에서는 카메라로 배경을 가져오기 때문에 해당 이미지를 분석하여야 현재 환경을 이용할 수 있다.

이를 위하여 환경을 분석하는 컴포넌트를 사용할 줄 알아야 한다.

사용법과 응용법에 대해 생각해보도록 한다.

<br>

## 특징점

이미지 분석 방법중 하나로, 각 픽셀에서 주변 픽셀과의 차이점이 급격하게 두드러지는 경우, 해당 픽셀을 특징점이라고 정의한다.

대표적인 특징점들의 집합으로는 윤곽선 또는 경계면을 들 수 있다.

이러한 특징점들을 모아서 이미지를 분석하여 바닥이나 건물 외벽등을 구분하는데에 도움을 줄 수 있다.

<br>

## 평면

AR에서 가장 중요하고도 자주 사용하는것이 평면 인식이다.

평평한 면을 찾아야 해당 위치에 오브젝트를 위치시키거나 상호작용을 할 ㅅ 있는 기반이 되기 때문이다.

해당 평면을 어떻게 구성할 수 있는지 알아보자.

<br>

### AR Plane

우선 평면을 이용하기 위해서는 게임 씬에 AR Plane Manager컴포넌트가 존재해야 한다.

AR Default Plane을 하나 생성한 뒤, 프리팹으로 정의시킨다.

그 뒤에 manager의 prefab항목에 방금 생성한 plane을 연결해준다.

만일 세로인식이 아닌 가로인식만 허용하고싶다면 manager의 everything 항목을 horizontal 수정해주면 된다.

이렇게 설정하고 빌드해준다면, 당연히 완벽하게 인식은 하지 못하지만, 어느정도는 평평한 면을 인식하는것을 볼 수 있다.

<br>

### 상호작용

이렇게 인식된 평면은 collider등의 여러가지 요소를 가진다.

이 말은 즉, 다른 오브젝트들과 상호작용할 수 있다는 의미를 가진다.

간단하게 rigidbody를 가진 물체는 해당 평면들을 통과하지 않는다.

<br>

## AR Raycast

raycast를 사용하는데 기본 물리의 raycast와는 조금 차이를 가지게 된다.

평면 인식을 하게 된다면, 조금 인식이 저조해서 바닥 사이에 구멍이 생긴 경우, 기본 물리의 raycast는 해당 공간을 통과한다.

하지만 AR raycast는 떨어진 평면 사이를 추측하고 보간하여 사이의 빈 공간도 평면으로 취급하기에 인식률이 좋지 않은 기기에서 평면의 일관성을 유지하고싶다면 선택지가 될 수 있다.

<br>

### AR Raycast Manager

raycast와 달리 AR Raycast를 사용하기 위해서는 AR Raycast Manager 컴포넌트가 필요하다.

해당 컴포넌트를 가져오게 된다면, 사용 방법은 일반 raycast와 거의 동일하게 사용하면 되겠다.

<br>
