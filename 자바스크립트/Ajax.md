ajax는 클라이언트 풀링 방식이다.

클라이언트 풀링 방식의 데이터 요청 방식
	- 사용자가 원하는 정보를 서버에게 요청해 데이터를 전달받는 방식
	
이에 반해 서버 푸시 방식은 사용자가 요청하지 않아도 특정 정보를 사용자에게 전달하는 방식이다.
![[Pasted image 20250703151936.png]]
![[Pasted image 20250703152045.png]]

ajax 동작 순서
	1. 사용자에 의해 요청 발생
	2. 발생한 요청 이벤트에 따라 이벤트 핸들러에 의해 자바스크립트 호출
	3. 자바스크립트에서 XMLHttpRequest객체를 사용하여 서버로 요청. 클라이언트는 서버 응답을 기다리지 않고 다른 작업 처리 가능.
	4. 서버는 전달받은 XMLHttpReuqest객체를 ajax요청을 수행
	5. 서버는 처리한 결과를 html, xml, json등의 형태로 클라이언트로 전달
	6. 서버에서 전달받은 데이터를 웹 페이지의 일부분만을 갱신하는 자바스크립트 호출
	7. 웹 페이지의 일부분이 새로 전달받은 데이터로 갱신되어 표현

fetch() 사용법
	첫번쨰 인자로 URL, 두번째 인자로 옵션 객체를 받고 Promise타입의 객체를 반환.
	반환된 객체는 API호출이 성공한 경우, 응답 객체를 resolve, 실패했을 경우 예외 객체를 reject한다.
	
	사용예시 - 
		fetch(url, options)
		.then((response) => console.log("response", response))
		.catch((error) => console.log("error", error));

	옵션 객체는 Http method, header, body등을 설정할 수 있다. 응답 객체로부터는 status, headers, body등을 읽어올 수 있다.


```js
GET호출
fetch("https://jsonplaceholder.typicode.com/posts/1").then((response) =>
  console.log(response)
);
// GET 방식은 요청 전문을 받지 않기 때문에 옵션 인자가 필요 없다.

fetch("https://jsonplaceholder.typicode.com/posts/1")
  .then((response) => response.json())
  .then((data) => console.log(data));

//대부분의 REST API는 json형태의 데이터로 응답하기 때문에 response객체는 json()를 제공한다.
//이 메서드를 호출하면 응답 객체로부터 json포맷의 body를 자바스크립트 객체로 변환하여 얻을 수 있다.

POST호출
//원격서버에서 관리하고 있는 데이터를 생성해야 한다면 header를 포함할 수 있는 POST방식 필요
fetch("https://jsonplaceholder.typicode.com/posts", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    title: "Test",
    body: "I am testing!",
    userId: 1,
  }),
}).then((response) => console.log(response));
//header옵션 - json포맷을 사용에 대해 명시
//method옵션 - POST지정
//body옵션 - 요청 전문을 json포맷으로 직렬화

PUT호출
fetch("https://jsonplaceholder.typicode.com/posts/1", {
  method: "PUT",
  headers: {
    "Content-Type": "application/json",
  },
  body: JSON.stringify({
    title: "Test",
    body: "I am testing!",
    userId: 1,
  }),
})
  .then((response) => response.json())
  .then((data) => console.log(data));
//데이터를 수정할 때 사용
//method - PUT지정
//다른 옵션은 POST랑 동일

DELETE호출
fetch("https://jsonplaceholder.typicode.com/posts/1", {
  method: "DELETE",
})
  .then((response) => response.json())
  .then((data) => console.log(data));
//데이터를 삭제할 때 사용
//보낼 데이터가 없기 때문에 headers와 body옵션이 필요없음
```

