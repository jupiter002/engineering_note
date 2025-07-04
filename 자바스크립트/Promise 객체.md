자바스크립트에서 비동기 처리는 현재 실행중인 작업과 별도로 다른 작업을 수행하는 걸 말한다.
비동기 특성상 실행 코드의 동작이 완료될 때까지 기다리지 않고 다음 코드를 먼저 수행하기 때문에,
비동기 작업의 결과에 따라 다른 작업을 수행하려면 콜백 함수를 사용했다.

하지만 콜백함수를 사용하면 가독성이 떨어지는 문제가 있고, 비동기 작업이 많아지면 콜백 함수를 중첩으로 사용해야 하는 문제가 있어 Promise객체로 반환값을 처리하게 되었다.

	Promise객체 - 비동기 작업의 최종 완료 또는 실패를 나타내는 Array 혹은 Object같은 독자적인 객체

Promise객체 생성
	new 키워드와 Promise생성자 함수를 사용한다. 생성자의 매개변수로 resolve, reject 두 개가 들어간다. 
	resolve는 작업이 성공했을 때, reject는 실패했을 때 결과를 알려주는 객체이다.
	
```js
	const myPromise = new Promise((resolve, reject) => {
		// 비동기 작업 수행
	    const data = fetch('서버로부터 요청할 URL');
	    
	    if(data)
	    	resolve(data); // 만일 요청이 성공하여 데이터가 있다면
	    else
	    	reject("Error"); // 만일 요청이 실패하여 데이터가 없다면
	})
```


프로미스 객체에 대한 처리
	프로미스 객체에  .then(), catch() 메소드 체이닝을 사용해 후속 처리를 할 수 있지만
	재사용성 때문에 함수로 만들어서 사용한다.
	
```js
	// 프로미스 객체를 반환하는 함수 생성
	function myPromise() {
	  return new Promise((resolve, reject) => {
	    if (/* 성공 조건 */) {
	      resolve(/* 결과 값 */);
	    } else {
	      reject(/* 에러 값 */);
	    }
	  });
	}
	
	// 프로미스 객체를 반환하는 함수 사용
	myPromise()
	    .then((result) => {
	      // 성공 시 실행할 콜백 함수
	    })
	    .catch((error) => {
	      // 실패 시 실행할 콜백 함수
	    });
```

fetch()가 메서드 내에서 프로미스 객체를 생성하여 서버로부터 데이터를 전달 성공시 resolve()해서 .then()
체이닝으로 처리한다. 실패시는 reject()해서 .cathc()체이닝한다.

프로미스 3가지 상태
	new Promise()생성자로 객체를 생성 후, 진행되는 비동기 작업은 언젠가 성공 또는 실패할 것이다. 이 과정에서 생기는 Pending(대기), Fulfilled(이행), Rejected(거부) 3가지 상태가 발생한다.

	Pending(대기) 상태
	아직 비동기 처리 로직이 완료되지 않은 상태이다.
```js
	const promise = new Promise((resolve, reject) => {
		setTimeout(() => {
	    	resolve("처리 완료")
	    }, 5000)
	});
	console.log(promise); // Pending (대기) 상태
```
	위 코드는 자바스크립트로 5초 뒤에 비동기처리를 완료시키는 코드이다.
	5초전이라면 Promise객체는 Pending상태일 것이다.

	Fulfiled(이행) 상태
	위 예시코드에서 5초가 지난후에 Promise객체를 확인하면 상태가 fulfilled상태인 것을 확인할
	 수 있다. 이것은 5초후에 resolve()가 수행되면서 성공을 알리는 개체(객체?)를 호출해 비동기
	 처리 로직이 성공적으로 완료됐다는 것을 의미한다.

	Rejected(거부) 상태
	어떠한 이유로 비동기 처리가 실패해 reject()가 실행되어 프로미스 객체의 상태가 rejected가
	된다. 이후 Promise의 처리는 catch()체이닝으로 처리한다.

프로미스 핸들러
	프로미스 객체의 결과를 핸들러를 통해 결과를 전달받고 다음 후속 작업을 진행할 수 있다.
	프로미스 핸들러는 프로미스의 상태에 따라 실행되는 콜백 함수라고 보면 된다.

	.then()
		- 프로미스가 fulfilled됐을때 실행할 콜백 함수를 등록, 새로운 프로미스 반환
	.catch()
		- 프로미스가 rejected됐을때 실행할 콜백 함수를 등록, 새루운 프로미스 반환
	.finally()
		- 프로미스의 실패, 성공 여부 상관없이 실행할 콜백 함수 등록, 새로운 프로미스 반환

프로미스 체이닝
	프로미스 핸들러를 연달아 달아놓는 것을 의미, 여러 개의 비동기 작업을 순차적으로 수행할 수 있다.

```js
	function doSomething() {
	  return new Promise((resolve, reject) => {
	      resolve(100)
	  });
	}
	
	doSomething()
	    .then((value1) => {
	        const data1 = value1 + 50;
	        return data1
	    })
	    .then((value2) => {
	        const data2 = value2 + 50;
	        return data2
	    })
	    .then((value3) => {
	        const data3 = value3 + 50;
	        return data3
	    })
	    .then((value4) => {
	        console.log(value4); // 250 출력
	    })
```
	위처럼 체이닝이 가능한 이유는 then()에서 값을 리턴하면, 반환값이 자동으로 프로미스 객체로 
	감싸져 반환되기 때문이다. 후에 다음 then()에서 반환된 프로미스 객체를 받아 처리하는 것이
	다.

프로미스 정적(static) 메서드
사용자 편의상 Promise객체를 생성하지 않고 바로 사용할 수 있다.

	Promise.resolve()
	Promise의 성공 결과를 반환한다.
	
```js
	// 프로미스 객체와 전혀 연관없는 함수
	function getRandomNumber() {
	  const num = Math.floor(Math.random() * 10); // 0 ~ 9 사이의 정수
	  return num;
	}
	
	// Promise.resolve() 를 사용하여 프로미스 객체를 반환하는 함수
	function getPromiseNumber() {
	  const num = getRandomNumber(); // 일반 값
	  return Promise.resolve(num); // 프로미스 객체
	}
```
	Promise.reject()
	Promise의 실패 결과를 반환한다.
```js
	// 주어진 사유로 거부되는 프로미스 생성
	const p = Promise.reject(new Error('error'));
	
	// 거부 사유를 출력
	p.catch(error => console.error(error)); // Error: error
```

	Promise.all()
	배열, Map, Set에 포함된 여러개의 프로미스 요소들을 한꺼번에 비동기 처리할 때 유용하게 사용
	할 수 있다. 모든 비동기 처리가 fulfilled될 때까지 기다려서, 모든 작업이 완료되면 then()
	핸들러가 작동되는 형태이다.

```js
	// 1. 서버 요청 API 프로미스 객체 생성 (fetch)
	const api_1 = fetch("https://jsonplaceholder.typicode.com/users");
	const api_2 = fetch("https://jsonplaceholder.typicode.com/users");
	const api_3 = fetch("https://jsonplaceholder.typicode.com/users");
	
	// 2. 프로미스 객체들을 묶어 배열로 구성
	const promises = [api_1, api_2, api_3];
	
	// 3. Promise.all() 메서드 인자로 프로미스 배열을 넣어, 모든 프로미스가 이행될 때까지 기
	다리고, 결과값을 출력
	Promise.all(promises)
	    .then((results) => {
	      // results는 이행된 프로미스들의 값들을 담은 배열.
	      // results의 순서는 promises의 순서와 일치.
	      console.log(results); // [users1, users2, users3]
	    })
	    .catch((error) => {
	      // 어느 하나라도 프로미스가 거부되면 오류를 출력
	      console.error(error);
	    });
```

	Promise.allSettled()
	Promise.all()의 개선 버전으로, 주어진 모든 프로미스가 처리되면 모든 프로미스 각각의 상태
	와 값을 모아놓은 배열을 반환한다.

```js
	// 1초 후에 1을 반환하는 프로미스
	const p1 = new Promise(resolve => setTimeout(() => resolve(1), 1000));
	
	// 2초 후에 에러를 발생시키는 프로미스
	const p2 = new Promise((resolve, reject) => setTimeout(() => reject(new 
	Error('error')), 2000));
	
	// 3초 후에 3을 반환하는 프로미스
	const p3 = new Promise(resolve => setTimeout(() => resolve(3), 3000));
	
	// 세 개의 프로미스의 상태와 값 또는 사유를 출력
	Promise.allSettled([p1, p2, p3])
		.then(result => console.log(result));
```



	Promise.any()
	Promise.all()반대되는 조건을 가진다. Promise.any()는 모든 프로미스 중 하나라도 완료되면
	 바로 반환한다.
	만일 요청한 모든 프로미스가 rejected되면 AggregateError객체를 사유로 하는 거부 프로미스
	를 반환한다.

	Promise.race()
	Promise.any()와 같이 가장 먼저 처리된 프로미스 결과값을 반환하지만 차이점이 있다.
	Promise.any()는 가장 먼저 fulfilled된 프로미스만 반환하는 반면, Promise.race()는
	fulfilled/rejected여부 상관없이 처리가 끝난 프로미스 결과값을 반환한다.

프로미스에 대한 처리가 많아지면 메소드 체이닝이 길어져서 콜백함수에서 겪었던 콜백 지옥을 볼 수 있을 것이다. 이걸 해결한 async/await가 있다.