# VR 개요

VR은 AR과 무엇이 다른지 간단하게 알아보자.

<br>

## 개념과 방식

AR은 현실 세계에 가상의 물체를 덧대어 표현하는 방식이라면
VR은 모든 구성요소를 가상의 구성요소로 구현하며 AR에 비하여 컨텐츠 구현의 제약이 비교적 적은 편이다.

<br>

## VR 멀미

VR은 가상세계를 인지하게 만드는 방식이기 때문에 현실세계와의 괴리감으로 인한 멀미가 발생할 가능성이 매우 높다.

따라서 VR컨텐츠를 구현할때에는 상당히 중요한 고려사항이 된다.

보통 정적으로 제자리에서 행동하는 프로그램들로 개발하는것이 정석이다.

<br>

# VR 개발환경 설정

VR프로그램 개발을 위해서는 AR때처럼 환경을 먼저 구성해야한다. 천천히 차근차근 진행해보도록 하자.

<br>

## VR 기기 설정

안드로이드기기때처럼 스토어에 있지 않은 앱을 테스트하기 위해 알수없는 앱 설치를 허용해야 한다.

따라서 해당 앱 설치를 허용하주는 과정을 거쳐야 한다.

<br>

## 유니티 프로젝트 설정

AR의 경우와 같이 VR개발을 위해서 별도의 설정이 필요하다.

- PackageManger > Unity Registry > VR 패키지를 다운로드한다.
- 설치가 완료되면 Input System을 적용하기 위해 에디터가 자동으로 재시작된다.
- 다음으로 XR Interaction Toolkit 패키지를 설치한다.(상호작용) [ 필요에 따라 samples의 starter assets과 xr device simulator, spatial keyboard등을 임포트한다. ]
- 또 XR hands 패키지도 설치한다. (핸드 트래킹)

- Project Settings > XR Plug-in Management > OpenXR을 체크해준다. ( oclulus가 아닌 이유는 openXR에 핸드 트래킹이 있기 때문)
- Project Settings > XR Plug-in Management > OpenXR > Project Validation 항목에서 경고사항을 읽어보고 필요에 따라서 수정해준다. (권고사항)
- 해당 항목을 사용하기 때문에 VR기기 설정에서 OpenXR 런타임을 사용함으로 설정해주면 에디터와 VR기기 실행이 동기화된다.

<br>

