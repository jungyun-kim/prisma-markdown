# Atlas Database Conventions

## 데이터베이스 설정
- Character Set: `utf8mb4`
- Collation: `utf8mb4_general_ci`

## 테이블 명명 규칙
- 스네이크 케이스(`snake_case`) 사용
- 복수형으로 작성 (예: `users`, `problem_sets`)
- 모두 소문자 사용

## 컬럼 명명 규칙
1. **Primary Key**
   - `id`: 일반적인 자동 증가 Primary Key
   - 특수한 경우 의미있는 이름 사용 (예: `variable_name`, `memo_uuid`)

2. **Foreign Key**
   - 참조하는 테이블의 단수형 + `_id` (예: `user_id`, `curriculum_id`)
   - 관계를 명확히 표현 (예: `parent_id`, `session_id`)

3. **일반 컬럼**
   - 스네이크 케이스 사용
   - 의미를 명확히 표현 (예: `hashed_password`, `access_token`)

4. **Boolean 컬럼**
   - `is_` 접두사 사용 (예: `is_default`, `is_active`, `is_step`)

5. **날짜/시간 컬럼**
   - 접미사로 시간 표현
   - `created_at`: 생성 시간
   - `updated_at`: 수정 시간
   - `modified_at`: 수정 시간
   - `removed_at`: 삭제 시간 (소프트 삭제)
   - `expires_at`: 만료 시간
   - `last_login`: 마지막 로그인 시간

6. **JSON/텍스트 데이터**
   - TEXT 타입 사용
   - 필요한 경우 MEDIUMTEXT 사용

## 인덱스 명명 규칙
- 기본 인덱스: 자동 생성
- 커스텀 인덱스: 
  - 단일 컬럼: `idx_{테이블명}_{컬럼명}`
  - 복합 컬럼: `idx_{테이블명}_{컬럼1}_{컬럼2}`

## 제약 조건
- Nullable 허용: 필요한 경우에만
- Unique 제약: 비즈니스 규칙에 따라 적용
- Foreign Key: 관계 무결성 보장

## 매핑 규칙 (Prisma)
- `@map`: 컬럼명 매핑
- `@@map`: 테이블명 매핑
- 모델명은 PascalCase (예: `UserAuth`, `LessonSession`)
- 필드명은 camelCase (예: `userId`, `createdAt`)
