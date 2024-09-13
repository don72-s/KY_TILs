# Character Controller 컴포넌트

3D 환경에서 캐릭터의 이동에서 유용하게 쓸 수 있는 컴포넌트가 있다.
Character Controller 라는 컴포넌트를 이용할 수가 있는데 해당 컴포넌트를 통한 이동방식은
transform, rigidbody를 통한 이동과는 조금 다른 원리를 기반으로 한다.

<br>

## 움직임의 원리

그렇다면 transform, rigidbody를 사용하지 않고 어떻게 이동을 구현했을까?

정답은 충돌체를 이용한 이동이다.

기본적으로 Character Controller 컴포넌트는 개인적인 Chapsule형태의 콜라이더를 가지고 있다.
즉, 해당 충돌체를 기반으로 작동한다는 이야기가 된다.

이동이 가능한지의 여부는 내부 속성에 따라서 달라지므로 각 수치를 조절해주면 된다.

<br>

### 움직이기

내부 속성을 직접 건드리기는 힘들지만, 컴포넌트에서 제공하는 함수를 사용하면 된다.

```cs
CharacterController cc;

cc.Move([이동 방향벡터]);
```

해당 함수를 실행하면, 주변 충돌체 상황을 분석하여 이동 효과를 제공하게 된다.

<br>

### 속성

Character Controller 컴포넌트에는 이동 구현에 유용한 속성들이 있다.

- Slope Limit : 이동할 수 있는 최대 경사면의 각도
- Step Offset : 오를 수 있는 최대 단차 높이
- isGrounded : 해당 캐릭터가 바닥을 밟고있는 상태인지를 나타낸다

<br>

## 유의점

Character Controller 컴포넌트는 Collider를 기반으로 설계되었으며 rigidbody가 없다.
즉, 중력의 영향을 받지 않기 때문에 떨어지는 기능을 직접 구현해주어야한다.

보통은 Update문 내부에 다음과 같이 작성하게 된다.

```cs
float ySpeed;

void Update(){

	if( cc.isGrounded){

		ySpeed = 0;

	}else{

		ySpeed -= Physics.gravity.y * Time.deltaTime;
		cc.Move(Vector3.down * ySpeed * Time.deltaTime;
		
	}

}
```

<hr>

#### isGrounded는 거짓말쟁이

isGrounded는 현재 캐릭터가 땅을 밟고있는지를 반환한다.
하지만 신뢰성이 정말 떨어지는 속성...이다

따라서 Raycast를 통해서 중력 효과와 같이 **직접 구현** 하는것이 일반적이다.

<br>

