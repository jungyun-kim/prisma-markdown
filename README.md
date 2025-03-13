# Atlas

## 백엔드 기술 스택
- Language: TypeScript
- Database: MySQL (with Prisma ORM)
  - raw sql 을 빌드해서 타입추론을 가능하게 하는 기능 https://www.prisma.io/docs/orm/prisma-client/using-raw-sql/typedsql?utm_campaign=typedsql
- API: HTTP REST
- Authentication: JWT + Cookie
- HTTP Client: got or @toss/ky
- Monitoring: AWS SNS-SQS
- Error Handling: Custom Error Classes
- 로시안 aws cdn 을 사용
- zod, joi ?
- redis 진짜 필요한가? (레슨 동시진입시, 락을 거는 용도)

## 프론트 기술 스택
 - Next.js
 - React
 - ?


## API 설계 규칙
- GET, POST 두가지 메소드만으로 구성(필요 시 PATCH 추가)
- GET은 서버의 자원을 얻어갈때 사용
- POST는 서버에서 자원을 생성할때 사용
- 멱등성 미들웨어로 같은 요청이라고 생각되면 `Idempotency-Key` 를 같은걸 넣어줌
- 

### HTTP 메소드
- **GET**: 리소스 조회
  - 단일 리소스: `/api/v1/users/:id`
  - 목록 조회: `/api/v1/users`
  - 필터링: `/api/v1/users?role=student`
  - 관계 조회: `/api/v1/users/:id/curriculums`

- **POST**: 리소스 생성 및 작업 실행
  - 리소스 생성: `/api/v1/users`
  - 작업 실행: `/api/v1/lessons/:id/start`
  - 복잡한 검색: `/api/v1/problems/search`

### 응답 코드
- **2xx**: 성공
  - 200: OK (GET 요청 성공)
  - 201: Created (POST로 리소스 생성 성공)
  - 204: No Content (작업 성공했지만 응답 데이터 없음)

- **4xx**: 클라이언트 에러
  - 400: Bad Request (잘못된 요청)
  - 401: Unauthorized (인증 필요)
  - 403: Forbidden (권한 없음)
  - 404: Not Found (리소스 없음)
  - 409: Conflict (리소스 충돌)

- **5xx**: 서버 에러
  - 500: Internal Server Error
  - 503: Service Unavailable

### API 응답 형식
```typescript
// 성공 응답
interface SuccessResponse<T> {
  success: true;
  data: T;
  message?: string;
}

// 에러 응답
interface ErrorResponse {
  success: false;
  error: {
    code: string;
    message: string;
    details?: any;
  };
}
```

### URL 명명 규칙
- 복수형 명사 사용: `/users`, `/problems`
- 소문자와 하이픈 사용: `/lesson-sessions`
- 행위는 명사로 표현: `/auth/login` (O), `/auth/log-in` (X)
- 버전 명시: `/api/v1/...`

### 쿼리 파라미터 규칙
- 필터링: `filter_필드명` (예: `filter_status=active`)
- 정렬: `sort=필드명:asc|desc` (예: `sort=created_at:desc`)
- 페이징: `page=1&limit=20`
- 검색: `search=검색어`

### 요청/응답 데이터 규칙
- 요청/응답 모두 `application/json` 사용
- 날짜는 ISO 8601 형식 사용
- 중첩된 객체는 최대 3depth까지만 허용
- `snake_case` 대신 `camelCase` 사용

## 개발 가이드라인
- 모든 API 엔드포인트는 인증이 필요 (특별한 경우 제외)
- 에러 응답은 항상 일관된 형식 사용
- 대용량 데이터는 페이지네이션 필수
- API 문서화는 Swagger/OpenAPI 사용

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

## 인증
### JWT 토큰 구조
```typescript
interface JWTPayload {
  userId: string;
  role: string;
  channelId: string;
  exp: number;
  iat: number;
}
```

### 토큰 관리
- Access Token: 2시간 유효
- Refresh Token: 없음
- Authorization 헤더: `Bearer {token}`

## 모니터링
### AWS SNS-SQS 구조
- SNS Topic: `atlas-monitoring`
- SQS Queues:
  - `atlas-error-queue`: 에러 로그
  - `atlas-audit-queue`: 감사 로그
  - `atlas-metric-queue`: 메트릭 데이터

### 모니터링 데이터 형식
```typescript
interface MonitoringEvent {
  timestamp: string;
  type: 'ERROR' | 'AUDIT' | 'METRIC';
  source: string;
  data: {
    userId?: string;
    action?: string;
    errorCode?: string;
    message: string;
    metadata?: Record<string, any>;
  };
}
```

## 에러 처리
### 에러 클래스 구조
```typescript
// 기본 에러 클래스
class AtlasError extends Error {
  constructor(
    public code: string,
    public status: number,
    message: string,
    public details?: any
  ) {
    super(message);
  }
}

// 구체적인 에러 클래스들
class ValidationError extends AtlasError {
  constructor(message: string, details?: any) {
    super('VALIDATION_ERROR', 400, message, details);
  }
}

class AuthenticationError extends AtlasError {
  constructor(message: string) {
    super('AUTHENTICATION_ERROR', 401, message);
  }
}

class AuthorizationError extends AtlasError {
  constructor(message: string) {
    super('AUTHORIZATION_ERROR', 403, message);
  }
}
```

### 에러 코드 체계
- `AUTH_`: 인증/인가 관련 에러
- `VAL_`: 유효성 검증 에러
- `BIZ_`: 비즈니스 로직 에러
- `SYS_`: 시스템 에러
- `EXT_`: 외부 서비스 에러

## HTTP 클라이언트
### Got 사용
```typescript
import got, { Got, Response } from 'got';

const client: Got = got.extend({
  prefixUrl: process.env.API_BASE_URL,
  responseType: 'json',
  retry: {
    limit: 2,
    methods: ['GET', 'POST']
  },
  hooks: {
    beforeRequest: [
      (options) => {
        options.headers = {
          ...options.headers,
          'Content-Type': 'application/json'
        };
      }
    ],
    afterResponse: [
      (response: Response) => {
        // 응답 후처리 로직
        return response;
      }
    ],
    beforeError: [
      error => {
        const { response } = error;
        if (response && response.body) {
          return new AtlasError(
            response.body.code,
            response.statusCode,
            response.body.message,
            response.body.details
          );
        }
        return error;
      }
    ]
  }
});

// 타입 안전성이 보장된 요청 메서드
const httpClient = {
  async get<T>(url: string, options?: any): Promise<T> {
    const response = await client.get(url, options);
    return response.body as T;
  },

  async post<T>(url: string, data: any, options?: any): Promise<T> {
    const response = await client.post(url, {
      ...options,
      json: data
    });
    return response.body as T;
  }
};
```

## 쿠키 기반 상태 관리
### 쿠키 설정
```typescript
interface CookieOptions {
  httpOnly: boolean;
  secure: boolean;
  sameSite: 'strict' | 'lax' | 'none';
  maxAge?: number;
  domain?: string;
  path?: string;
}

const defaultCookieOptions: CookieOptions = {
  httpOnly: true,
  secure: process.env.NODE_ENV === 'production',
  sameSite: 'lax'
};
```

### 주요 쿠키 항목
```typescript
interface StateCookies {
  // 인증 관련
  'atlas.auth': string;       // JWT 토큰
  'atlas.refresh': string;    // 리프레시 토큰
  
  // 사용자 설정
  'atlas.preferences': {
    theme: string;
    language: string;
  };
  
  // 학습 상태
  'atlas.learning': {
    lastLessonId: string;
    currentUnit: string;
    progress: number;
  };
  
  // 세션 관리
  'atlas.session': {
    deviceId: string;
    lastActive: string;
  };
}
```

### 쿠키 관리 유틸리티
```typescript
const cookieUtils = {
  set(res: Response, name: string, value: any, options?: Partial<CookieOptions>) {
    res.cookie(name, value, {
      ...defaultCookieOptions,
      ...options
    });
  },

  clear(res: Response, name: string) {
    res.clearCookie(name, defaultCookieOptions);
  },

  // 특정 도메인의 모든 쿠키 제거
  clearAll(res: Response) {
    const cookieNames = [
      'atlas.auth',
      'atlas.refresh',
      'atlas.preferences',
      'atlas.learning',
      'atlas.session'
    ];
    cookieNames.forEach(name => this.clear(res, name));
  }
};
```

### 쿠키 보안
- `httpOnly`: XSS 공격 방지
- `secure`: HTTPS 통신 강제
- `sameSite`: CSRF 공격 방지
- 민감한 정보는 암호화하여 저장
- 정기적인 쿠키 만료 및 갱신