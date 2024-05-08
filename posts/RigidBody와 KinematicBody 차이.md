---
title: RigidBody와 KinematicBody 차이
created: 2024-03-19 14:32
last-updated: 2024-03-19 14:32
tags:
  - gameDev
  - godot
  - toyproject
description: 노드가 캐릭터를 밀어낼 수 있다면 RigidBody이다
---

## 👯‍♂️ Rigidbody? Kinematic Body?

Godot에서 쓰이는 용어들에 대해서 조금씩 익숙해져가고 있다. 

- rigidbody란 한국어로 강체, 물리학에서 형태가 고정되어 변하지 않는 물체를 가리킨다. 강체는 외력이 가해져도 모양이나 크기가 변형되지 않는다.(위키출처) 유니티에서 쓰인느 리지드 바디는...  게임 오브젝트의 물리적 동작을 가능하게 하는 주요 컴포넌트입니다. 리지드바디가 연결된 오브젝트는 중력에 즉시 반응합니다.. 흥미로운건 고닷에서도 리지드 바디는 스크립트로 다이렉트로 컨트롤하면 안되고 이 물체에 물리엔진을 통해 우밎ㄱ이도록해야한다(중력, impulses등)


- 한편 키네마틱바디는 유니티에서 리지드바디 2D는 시뮬레이션에서 매우 명시적인 사용자 제어 하에서만 움직이도록 설계됩니다.동역학 리지드바디 2D는 중력과 힘의 영향을 받지만, 키네마틱 리지드바디 2D는 그렇지 않습니다.따라서 키네마틱 리지드바디 2D는 동적 리지드바디 2D보다 시스템 리소스를 덜 소모해서 더 빠르게 시뮬레이션할 수 있습니다. < 라고 표현되어있다, 

- 한편 캐릭터바디는 캐릭터를 위해 만들어진거임 그리고 충돌이나 반응과정이 스크립트로 반드시 구현되어야햄


![](https://i.imgur.com/Fd5yAqe.png)




## 👯‍♂️ 결론

- `내가 만들고 싶어하는 노드가 캐릭터 밀어낼 수 있는가?` 를 질문해보자!
	- 예스: -> rigidbody
	- 놉! -> characterbody
- 이 노드를 무엇이 움직이게 하는가 물리엔진이라면 Rigidbody이고  이 노드를 움직이는게 스크립트라면 CharacterBody




## 👯‍♂️ Ref & LINKS TO THIS PAGE

-  [웹,  godot tutorial](https://docs.godotengine.org/en/4.1/tutorials/physics/physics_introduction.html)
