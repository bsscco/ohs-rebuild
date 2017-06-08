### MVC 잘 적용하기
- View와 Model 간에 서로 모르게 한다.
	- View와 Model의 재활용 가능성이 높아진다.
- Controller의 역할
	- Controller는 View와 Model을 가진다(has a).
	- View에게 갱신을 요청하고, 갱신을 위한 데이터를 제공하는 것은 Controller의 역할이다.
	- View에서 발생한 이벤트를 핸들링하는 것은 Controller의 역할이다. 핸들링 안에는 Model에 데이터를 조작을 요청하는 작업도 포함한다.

