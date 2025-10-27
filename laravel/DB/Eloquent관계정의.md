주요메서드
- **hasOne**: 일대일 관계에서 부모 모델이 자식 모델을 소유.
- **belongsTo**: 자식 모델이 부모 모델에 속함.
- **hasMany**: 일대다 관계에서 부모 모델이 여러 자식 모델을 소유.
- **belongsToMany**: 다대다 관계를 정의하며, 중간 테이블을 사용.
- **morphTo / morphMany**: 다형성 관계를 정의.
- **hasManyThrough**: 중간 모델을 통한 관계 정의.

예시:
	
```php
// app/Models/User.php namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class User extends Model { public function profile() 
	{
		return $this->hasOne(Profile::class); 
	} 
}
부모 모델( User )은 자식 모델( Profile )을 소유한다.
return $this->hasOne(Profile::class, 'custom_foreign_key');
두번째 인자로 외래키명을 지정할 수 있다. 
```

with: 보통의 데이터를 조회하는 상황에 따라 Lazy Loading방식에서는 쿼리수가 N + 1로 늘어날 수 있음.
with메소드를 사용해 Eager Loading방식을 사용한다면 모델에 관계된 테이블에 대해서 한꺼번에 데이터를 조회하기 때문에 쿼리수가 무한으로 늘어나는 문제에 대해서 비교적 자유로울 수 있음.