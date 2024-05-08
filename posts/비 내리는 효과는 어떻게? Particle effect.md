---
title: 비 내리는 효과는 어떻게? Particle effect
created: 2024-03-19 10:23
last-updated: 2024-03-19 10:23
description: godot에서 particle effect구현해보자
tags:
  - gameDev
  - godot
  - toyproject
---
```ad-note
title: 키워드
- particle effect
- opeGL
- METAL API  
```


## 👯‍♂️ 비 효과도 OOP로 구현 가능??;

내 생각에는 
- 빗방울 하나 ==  객체
- 내리는 여러개의 빗방울(=비) == 인스턴스들

원래는 "아하! 그러면 빗방울 인스턴스를 여러개 만들면 되겠구나 " 라는 생각을했다



아래는 내 행복회로

1. 빗방울 객체 만든다(고닷에서는 빗방울 하나를 scene으로 만들고 이를 인스턴스화해서 사용 할 계획이었다)  
    그리고 아래 RainDropBackground를 Main 씬에서 넣었음
    

```
- RainDropBackground(Node2D)
|- Raindrop(RigidBody)
||- AnimationSprite2D
||-CollisionShape2D
```

2. 0.25초마다 동시에 10방울이 동시에 떨어진다. → Timer 노드를 사용. 0.25초 타임아웃때마다 빗방울객체 10개씩 만드는 함수 호출(0.25초마다 빗방울이 떨어진다)
    
3. 각 각의 방울들 인스턴스 속도는 랜덤하게 조정한다 근데 여기서 아래 두 가지를 고민했다
    
    1. 첫번째 이미지→ 만약에 Raindrop 스크립트에서 객체자체에 만들고 속도를 랜덤하게 만들면? → 각 각의 물방울들 속도가 똑같아보인다
        
    2. 두번째 이미지 → 만약에 RaindropBackground 스크립트에서 인스턴스 만들 때마다 속도를 변화시키면? → 각 각의 물방울들 속도가 달라진게 보인다

| 객체 자체에 속도부여시(=모든 인스턴스 속도 같음)         | 각 인스턴스들의 속도가 다를경우                    |
| ------------------------------------ | ------------------------------------ |
| ![](https://i.imgur.com/gM16tKh.png) | ![](https://i.imgur.com/B3Evupq.png) |

파이썬으로 표현하자면 이런 느낌이다

```python

################## 첫 번째 시도 (객체 멤버변수에 직접적으로 랜던값 부여)
  
class Drop2:
  velocity = 0

drops = []
for i in range(0,10):
  temp = Drop2()
  # Drop2의 모든 인스턴스에 공통적으로 적용됨
  Drop2.velocity = random.random()
  drops.append(temp)

################## 두 번째 시도 (객체 만들 때마다 랜덤값 부여 )

import random
class Drop:
  def __init__(self):
  # `Drop` 인스턴스는 서로 다른 `velocity` 값을 가지게 된다
    self.velocity = random.random()

for i in range(0,10):
  print(Drop().velocity) 

```


하지만 이러나 저러나 아무리 구현해봐도 빗방울 떨어지는 모습이 비같지가 않고 쌀 알처럼 무겁게 느껴졌다 그래서 찾아보니 파티클 효과란게 있었다; 



## 👯‍♂️ 쉽고 빠른길; Particle Effect

```ad-important
title: 파티클 효과란?

입자 형태의 이미지를 사용하여 게임 화면에 만들어내는 화염, 폭발, 연기 등과 같은 효과  
- 유니티에서 파티클 시스템이란, 파티클(Particles) 은 파티클 시스템에 의해 큰 숫자로 표시되고 이동되는 작고 단순한 이미지 또는 메시입니다. 각 파티클은 유체 또는 비정질 엔티티의 작은 부분을 나타내며, 모든 파티클의 효과가 전체 엔티티의 느낌을 만듭니다. 예를 들어 연기 구름을 사용하면 각 파티클마다 작은 구름과 비슷한 작은 연기 텍스처가 생깁니다. 이러한 소형 구름이 씬 영역에 다수 배치되면 전체적인 효과는 더욱 부피가 큰 구름이 형성됩니다.  
- 그러니까 비라든가 불났을 때 연기구름 같은건 파티클이란걸 이용한다!
```

그리고 Godot에서도 이러한 효과를 제공해준다

![](https://i.imgur.com/ZpxgEZt.png)



### 하지만 m1을 사용 중이라면 Godot CPUParticle만 사용가능

<mark style="background: #FFF3A3A6;">GPUParticle2D와</mark> CPUParticle2D 노드가 존재하는데 애플 실리콘칩을 탑재한 M시리즈에서는 CPUParticle만 사용가능하다.. 튜토리얼에서는 GPUParticle2D로 해서 나도 할려고 했는데 고닷 프로젝트가 자꾸 멈춰서 놀랬었음.

찾아보니 애플 실리콘 맥에서는 OpenGL 드라이버가 빠져있고 저 GPUParticle은 OpenGL을 사용하고있다. 그리고 Godot4.2 stable version Compatibility에서는 OpenGL이 디폴트로 설정되어있어서 그렇다. 사용자가 실제로 OpenGL이 있는 것과 무관하게 디폴트로 이게 설정되어있어서 수정해달라고 이에 대한 [논의](https://github.com/godotengine/godot-proposals/issues/8006)도 진행 중인 것으로 보인다 ㅜㅜ (240125기준)

> Godot 4 currently supported Vulkan for its higher end RenderingDevice API (Forward+ and Mobile rendering methods), and OpenGL 3.3 for its Compatibility rendering method.
> 
> The Project Manager defaults to OpenGL, so it works on most platforms. But then if the user's GPU / drivers don't support Vulkan, the process is subpar

근데 openGL이 뭐지?

> 3D 게임 만들때 어느 사물의 질감, 그림자등을 표현 할 때 복잡한 컴퓨터 수식같은게 필요한데 이러한 연산들을 대신 해주고 하드웨어를 제어할 수 있도록 도와주는 라이브러리. 이런 라이브러리 사용하므로써 그래픽 디자이너들은 계산에 대한 저차원적인 고민보다는 디자인 작업에 더 많은 시간을 할애할 수 있다! 예를 들면 공 하나 만들고 싶다면 공 좌표와 텍스처 조명효과등을 계산했어야하는데 OpenGL이 제공하는 함수를 사용해 필요한 계산들을 처리 할 수 있다고함..

그리고 메탈이 뭐지?

> 2019년에 애플에서는 더 이상 OpenGL을 지원하지 않고 애플에서 자체 제발한 컴퓨터 그래픽 라이브러리 Metal API를 사용하겠다고 발표함. 이 메탈은 애플 기기를 위해 최적화된 그래픽 레이어를 제공한다고한다. 더 이상 OpenGL을 사용하지 않고 Metal을 사용하겠다고 한 것은 애플 플랫폼에서 동작하는 게임이나 그래픽을 많이 사용하는 어플을 만들려면 Metal을 사용해야한다는 뜻임 그렇지 않으면 Opengl → Metal로 번역하는 방법을 찾아야한다는 뜻 + 그리고 OpenGL을 사용하는 앱 또한 애플 기기에서 사용 할 수 없단느 뜻을 의미함



## 👯‍♂️ Ref & LINKS TO THIS PAGE

-  [유튜브, godot에서 파티클 효과 구현]([https://www.youtube.com/watch?v=W5lR6Q31VFQ](https://www.youtube.com/watch?v=W5lR6Q31VFQ))
- [웹, godot 공식문서]([https://docs.godotengine.org/en/stable/tutorials/2d/particle_systems_2d.html](https://docs.godotengine.org/en/stable/tutorials/2d/particle_systems_2d.html))