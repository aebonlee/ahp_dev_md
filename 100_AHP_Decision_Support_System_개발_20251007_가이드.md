# AHP Decision Support System - 개발 가이드

## 📖 목차

1. [프로젝트 구조](#프로젝트-구조)
2. [개발 환경 설정](#개발-환경-설정)
3. [컴포넌트 아키텍처](#컴포넌트-아키텍처)
4. [상태 관리](#상태-관리)
5. [API 설계](#api-설계)
6. [코딩 컨벤션](#코딩-컨벤션)
7. [테스팅 가이드](#테스팅-가이드)
8. [배포 가이드](#배포-가이드)

## 🏗️ 프로젝트 구조

### 전체 구조
```
ahp-decision-system/
├── frontend/                 # React + TypeScript 프론트엔드
│   ├── src/
│   │   ├── components/       # UI 컴포넌트
│   │   │   ├── admin/        # 관리자 기능 (A-0~A-4)
│   │   │   ├── evaluator/    # 평가자 기능 (R-1~R-2)
│   │   │   ├── common/       # 공통 컴포넌트
│   │   │   └── layout/       # 레이아웃 컴포넌트
│   │   ├── config/           # 설정 파일
│   │   ├── data/             # 데모 데이터
│   │   ├── hooks/            # 커스텀 훅
│   │   ├── services/         # API 서비스
│   │   ├── store/            # 상태 관리
│   │   └── utils/            # 유틸리티 함수
│   ├── public/               # 정적 파일
│   ├── package.json          # 의존성 관리
│   └── tailwind.config.js    # Tailwind 설정
├── backend/                  # Node.js + Express 백엔드
│   ├── src/
│   │   ├── routes/           # API 라우트
│   │   ├── services/         # 비즈니스 로직
│   │   ├── database/         # 데이터베이스 설정
│   │   ├── middleware/       # 미들웨어
│   │   ├── types/            # TypeScript 타입
│   │   └── utils/            # 유틸리티
│   ├── migrations/           # 데이터베이스 마이그레이션
│   └── package.json          # 의존성 관리
├── docs/                     # 프로젝트 문서
└── README.md                 # 프로젝트 개요
```

### 컴포넌트 구조 상세

#### 관리자 편 컴포넌트 (`frontend/src/components/admin/`)
```
admin/
├── LandingPage.tsx           # A-0: 서비스 신청/시작하기
├── ProjectCreation.tsx       # A-1: 프로젝트 생성
├── ModelBuilding.tsx         # A-2: 모델구축 메인
├── CriteriaManagement.tsx    # A-2-1: 기준 추가
├── AlternativeManagement.tsx # A-2-2: 대안 추가
├── EvaluatorAssignment.tsx   # A-2-3: 평가자 배정
├── ModelFinalization.tsx     # A-2-4: 모델 구축 확정
├── EvaluationResults.tsx     # A-3: 평가결과 확인 메인
├── GroupWeightAnalysis.tsx   # A-3-1: 그룹별 가중치 도출
├── SensitivityAnalysis.tsx   # A-3-2: 민감도 분석
└── ProjectCompletion.tsx     # A-4: 프로젝트 완료
```

#### 평가자 편 컴포넌트 (`frontend/src/components/evaluator/`)
```
evaluator/
├── ProjectSelection.tsx      # R-1: 프로젝트 선택
├── PairwiseEvaluation.tsx    # R-2: 쌍대비교 평가
├── MatrixGrid.tsx           # 1-9 스케일 매트릭스
├── JudgmentHelper.tsx       # 판단 도우미 사이드패널
└── DirectInputEvaluation.tsx # R-2: 직접입력 평가
```

## 🛠️ 개발 환경 설정

### 필수 요구사항
- **Node.js**: 20.x 이상
- **npm**: 9.x 이상
- **Git**: 최신 버전
- **PostgreSQL**: 14.x 이상 (선택사항)

### 개발 환경 구축

1. **저장소 클론**
```bash
git clone https://github.com/aebonlee/AHP_forPaper.git
cd AHP_forPaper
```

2. **프론트엔드 설정**
```bash
cd frontend
npm install
npm start  # http://localhost:3000
```

3. **백엔드 설정**
```bash
cd backend
npm install
npm run dev  # http://localhost:3001
```

4. **환경 변수 설정**
```bash
# backend/.env
DATABASE_URL=postgresql://username:password@localhost:5432/ahp_db
JWT_SECRET=your-super-secret-jwt-key-here
NODE_ENV=development
CORS_ORIGIN=http://localhost:3000
PORT=3001
```

## 🏛️ 컴포넌트 아키텍처

### 컴포넌트 설계 원칙

1. **단일 책임 원칙**: 각 컴포넌트는 하나의 명확한 기능만 담당
2. **재사용성**: 공통 컴포넌트는 `common/` 디렉토리에 배치
3. **TypeScript 활용**: 모든 컴포넌트에 엄격한 타입 정의
4. **Props 인터페이스**: 명확한 Props 타입 정의

### 컴포넌트 예시

```typescript
// 컴포넌트 인터페이스 정의
interface ProjectSelectionProps {
  evaluatorId: string;
  onProjectSelect: (projectId: string, projectTitle: string, evaluationMethod: 'pairwise' | 'direct') => void;
}

// 컴포넌트 구현
const ProjectSelection: React.FC<ProjectSelectionProps> = ({ 
  evaluatorId, 
  onProjectSelect 
}) => {
  // 상태 관리
  const [assignedProjects, setAssignedProjects] = useState<AssignedProject[]>([]);
  
  // 이벤트 핸들러
  const handleProjectEnter = (project: AssignedProject) => {
    if (project.status === 'completed') return;
    onProjectSelect(project.id, project.title, project.evaluationMethod);
  };

  // 렌더링
  return (
    <div className="max-w-4xl mx-auto">
      {/* 컴포넌트 내용 */}
    </div>
  );
};
```

### 공통 컴포넌트

#### Button 컴포넌트
```typescript
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'danger';
  size?: 'sm' | 'md' | 'lg';
  loading?: boolean;
  disabled?: boolean;
  onClick?: () => void;
  children: React.ReactNode;
}
```

#### Card 컴포넌트
```typescript
interface CardProps {
  title?: string;
  children: React.ReactNode;
  className?: string;
}
```

## 🔄 상태 관리

### React Hooks 기반 상태 관리

1. **useState**: 컴포넌트 로컬 상태
2. **useEffect**: 사이드 이펙트 처리
3. **useContext**: 전역 상태 (필요시)

### 상태 관리 패턴

```typescript
// 기본 상태 관리
const [user, setUser] = useState<User | null>(null);
const [loading, setLoading] = useState(false);
const [error, setError] = useState<string>('');

// 복합 상태 관리
const [formData, setFormData] = useState({
  title: '',
  description: '',
  objective: ''
});

// 상태 업데이트
const handleInputChange = (field: string, value: string) => {
  setFormData(prev => ({
    ...prev,
    [field]: value
  }));
};
```

### 전역 상태 관리 (App.tsx)

```typescript
function App() {
  // 전역 상태
  const [user, setUser] = useState<User | null>(null);
  const [activeTab, setActiveTab] = useState('landing');
  const [selectedProjectId, setSelectedProjectId] = useState<string | null>(null);

  // 역할별 초기 화면 설정
  useEffect(() => {
    if (user) {
      if (user.role === 'admin') {
        setActiveTab('landing');
      } else if (user.role === 'evaluator') {
        setActiveTab('evaluator-dashboard');
      }
    }
  }, [user]);
}
```

## 🌐 API 설계

### REST API 엔드포인트

```
/api/auth/
├── POST /login          # 로그인
├── POST /logout         # 로그아웃
├── GET  /profile        # 프로필 조회
└── POST /refresh        # 토큰 갱신

/api/projects/
├── GET    /             # 프로젝트 목록
├── POST   /             # 프로젝트 생성
├── GET    /:id          # 프로젝트 상세
├── PUT    /:id          # 프로젝트 수정
└── DELETE /:id          # 프로젝트 삭제

/api/evaluations/
├── GET    /:projectId   # 평가 데이터 조회
├── POST   /pairwise     # 쌍대비교 저장
├── POST   /direct       # 직접입력 저장
└── GET    /results/:id  # 결과 조회
```

### API 응답 형식

```typescript
// 성공 응답
interface ApiResponse<T> {
  success: true;
  data: T;
  message?: string;
}

// 에러 응답
interface ApiError {
  success: false;
  error: string;
  code?: number;
}

// 사용 예시
const response: ApiResponse<Project[]> = {
  success: true,
  data: [
    {
      id: '1',
      title: 'IT 시스템 선택',
      description: '...',
      status: 'active'
    }
  ]
};
```

## 📝 코딩 컨벤션

### TypeScript 타입 정의

```typescript
// 인터페이스 네이밍: PascalCase
interface ProjectData {
  id: string;
  title: string;
  description: string;
  status: 'draft' | 'active' | 'completed';
}

// 컴포넌트 Props: 컴포넌트명 + Props
interface ProjectCardProps {
  project: ProjectData;
  onSelect: (id: string) => void;
}

// 상수: UPPER_SNAKE_CASE
const API_BASE_URL = 'https://api.example.com';

// 함수: camelCase
const calculateConsistencyRatio = (matrix: number[][]): number => {
  // 구현
};
```

### CSS 클래스 네이밍 (Tailwind CSS)

```typescript
// 기본 스타일링
<div className="max-w-4xl mx-auto p-6">
  <h1 className="text-3xl font-bold text-gray-900 mb-4">
    제목
  </h1>
  <p className="text-gray-600 mb-6">
    설명 텍스트
  </p>
</div>

// 조건부 스타일링
<button className={`px-4 py-2 rounded transition-colors ${
  isActive 
    ? 'bg-blue-500 text-white' 
    : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
}`}>
  버튼
</button>
```

### 컴포넌트 구조

```typescript
// 1. Import 구문
import React, { useState, useEffect } from 'react';
import Button from '../common/Button';

// 2. 타입 정의
interface ComponentProps {
  // props 정의
}

// 3. 컴포넌트 정의
const Component: React.FC<ComponentProps> = ({ prop1, prop2 }) => {
  // 4. 상태 정의
  const [state, setState] = useState();

  // 5. 이펙트
  useEffect(() => {
    // 사이드 이펙트
  }, []);

  // 6. 이벤트 핸들러
  const handleClick = () => {
    // 이벤트 처리
  };

  // 7. 렌더링
  return (
    <div>
      {/* JSX */}
    </div>
  );
};

// 8. Export
export default Component;
```

## 🧪 테스팅 가이드

### 테스트 전략

1. **단위 테스트**: 개별 함수/컴포넌트
2. **통합 테스트**: 컴포넌트 간 상호작용
3. **E2E 테스트**: 사용자 시나리오

### 테스트 작성 예시

```typescript
// 유틸리티 함수 테스트
describe('calculateConsistencyRatio', () => {
  it('should return 0 for perfectly consistent matrix', () => {
    const matrix = [[1, 2, 3], [0.5, 1, 1.5], [0.33, 0.67, 1]];
    const cr = calculateConsistencyRatio(matrix);
    expect(cr).toBeLessThan(0.1);
  });
});

// 컴포넌트 테스트
describe('ProjectCard', () => {
  it('should render project information correctly', () => {
    const project = {
      id: '1',
      title: 'Test Project',
      description: 'Test Description',
      status: 'active'
    };
    
    render(<ProjectCard project={project} onSelect={jest.fn()} />);
    
    expect(screen.getByText('Test Project')).toBeInTheDocument();
    expect(screen.getByText('Test Description')).toBeInTheDocument();
  });
});
```

## 🚀 배포 가이드

### 로컬 빌드

```bash
# 프론트엔드 빌드
cd frontend
npm run build

# 백엔드 빌드
cd backend
npm run build
```

### Docker 배포

```bash
# 전체 시스템 배포
docker-compose up -d

# 개별 서비스 배포
docker build -t ahp-frontend ./frontend
docker build -t ahp-backend ./backend
```

### Render.com 배포

1. GitHub 저장소 연결
2. `render.yaml` 설정 자동 적용
3. 환경 변수 자동 설정
4. 자동 배포 실행

## 🔧 개발 도구 및 유틸리티

### 권장 VS Code 확장

- **ES7+ React/Redux/React-Native snippets**
- **TypeScript Importer**
- **Tailwind CSS IntelliSense**
- **Prettier - Code formatter**
- **ESLint**

### 개발 스크립트

```json
{
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "lint": "eslint src --ext .ts,.tsx",
    "format": "prettier --write src/**/*.{ts,tsx}"
  }
}
```

## 📚 추가 학습 자료

- [React 공식 문서](https://reactjs.org/docs)
- [TypeScript 핸드북](https://www.typescriptlang.org/docs)
- [Tailwind CSS 문서](https://tailwindcss.com/docs)
- [AHP 방법론](https://en.wikipedia.org/wiki/Analytic_hierarchy_process)

---

이 가이드는 AHP Decision Support System의 개발을 위한 기본 지침입니다. 
추가 질문이나 개선사항이 있으면 언제든지 문의해 주세요.