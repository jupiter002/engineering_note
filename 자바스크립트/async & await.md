Promise의 체이닝이 길어지다보면 구현 의도를 쉽게 파악할 수 없는 문제를 해결하고자 추가됐다.
async/await는 Promise를 대체하기 위해서 나온 기술이 아니다. 내부적으로는 Promise객체를 사용해서 비동기 처리를 하고 가독성을 향상시키기 위해서 작성자에게 보이는 문법만 다르게 해줄 뿐이다.

async/await 사용법
함수를 만들 때 function키워드 앞에 async를 쓰고, 비동기로 처리할 부분에 await를 쓰면 된다.

```js
// 프로미스 객체 반환 함수
function delay(ms) {
  return new Promise(resolve => {
    setTimeout(() => {
      console.log(`${ms} 밀리초가 지났습니다.`);
      resolve()
    }, ms);
  });
}

// 기존 Promise.then() 형식
function main() {
  delay(1000)
      .then(() => {
        return delay(2000);
      })
      .then(() => {
        return Promise.resolve('끝');
      })
      .then(result => {
        console.log(result);
      });
}

// async/await 방식
async function main() {
  await delay(1000);
  await delay(2000);
  const result = await Promise.resolve('끝');
  console.log(result);
}
```
asnyc키워드
	function앞에 async키워드를 붙여주는 것 만으로, 함수내에서 await키워드를 사용할 수 있게 된다. 
	await키워드를 사용하려면 async function의 정의가 필요하다.

async function의 반환값
	async funtion의 반환값은 항상 Promise객체이다. 단순 정수, 문자열을 반환해도 Promise객
	체 형태로 감싸서 반환이 된다.

	Promise객체에서 사용할 수 있는 정적 메서드들로 반환되는 프로미스의 상태를 변경할 수 있다.
	Promise.rejected()를 사용해도 reject가 반환되지만 throw키워드로 예외를 발생시켜도 
	reject상태가 반환된다.

	함수내에서 return을 하지 않아도 fulfilled상태의 undefined값을 가진 Promise객체가 반환된다.
	async function의 반환값은 Promise객체이기 때문에 then핸들러를 사용할 수 있다. 하지만
	많이 사용하면 프로미스 지옥에 빠질 수 있어서 기본적으로 await키워드를 사용한다.

await 키워드
	then핸들러보다 세련되고 프로미스 지옥에 빠지지 않게 해주는 문법이다.
	
```js
	// then 핸들러 방식
	fetch(url)
	    .then(res => res.json()) // 응답을 JSON으로 파싱
	    .then(data => {
	      // data 처리
	      console.log(data);
	    })

	// await 방식
	async function func() {
	    const res = await fetch(url); // 요청을 기다림
	    const data = await res.json(); // 응답을 JSON으로 파싱
	    // data 처리
	    console.log(data);
	}
	func();
```


	await키워드를 사용하면 메소드 체이닝을 사용할 필요가 없고 비동기 처리할 부분에 await키워
	드를 쓰면 되기 때문에 보다 명시적이게 코드를 작성할 수 있다.

	await는 Promise처리가 끝날때까지 기다린다.

	Promise비동기 처리가 끝날때까지 코드 실행을 일시 중지하고, 대기한다라는 뜻으로 보면 된다.
```js
	async function getData() {
	  const response = await fetch('https://jsonplaceholder.typicode.com/users/1');
	  const data = await response.json();
	  console.log(data):
	}
```
	위 코드에서 fetch()비동기 함수를 호출하고, 반환된 Promise를 await키워드로 처리한다. 이
	로 인해 함수 내 코드 실행이 일시 중지되고, fetch()로 요청한 결과가 전달되어서야 다음 코드
	를 실행하게 된다. 이 과정에서 await는 Promise를 처리하고 결과를 반환하는 데, 비동기적인
	작업을 동기적으로 처리할 수 있게 된다.

	"동기적"은 "순차적"과 일맥상통하는 단어다.


	async/await 에러 처리

	기존의 Promise으 메소드 체이닝 방식으로는 catch()핸들러를 중간마다 넣어줘야 에러 처리가
	가능했다.
	하지만 async/await는 흔히 사용했던 try/catch문으로 에러 처리가 가능하다.
	
```js
	async function func() {
	
	    try {
	        const res = await fetch(url); // 요청을 기다림
	        const data = await res.json(); // 응답을 JSON으로 파싱
	        // data 처리
	        console.log(data);
	    } catch (err) {
	        // 에러 처리
	        console.error(err);
	    }
	
	}
	func();
	```
	async/await를 사용하면 비동기코드를 동기코드처럼 보이게 작성할 수 있다.

async/await의 함정
	async/await는 Promise객체를 세련되게 처리하기 위해서 추가된 문법이지만 남용하면 성능 문제가 발생할 수 있다. await키워드가 Promise비동기 처리가 해결될 때까지 코드 실행을 일시 중지하는데, 병렬 처리로 2초 만에 해결할 수 있는 작업을 2 \*n초만큼 소모되게 할 수도 있기 때문이다


```js
//틀린 예시
function getApple(){
  return new Promise( (resolve, reject) => {
	setTimeout(() => resolve("apple"), 1000);
  })
}

function getBanana(){
  return new Promise( (resolve, reject) => {
	setTimeout(() => resolve("banana"), 1000);
  })
}
// getApple(), getBanana()를 서버에 요청하는 소스라고 생각한다.
async function getFruites(){
  let a = await getApple(); 
  let b = await getBanana(); 
  console.log(`${a} and ${b}`);
}

getFruites();

//좋은 예시
async function getFruites(){

  let getApplePromise = getApple(); // async함수를 미리 논블록킹으로 실행한다. 
  let getBananaPromise = getBanana(); // async함수를 미리 논블록킹으로 실행한다. 
  
  // 이렇게 하면 각각 백단에서 독립적으로 거의 동시에 실행되게 된다.
  console.log(getApplePromise)
  console.log(getBananaPromise)
  
  let a = await getApplePromise; // 위에서 받은 프로미스객체 결과 변수를 await을 통해 꺼낸다.
  let b = await getBananaPromise; // 위에서 받은 프로미스객체 결과 변수를 await을 통해 꺼낸다.
  
  console.log(`${a} and ${b}`); // 본래라면 1초+1초 를 기다려야 하는데, 위에서 1초기다리는 함수를 바로 연속으로 비동기로 불려왔기 때문에, 대충 1.01초만 기다리면 처리된다.
})
```
	getApple(), getBanana() 두 비동기 로직이 순서를 지켜야하는 로직이라면 틀린 예시의 방법
	이 옳다. 하지만 두 로직상 연관이 없기 때문에 무조건 순차적으로 실행시킬 이유가 없다. 이 경
	우에는 성능을 개선하기 위해서 좋은 예시의 방법으로 사용하면 된다.

	다른 방법으로는 Promise.all()를 사용하면 된다. 위처럼 구성할 경우 비동기 처리의 완료 시점
	을 가늠하기 어렵기 때문에 실무에선 Promise.all()로 처리한다.

	Promise.all()은 매개변수로 들어간 프로미스 함수들이 reslove가 되야 반환값을 받는다. 배열
	인자의 각 프로미스 함수들은 개별적으로 처리되어 시간을 단축할 수 있다.
	반환값은 프로미스 함수의 반환값들이 배열로 담겨져 있다.
```js
async function getFruites(){
  console.time();
  
  // 구조 분해로 각 프로미스 리턴값들을 변수에 담는다.
  let [ a, b ] = await Promise.all([getApple(), getBanana()]); 
  console.log(`${a} and ${b}`);
  
  console.timeEnd();
}

getFruites();
```

top-level await
	기존에는 await키워드를 사용하기 위해서는 무조건 async function을 정의하고 함수안에서 await
	키워드 사용이 가능했다. 하지만 함수로 감싸지 않아도 될 코드를 Promise객체를 처리하기 위해서
	억지로 감싸 IIFE로 사용해왔다.	```
```js
//기존 방식
(async function func1() {
	const res = await fetch(url); // 요청을 기다림
	const data = await res.json(); // 응답을 JSON으로 파싱
	console.log(data);
})();
```
이후에 추가된 문법에서는 async키워드로 함수를 만들지 않아도 await키워드를 사용할 수 있게 된다.
```js
// Top Level에선 async funciton 정의없이 곧바로 await 키워드 사용이 가능하다
const res = await fetch(url); // 요청을 기다림
const data = await res.json(); // 응답을 JSON으로 파싱
console.log(data);
```
