Conditional class
```php
@php
    $isActive = false;
    $hasError = true;
@endphp

	<span @class([ //@class를 사용하면 변수에 상태에 따라서 html태그의 클래스명을 on/off처럼 다룰 수 있다.
    'p-4',
    'font-bold' => $isActive,
    'text-gray-500' => ! $isActive,
    'bg-red' => $hasError,
])></span>

<span class="p-4 text-gray-500 bg-red"></span>
```
