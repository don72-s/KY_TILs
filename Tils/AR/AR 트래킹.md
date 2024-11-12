# 이미지 트래킹

잘 알려진 ar앱들중 카드나 qr코드를 바닥에 깔아주고 이를 기반으로 모델들이 생겨나는것을 봤을것이다.

이러한 ar앱들은 어떻게 구현한건지 알아보자.

<br>

## AR Tracked Image Manager

AR Core에서 제공하는 기능답게 해당 매니저 컴포넌트가 필요하다.

해당 컴포넌트의 속석에 다음과 같은 요소들을 넣으면 된다.

- Serialized Library : 감지할 이미지 파일
- Max Number Of Moving : 여러개의 이미지가 감지되었을 때, 생성할 최대 모델의 갯수
- Tracked Image Prefab : 감지된 이미지 위에 생성할 모델 프리

<br>

### reference image library

감지할 이미지들을 모아놓은 종류의 파일이다.
해당 파일은
Create -> XR -> Reference Image Library 파일을 생성한 뒤
해당 파일의 이미지를 설정하고 속성들을 설정해주면 된다.

- Specify Size : 현실세계에서 인식할 실제 크기 사이즈를 지정한다. 인식률이 더 좋게 만들수 있다.

<br>

## 여러개의 이미지 트래킹

기본적으로 manager를 통해서 이미지 트래킹을 사용하면 하나의 이미지와 하나의 프리팹만 생성이 가능하다.

하지만 여러개의 이미지와 프리팹을 트래킹할 수 없다.

이를 위해서는 조금 추가적인 작업이 필요하다.

```cs
public class Multi : MonoBehaviour
{
    [SerializeField]
    ARTrackedImageManager imageManager;

    [SerializeField]
    GameObject prefab1;
    [SerializeField]
    GameObject prefab2;

    private void OnEnable() {

        imageManager.trackedImagesChanged += Changed;

    }

    private void OnDisable() {

        imageManager.trackedImagesChanged -= Changed;

    }

    void Changed(ARTrackedImagesChangedEventArgs _args) {

        //새로운 트래킹 이미지가 감지되었을 때
        foreach (ARTrackedImage _img in _args.added) {

            GameObject obj = null;

            switch (_img.referenceImage.name) {

                case "Type1":
                    obj = Instantiate(prefab1);
                    break;

                case "Type2":
                    obj = Instantiate(prefab2);
                    break;

            }

            obj.transform.SetPositionAndRotation(_img.transform.position, _img.transform.rotation);
            obj.transform.SetParent(_img.transform);
        
        }

        //트래킹되었던 이미지가 변경(이동, 회전)되었을 때
        foreach (ARTrackedImage _img in _args.updated) {

            _img.transform.GetChild(0).transform.SetPositionAndRotation(transform.position, _img.transform.rotation);

        }

        //트래킹되던 이미지가 사라졌을 때
        foreach (ARTrackedImage _img in _args.removed) {
        
            Destroy(_img.transform.GetChild(0).gameObject);
        
        }


    }

}
```

업데이트의 갱신을 부모자식에게 맡기는 꼴이 되었지만, 큰 골자는 다음과 같다.

이벤트를 받아서 대응하는 행동들을 지정하는것이다.

조금만 더 신경쓴다면 오브젝트풀 형식으로 개선할수도 있겠다.

<br>


# 페이스 트래킹

이번에는 얼굴을 트래킹하는 방법에 대해 알아보자.

<br>

## 원리

기본적인 구조는 이미지 트래킹과 같다.

기본적인 얼굴 트래킹을 사용한다면 머터리얼을 수정하여 얼굴을 덮는 피부를 수정할 수 있다.

<br>

## 상세한 구성요소 조정

이미지 트래킹처럼 더욱 상세한 조정을 할 수 있다.

얼굴 트래킹은 400여개의 정점으로 이루어져있으므로, 이미지트래킹처럼 변화할때의 이벤트를 받아서 특정 정점의 위치를 알 수 있다.

이를 통해서 눈, 입등의 위치에 원하는 오브젝트나 이미지등을 배치할 수 있다.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.XR.ARFoundation;

public class FaceController : MonoBehaviour
{
    [SerializeField] ARFaceManager faceManager;

    [SerializeField] List<GameObject> cubes = new List<GameObject>(468);

    private void Awake()
    {
        for (int i = 0; i < 468; i++)
        {
            GameObject cube = Instantiate(eyePrefab);
            cubes.Add(cube);
        }
    }

    private void OnEnable()
    {
        faceManager.facesChanged += OnFaceChange;
    }

    private void OnDisable()
    {
        faceManager.facesChanged -= OnFaceChange;
    }

    private void OnFaceChange(ARFacesChangedEventArgs args)
    {
        // 추적중인 얼굴에 변경사항(위치, 회전)이 있을 때
        if (args.updated.Count > 0)     // 현재는 얼굴 하나만 적용하는 중
        {
            // ARFace를 가져와서
            ARFace face = args.updated[0];

            // 얼굴에 있는 모든 점을
            for (int i = 0; i < face.vertices.Length; i++)
            {
                // 얼굴 기준의 위치를 월드위치로 변환
                Vector3 vertPos = face.transform.TransformPoint(face.vertices[i]);

                // 생성한 큐브들을 기준의 위치로 이동
                cubes[i].transform.position = vertPos;
            }
        }
    }
}

```

<br>
