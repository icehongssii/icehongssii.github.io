---
title: sprite sheet로 만들어 놓자
created: 2024-03-19 10:43
last-updated: 2024-03-19 10:43
tags:
  - gameDev
  - toyproject
description: sprite sheet형태로 애셋관리시 성능도움
---
```ad-note
title: 키워드
- 스프라이트
- 스프라이트 시트 사용이유
```

## 👯‍♂️ 스프라이트란?

- 게임 개발에서 '스프라이트(Sprite)'는 주로 2D 비트맵 이미지 또는 애니메이션을 지칭하는용어
- 게임 내의 캐릭터, 물체, 배경 요소 등을 표현하기 위해 사용됨
- (게임 내 아트 에셋으로 이해했다)

![](https://i.imgur.com/QS9pcEx.png)

## 👯‍♂️ 근데 스프라이트 시트를 왜 쓰는거지?

 캐릭터 sprite sheet이라고 하면 아래 표에서 첫번째 사진처럼 하나의 파일에 캐릭터의 여러가지 동작을 프레임단위로 구성한 것을 의미한다.. 근데 왜 캐릭터 스프라이트 시트를 사용해야하는가? 여러개의 이미지파일로 한 동작을 구현하는 것보다 하나의 이미지 파일에서 여러개의 동작을 표현하는게 메모리와 퍼포먼스 측면에서 좋기 때문이었다..

> A sprite sheet is a large image that contains multiple smaller images, called sprites, that are used for animation or graphics in games. [A sprite sheet can reduce the number of files and memory needed to load and display the sprites, as well as improve the performance and efficiency of the game1](https://www.artstation.com/marketplace/game-dev/assets/2d/sprite-sheets)[2](https://opengameart.org/). [A sprite sheet can be created manually or with a tool like TexturePacker3](https://www.gamedeveloper.com/design/texturepacker---sprite-sheets-creator-for-your-games), which can automatically pack and optimize the sprites into a single image.

여튼 보통 스프라이트 시트는 이런식으로 구성되어있어서 뛰는 것만 나름 만들어 보았는데

| 정상적인 스프라이트시트                             | 내가 만든 스프라이트 시트. 간격이 엉망진창이다 딱 봐도          | 고닷에서 자를 수 있다                             |
| ---------------------------------------- | ---------------------------------------- | ---------------------------------------- |
| ![](https://i.imgur.com/hv7qsla.png)<br> | ![](https://i.imgur.com/aPFf1k2.png)<br> | ![](https://i.imgur.com/ULyt6gb.png)<br> |


각 프레임들이 중앙 정렬되어있지 않고 간격도 엉망진창이라 실제로 뛰는걸 보면 저 캡슐형 타원에서 계속 빠져나가는게 보인다. 그래서 결국 메모리 관리는 포기하고 하나씩 svg 파일로 만들어서 임포트하니까 많이 벗어나지 않는게 보인다

| 타원에서 많이 벗어난다(스프라이트 시트)               | 타원에서 덜 벗어난다(하나씩 임포트)                 |
| ------------------------------------ | ------------------------------------ |
| ![](https://i.imgur.com/QWDoJxo.png) | ![](https://i.imgur.com/1ULIQde.png) |

## 👯‍♂️ Ref & LINKS TO THIS PAGE

- [웹, what is sprite sheet](https://spritesheeteditor.com/spritesheet.html)


