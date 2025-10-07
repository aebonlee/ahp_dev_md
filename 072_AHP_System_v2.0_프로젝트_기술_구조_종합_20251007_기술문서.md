# AHP System v2.0 프로젝트 기술 구조 종합 문서

**문서 번호**: docs_08-02  
**작성일**: 2025년 9월 7일  
**프로젝트**: AHP System v2.0 기술 아키텍처  
**작성자**: Claude Code + aebonlee  

---

## 🏗️ 전체 시스템 아키텍처

### 3계층 구조
```
┌─────────────────────────────────────┐
│         프론트엔드 (React)           │
│    https://aebonlee.github.io/      │
│           ahp_app/                  │
└─────────────────┬───────────────────┘
                  │ HTTP/HTTPS
                  │ API 호출
┌─────────────────▼───────────────────┐
│         백엔드 (Node.js)            │
│   https://ahp-platform.onrender.com│
└─────────────────┬───────────────────┘
                  │ SQL 쿼리
                  │ 세션/쿠키
┌─────────────────▼───────────────────┐
│      데이터베이스 (PostgreSQL)       │
│   dpg-d2q8l5qdbo4c73bt3780-a       │
└─────────────────────────────────────┘
```

---

## 🖥️ 프론트엔드 상세 구조

### React 앱 구조
```
src/
├── components/          # 재사용 가능한 컴포넌트
│   ├── common/         # 공통 UI 컴포넌트
│   ├── auth/           # 인증 관련 컴포넌트  
│   ├── admin/          # 관리자 전용 컴포넌트
│   └── evaluation/     # 평가 관련 컴포넌트
├── pages/              # 페이지 레벨 컴포넌트
│   ├── HomePage.tsx    # 메인 랜딩 페이지
│   ├── LoginPage.tsx   # 로그인 페이지
│   ├── DashboardPage.tsx # 대시보드
│   └── ProjectsPage.tsx  # 프로젝트 관리
├── store/              # 상태 관리 (Zustand)
│   └── authStore.ts    # 인증 상태
├── services/           # 외부 서비스 연동
│   └── api.ts         # HTTP 클라이언트
├── config/            # 설정 파일
│   └── api.ts        # API 엔드포인트 정의
├── types/            # TypeScript 타입 정의
│   └── index.ts     # 공통 인터페이스
├── utils/           # 유틸리티 함수
├── hooks/          # 커스텀 React Hooks
└── styles/        # CSS 및 스타일
```

### 상태 관리 패턴 (Zustand)
```typescript
// authStore.ts 예시
interface AuthState {
  user: User | null;
  isAuthenticated: boolean;
  isLoading: boolean;
  error: string | null;
  
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
  checkSession: () => Promise<void>;
}
```

### 라우팅 구조
```typescript
Routes:
  / → HomePage (공개)
  /login → LoginPage (공개)
  /dashboard → DashboardPage (인증 필요)
  /projects → ProjectsPage (인증 필요)
```

---

## 🔧 백엔드 시스템 구조

### Express.js 기반 RESTful API
```
backend/
├── src/
│   ├── routes/         # API 라우트
│   │   ├── auth.ts    # 인증 관련 API
│   │   ├── projects.ts # 프로젝트 API
│   │   ├── users.ts   # 사용자 API
│   │   └── evaluation.ts # 평가 API
│   ├── middleware/    # 미들웨어
│   │   ├── auth.ts   # 인증 미들웨어
│   │   └── validation.ts # 데이터 검증
│   ├── models/       # 데이터 모델
│   ├── services/     # 비즈니스 로직
│   ├── utils/       # 유틸리티
│   └── database/    # DB 연결 및 마이그레이션
├── migrations/      # DB 스키마 변경
└── scripts/        # 유틸리티 스크립트
```

### 주요 API 엔드포인트
```typescript
// 인증 관련
POST /api/auth/login
POST /api/auth/logout  
GET /api/auth/session

// 프로젝트 관리
GET /api/projects
POST /api/projects
GET /api/projects/:id
PUT /api/projects/:id
DELETE /api/projects/:id

// 평가 관련
POST /api/evaluate
GET /api/results/:projectId

// 사용자 관리
GET /api/users/profile
PUT /api/users/update
```

---

## 🗄️ 데이터베이스 스키마

### PostgreSQL 테이블 구조
```sql
-- 사용자 테이블
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR UNIQUE NOT NULL,
  name VARCHAR NOT NULL,
  role VARCHAR CHECK (role IN ('admin', 'user', 'evaluator')),
  created_at TIMESTAMP DEFAULT NOW()
);

-- 프로젝트 테이블
CREATE TABLE projects (
  id UUID PRIMARY KEY,
  title VARCHAR NOT NULL,
  description TEXT,
  status VARCHAR CHECK (status IN ('draft', 'active', 'completed')),
  created_by UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW()
);

-- 기준 테이블 (계층 구조)
CREATE TABLE criteria (
  id UUID PRIMARY KEY,
  project_id UUID REFERENCES projects(id),
  name VARCHAR NOT NULL,
  description TEXT,
  parent_id UUID REFERENCES criteria(id),
  weight DECIMAL
);

-- 대안 테이블
CREATE TABLE alternatives (
  id UUID PRIMARY KEY,
  project_id UUID REFERENCES projects(id),
  name VARCHAR NOT NULL,
  description TEXT
);

-- 쌍대비교 테이블
CREATE TABLE comparisons (
  id UUID PRIMARY KEY,
  project_id UUID REFERENCES projects(id),
  evaluator_id UUID REFERENCES users(id),
  criterion_id UUID REFERENCES criteria(id),
  item_a VARCHAR NOT NULL,
  item_b VARCHAR NOT NULL,
  value DECIMAL NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);
```

---

## 🔐 인증 및 보안

### 세션 기반 인증
```typescript
// 쿠키 기반 세션 관리
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    maxAge: 30 * 60 * 1000 // 30분
  }
}));
```

### CORS 설정
```typescript
app.use(cors({
  origin: [
    'https://aebonlee.github.io',
    'http://localhost:3000' // 개발용
  ],
  credentials: true
}));
```

### 미들웨어 보안
- **입력 검증**: Joi 스키마 검증
- **SQL 인젝션 방지**: Parameterized queries
- **XSS 방지**: 입력 sanitization
- **CSRF 보호**: CSRF 토큰 검증

---

## 📦 빌드 및 배포

### 프론트엔드 배포 (GitHub Pages)
```bash
# 빌드 프로세스
npm run build
├── React 앱 최적화 빌드
├── TypeScript 컴파일
├── CSS 최적화
├── 번들 압축 (gzip)
└── 정적 파일 생성

# 배포 프로세스
npm run deploy
├── gh-pages 브랜치 생성
├── build 폴더 업로드
├── GitHub Pages 자동 배포
└── DNS 업데이트
```

### 백엔드 배포 (Render.com)
```bash
# 자동 배포 프로세스
git push → Render webhook
├── Docker 컨테이너 빌드
├── Node.js 앱 실행
├── PostgreSQL 연결
└── 도메인 연결
```

---

## 📊 성능 최적화

### 프론트엔드 최적화
```typescript
// 코드 스플리팅
const LazyComponent = React.lazy(() => import('./Component'));

// 메모이제이션
const MemoizedComponent = React.memo(Component);

// 상태 최적화 (Zustand)
const useStore = create((set) => ({
  // 최소한의 리렌더링
}));
```

### 빌드 최적화 결과
- **JavaScript**: 90.25 kB (gzipped)
- **CSS**: 2.25 kB (gzipped)  
- **총 크기**: ~94 kB (매우 경량)

### 백엔드 최적화
- **연결 풀링**: PostgreSQL 연결 재사용
- **캐싱**: Redis 세션 저장소
- **압축**: gzip 응답 압축
- **CDN**: 정적 파일 CDN 배포

---

## 🔗 외부 서비스 연동

### GitHub 연동
- **저장소**: https://github.com/aebonlee/ahp_app
- **자동 배포**: GitHub Actions
- **이슈 트래킹**: GitHub Issues

### 클라우드 서비스
- **프론트엔드 호스팅**: GitHub Pages
- **백엔드 호스팅**: Render.com
- **데이터베이스**: Render PostgreSQL
- **DNS**: GitHub Pages 도메인

---

## 🛠️ 개발 도구 및 워크플로우

### 개발 환경
```json
{
  "IDE": "VS Code",
  "Node.js": ">=18.0.0",
  "npm": ">=8.0.0",
  "Git": "Latest",
  "Browser": "Chrome DevTools"
}
```

### 코드 품질 도구
- **TypeScript**: 타입 안정성
- **ESLint**: 코드 스타일 검사
- **Prettier**: 코드 포매팅
- **Husky**: Git hooks

### 테스팅 전략
```typescript
// 단위 테스트
describe('API Service', () => {
  it('should handle authentication', async () => {
    // 테스트 코드
  });
});

// 통합 테스트
describe('User Flow', () => {
  it('should login and navigate to dashboard', () => {
    // E2E 테스트
  });
});
```

---

## 📱 반응형 디자인 시스템

### 브레이크포인트
```css
/* 모바일 */
@media (max-width: 768px) { 
  /* 세로 스택 레이아웃 */ 
}

/* 태블릿 */
@media (min-width: 769px) and (max-width: 1199px) {
  /* 2열 그리드 레이아웃 */
}

/* 데스크탑 */
@media (min-width: 1200px) {
  /* 3열 그리드 레이아웃 */
}
```

### 디자인 토큰
```css
:root {
  --primary-color: #007bff;
  --secondary-color: #6c757d;
  --success-color: #28a745;
  --danger-color: #dc3545;
  --warning-color: #ffc107;
  
  --font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI';
  --border-radius: 5px;
  --box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}
```

---

## 🔄 데이터 플로우

### 인증 플로우
```
1. 사용자 로그인 시도
2. 프론트엔드 → 백엔드 POST /api/auth/login
3. 백엔드 → 데이터베이스 사용자 확인
4. 세션 생성 및 쿠키 설정
5. 프론트엔드 상태 업데이트
6. 대시보드로 리다이렉트
```

### 프로젝트 생성 플로우
```
1. 사용자 프로젝트 생성 폼 작성
2. 프론트엔드 → 백엔드 POST /api/projects
3. 백엔드 데이터 검증
4. 데이터베이스 프로젝트 저장
5. 실시간 UI 업데이트
```

---

## 🚀 확장성 고려사항

### 수평 확장
- **로드 밸런서**: 다중 서버 인스턴스
- **데이터베이스 클러스터**: 읽기 복제본
- **CDN**: 정적 자원 분산

### 기능 확장
- **마이크로서비스**: 도메인별 서비스 분리
- **이벤트 스트리밍**: 실시간 업데이트
- **API Gateway**: 통합 API 관리

### 데이터 확장
- **파티셔닝**: 대용량 데이터 분할
- **캐싱 레이어**: Redis/Memcached
- **검색 엔진**: Elasticsearch

---

## 📋 모니터링 및 로깅

### 프론트엔드 모니터링
- **에러 트래킹**: Sentry
- **성능 모니터링**: Web Vitals
- **사용자 분석**: Google Analytics

### 백엔드 모니터링
- **APM**: New Relic/Datadog
- **로그 집계**: Winston + CloudWatch
- **헬스 체크**: /health 엔드포인트

---

## 🔧 개발 환경 설정

### 로컬 개발 환경
```bash
# 프론트엔드 설정
git clone https://github.com/aebonlee/ahp_app.git
cd ahp_app
npm install
npm start

# 백엔드 설정 (기존 서버)
# https://ahp-platform.onrender.com 사용
```

### 환경 변수
```bash
# .env (프론트엔드)
REACT_APP_API_BASE_URL=https://ahp-platform.onrender.com
REACT_APP_ENVIRONMENT=production
PUBLIC_URL=/ahp_app/

# .env (백엔드)
DATABASE_URL=postgresql://...
SESSION_SECRET=...
CORS_ORIGIN=https://aebonlee.github.io
```

---

## 📚 기술 문서 참조

### 핵심 라이브러리 문서
- **React**: https://react.dev/
- **TypeScript**: https://www.typescriptlang.org/
- **Zustand**: https://zustand-demo.pmnd.rs/
- **React Router**: https://reactrouter.com/
- **Axios**: https://axios-http.com/

### 클라우드 서비스 문서
- **GitHub Pages**: https://pages.github.com/
- **Render**: https://render.com/docs
- **PostgreSQL**: https://www.postgresql.org/docs/

---

## 💡 결론

AHP System v2.0은 현대적인 웹 기술 스택을 활용하여 확장 가능하고 유지보수가 용이한 
시스템으로 설계되었습니다. 프론트엔드의 React + TypeScript 조합과 백엔드의 
Node.js + PostgreSQL 조합은 안정적이고 성능이 우수한 애플리케이션을 제공합니다.

**주요 기술적 성과:**
- ✅ 100% TypeScript 적용으로 타입 안정성 확보
- ✅ 모던 React 패턴과 상태 관리 최적화
- ✅ RESTful API 설계와 보안 강화
- ✅ 완전한 반응형 디자인과 성능 최적화
- ✅ CI/CD 파이프라인과 자동 배포 시스템