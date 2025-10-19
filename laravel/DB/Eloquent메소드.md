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