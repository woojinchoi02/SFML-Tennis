# SFML-Tennis 기능 추가
---------
제가 구현한 기능은 4번 주고받으면 바의 크기가 줄어드는 것과 2번 주고받을 때마다 공의 속도가 증가하는 것입니다.

1. 바가 줄어드는 장면
   
![크기 작아지는거](https://github.com/woojinchoi02/SFML-Tennis/assets/162526228/571b05d5-8879-495b-bff6-83aeaf3e9c9c)

바의 크기를 조절하는 함수입니다. 줄어들 수 있는 최대치를 만들어 일정치 이상 작아지면 더 이상 작아질 수 없게 만들었습니다.

![크기 조절함수](https://github.com/woojinchoi02/SFML-Tennis/assets/162526228/45a92c74-8410-4868-857b-99a0954be0df)

2. 공이 빨라지는 장면

![크기 조절함수](https://github.com/woojinchoi02/SFML-Tennis/assets/162526228/9e4016b6-4251-431f-b7bc-b6102af7a74e)

두 번 정도 튕길 때마다 공이 점점 빨라지도록 설계하였습니다.

![스피드 조절 함수](https://github.com/woojinchoi02/SFML-Tennis/assets/162526228/2ea5a9f6-3dd1-4f6d-8e0a-e44b08109434)

전체 구현 장면입니다.

![전체](https://github.com/woojinchoi02/SFML-Tennis/assets/162526228/1d3e587f-4081-47e2-83ab-2138f087617f)

득점이 생기면 다시 원래의 초기값을 가지도록 설계하였습니다.






