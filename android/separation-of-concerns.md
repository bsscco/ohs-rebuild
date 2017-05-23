### MLVC(Model-LightView-Controller)
- 뭔가?
	- MVC에서 서로 종속되는 ```View```와 ```Model``` 사이의 관계를 끊어서 ```View```의 재활용성을 높힘.
- 구현방법
	- ```View```의 UI를 바꾸는 작업은 ```View```에 함수로 정의해서 함수로만 접근하도록 한다.
	- 이렇게 하면 ```View```는 ```Model```에 대해 몰라도 된다.
