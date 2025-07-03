window 객체

	웹 브라우저의 창을 나타내는 객체
	window객체의 메소드는 전역 함수, 프로퍼티는 전역 변수가 된다.
	DOM요소들도 모두 window객체의 프로퍼티가 된다.
	
	브라우저 창 크기 조절
		innerHeight
		innerWidth
	브라우저 새 창 열기
		open()
	브라우저 창 닫기
		close()
	대화상자
		alert() - 사용자에게 간단한 메시지를 보여줌. 확인 버튼을 눌러야 다음 작업 수행 가능
		confirm() - 메시지를 보여주고, 확인이나 취소 버튼을 누르면 결과를 boolean으로 반환. 확인 : true, 취소 : false
		prompt() - 메시지를 보여주고, 사용자가 입력한 문자열을 반환.

location 객체
	현재 브라우저에 표시된 html문서의 주소를 얻고나, 새 문서를 불러올 때 사용
	
	location.href - 현재 문서의 URL주소를 문자열로 반환, url을 지정해서 페이지 이동도 가능
	location.hostname - 현재 문서의 인터넷 호스트 이름 반환
	location.pathname - 현재 문서의 URL경로명
	location.assign - 브라우저 창에 지정된 URL주소에 존재하는 문서를 불러옴
	location.replace - 이전 페이지 정보를 히스토리에 제거. 페이지 이동 후 기존 페이지로 이동 불가

hisotry 객체
	브라우저 히스토리 정보를 문서와 문서 상태 목록으로 저장하는 객체
	
	history.length - 브라우저 히스토리 목록 개수 반환
	history.back(), forward() - 앞, 뒤로 가기
	history.go() - 전달 받는 인수의 값 만큼 히스토리 목록을 건너뛸 수 있음

screen 객체
	사용자의 화면에 대한 정보를 저장하는 객체

	screen.width, height - 사용자의 화면 크기를 픽셀 단위로 반환
		- window.outerWidth, outerHeight는 브라우저의 창 크기를 반환
	screen.availWidth, availHeight - 실제 사용할 수 있는 화면의 크기를 픽셀 단위로 반환

navigator 객체
	브라우저에 대한 정보를 저장하는 객체
	
	navigator.appVersion - 사용중인 브라우저 버전 정보를 문자열로 반환
	navigator.userAgent - appVerison + 상세정보
	navigator.platform - 사용중인 운영체제 정보
	navigator.language - 사용중인 브라우저 언어 설정
	navigator.javaEnabled - 사용중인 브라우저가 자바 애플릿을 실행할 수 있는지 여부
	navigator.cookieEnabled - 사용중인 브라우저가 쿠키를 사용할 수 있는지 여부
