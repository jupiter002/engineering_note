Object::all() - 지정한 테이블내의 모든 데이터 조회
Object::find($key) - 테이블의 기본키에 대해서 $key값으로 조회
Object::where('column_name', $param) - 컬럼의 값과 $param일치하는 조건 생성
Object::first - DB의 기본 순서에 따라 첫번째 단일행을 조회
Object::create - 새 레코드 생성
Object::with(Relationship model) - Model객체에 선언해놓은 관계모델명을 설정하면 
Instance->update - 기존 레코드 업데이트
Instance->delete - 레코드 삭제
Instance->save - 기존 레코드 업데이트 또는 레코드 생성
save메소드는 create, update와 다르게 연관 배열 형식을 통한 대량 할당이 아닌 개별 속성 설정 방식을 사용함.
대량 할당 : create(['userId' => 'isid3', 'password' => 'ispassword3'])
개별 할당 : $user->userId = 'isid3'
			$user->password = 'ispassword3'
			$user->save();
Instance->paginate - 조회한 레코들에 대해서 페이지네이션을 할 수 있음.
- **$perPage**: 페이지당 표시할 레코드 수 (기본값: 15).
- **$columns**: 조회할 컬럼 (기본값: ['*'], 즉 모든 컬럼).
- **$pageName**: URL 쿼리 파라미터 이름 (기본값: 'page').
- **$page**: 현재 페이지 번호 (기본적으로 request('page')에서 자동 가져옴).
	```php
	$users = User::paginate(
		$perPage, $columns, $pageName, $page
	);
	```
반환 메타데이터
- **$paginatedData->total()**: 전체 레코드 수.
- **$paginatedData->perPage()**: 페이지당 레코드 수.
- **$paginatedData->currentPage()**: 현재 페이지 번호.
- **$paginatedData->lastPage()**: 마지막 페이지 번호.
- **$paginatedData->hasMorePages()**: 추가 페이지 여부.
- **$paginatedData->links()**: 페이지네이션 링크 HTML.
- **$paginatedData->items()**: 현재 페이지의 레코드 배열.