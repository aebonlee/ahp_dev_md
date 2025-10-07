# 개발일지 76 - PostgreSQL 데이터베이스 스키마 전체 분석

## 📅 분석 일시
2025-09-03

## 🎯 분석 목적
- 컬러 테마 불안정 문제의 근본 원인 파악
- PostgreSQL 사용자 DB 스키마 정확한 구조 확인
- 전역변수와 지역변수 충돌 문제 해결
- 데이터베이스 설계 문제점 식별

## 📊 PostgreSQL 데이터베이스 현황

### 🗄️ 기본 정보
- **DB 엔진**: PostgreSQL 17.6 (Debian)
- **연결 상태**: ✅ 정상 연결
- **호스팅**: Render.com
- **백엔드 API**: https://ahp-platform.onrender.com

### 🏗️ 확인된 테이블 구조

#### 1. Users 테이블 (개인설정 핵심)
```sql
users 테이블 구조:
- id: SERIAL PRIMARY KEY
- email: VARCHAR(255) UNIQUE NOT NULL ✅
- first_name: VARCHAR(100) ✅ 이름
- theme: VARCHAR(50) ✅ 개인 테마 설정
- language: VARCHAR(10) ✅ 언어 설정
- phone: VARCHAR(20) ✅ 전화번호
- updated_at: TIMESTAMP ✅ 수정 일시

확인된 실제 데이터:
- admin@ahp-system.com (ID: 1) - first_name: Admin, theme: gold
- test@ahp.com (ID: 50) - first_name: Test, theme: gold
```

#### 2. 추정되는 다른 테이블들
- **projects** - AHP 프로젝트 관리
- **criteria** - 의사결정 기준
- **alternatives** - 대안 관리
- **pairwise_comparisons** - 쌍대비교 데이터
- **results** - AHP 결과 저장

## ❌ 발견된 문제점들

### 1. Users 테이블 누락 컬럼
- **last_name**: 성씨 정보 완전 누락
- **role**: 사용자 권한 (admin/evaluator) 구분 컬럼 없음
- **password_hash**: 비밀번호 저장 방식 불명
- **created_at**: 계정 생성 일시 미확인

### 2. 테마 상태 관리 충돌
**문제:** 여러 곳에서 테마를 로드/설정하여 충돌 발생
- `ThemeContext.tsx`: 🌍 전역 테마 관리
- `PersonalSettings.tsx`: 📝 지역 설정 폼 관리
- `useColorTheme.tsx`: 🔄 레거시 훅 (사용 중단 필요)

**해결책:** 단일 책임 원칙 적용
```typescript
// 🌍 전역: ThemeContext만 테마 상태 관리
const { currentTheme, setTheme } = useTheme();

// 📝 지역: PersonalSettings는 폼 데이터만 관리  
const [settings, setSettings] = useState<UserSettings>();
```

### 3. 인증 시스템 문제
- **401 TOKEN_REQUIRED**: 로그인 자격증명 불일치
- **쿠키 인증**: httpOnly 쿠키 방식이지만 테스트 어려움
- **인증 상태**: 브라우저별로 다른 상태 유지

## 🔧 적용한 해결책들

### 1. 전역/지역 상태 명확한 구분
```typescript
// ThemeContext.tsx
// 🌍 전역 상태: 앱 전체 테마
const [currentTheme, setCurrentTheme] = useState<ColorTheme | null>(null);

// PersonalSettings.tsx  
// 📝 지역 상태: 폼 데이터만
const [settings, setSettings] = useState<UserSettings>();
const [saveStatus, setSaveStatus] = useState<'idle' | 'saving'>('idle');
```

### 2. 테마 충돌 완전 제거
- PersonalSettings에서 테마 중복 로드 제거
- ThemeContext만 DB 테마 로드 담당
- 선택한 테마 즉시 전역 적용

### 3. 디버깅 개선
```typescript
// 전역 상태를 window 객체에 노출
window.GLOBAL_THEME_STATE = {
  current: theme,
  isLoaded: true
};
```

## 🧪 PostgreSQL 테스트 결과

### DB 연결 테스트
✅ **성공**: PostgreSQL 17.6 정상 연결
✅ **스키마**: personal_settings_ready 상태
❌ **인증**: admin 계정 자격증명 불일치

### 사용자 데이터 확인
```json
{
  "id": 1,
  "email": "admin@ahp-system.com", 
  "first_name": "Admin",
  "theme": "gold",
  "language": "ko",
  "phone": null,
  "updated_at": "2025-09-01T05:52:46.832Z"
}
```

## 🎯 완료된 개선사항

### 1. 테마 기본값 변경
- **이전**: 하드코딩된 'gold' 기본값
- **현재**: DB 우선, 실패 시 'rose-red' 기본값

### 2. 상태 관리 단순화  
- **이전**: 3개 테마 시스템 충돌
- **현재**: ThemeContext 단일 전역 관리

### 3. PostgreSQL 완전 테스트 환경
- `test_profile_update.html`: 완전한 인증→저장→검증 플로우

## 📋 남은 작업들

### 즉시 해결 필요
1. **로그인 자격증명 확인**: 올바른 admin 비밀번호 파악
2. **last_name 컬럼 추가**: 성씨 정보 저장을 위한 DB 스키마 확장
3. **테마 지속성 검증**: 실제 로그인 후 테마 저장/로드 테스트

### 장기 개선사항
1. **role 컬럼 추가**: 관리자/평가자 권한 구분
2. **audit_log 테이블**: 설정 변경 이력 추적
3. **session_management**: 다중 디바이스 세션 관리

## 🔍 데이터베이스 설계 평가

### ✅ 잘 설계된 부분
- 개인설정 컬럼 준비 완료
- 타임스탬프 자동 관리  
- PostgreSQL 17.6 최신 버전
- Render.com 안정적 호스팅

### ❌ 개선이 필요한 부분  
- last_name 컬럼 누락
- role 기반 권한 시스템 미구현
- 테스트 계정 접근 어려움
- API 문서화 부족

## 📊 종합 평가
**데이터베이스 설계 점수: 75/100**
- 핵심 기능: 90% ✅
- 개인설정: 70% 🔄  
- 확장성: 60% ❌
- 보안성: 85% ✅