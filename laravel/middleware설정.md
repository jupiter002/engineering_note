(인증/인가 sanctum의 경우 아래와 같이 사용함. 타 미들웨어 경우 달라질 수 있음.)
app/config/auth.php
	- default영역 : 어떤 guard를 기본으로 설정할지 결정
	- guard영역 : 
		- 인증방식(session, sanctum, passport)을 결정할 수 있음
		- 제공자 연결 : 특정 유저 모델과 DB를 연결하는 Provider를 사용해서 유저 데이터를 활용할 수 있음
		- 유저별로 다른 인증방식을 사용할 수 있음(다중 인증방식 제공)
	- provider영역 : 유저별 Provider명 지정 및 ORM, Model선택
app/routes/~.php
	- route::httpMethod앞 혹은 뒤쪽에 middleware('example:guardName')설정