# 핸드 트래킹

최신 VR은 손의 움직임을 직접 분석하고 이용한다.
이를 어떻게 사용할 수 있을까?

<br>

## 사전 준비.

우선 vr의 설정에 들어가서 트래킹 항목에서 핸즈 트래킹을 켜주어야 한다.

다음으로 유니티의 Package Manager > XR Hands 패키지를 다운로드 한다.

Project Settings > XR Plug-in Management > OpenXR항목으로 들어간 뒤 Enableed Interaction Profiles에 Microsoft Hand Interaction Profile과 Meta Quest Ttouch Pro Controller Profile, Oculus Touch Controller Profile 세가지를 추가해준다.

다음으로 하단의 Hand Interaction Poses, Hand Tracking Subsystem, Meta Hand Tracking Aim 세가지 항목의 체크박으를 체크해준다.

여기까지 핸드 트래킹을 이용하기 위한 환경 설정이다.

<br>

## 제스쳐 등록

핸즈 파일을 생성해서 각 값들을 통하여 제스쳐를 정의할 수 있다.

또 해당 제스쳐를 이용하기 위해서는 Static Hand Gesture컴포넌트를 등록해서 이용하면 된다.

마지막으로 해당 손을 계속 추척하며 상태를 확인해야 하기 때문에 XR Hand Tracking Events 연결이 필요하다.

<br>