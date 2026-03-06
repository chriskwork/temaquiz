# Spanish-Korean Learning App - 개발 지시서

> Claude Code와 개발자가 참조할 수 있는 전체 개발 가이드

**프로젝트명:** Spanish-Korean Quiz App  
**목표:** 스페인어-한국어 학습 앱 (사용자 1,000명)  
**기술 스택:** React (PWA) + Laravel + MySQL

---

## 📋 목차

1. [프로젝트 구조](#프로젝트-구조)
2. [개발 환경 설정](#개발-환경-설정)
3. [데이터베이스 스키마](#데이터베이스-스키마)
4. [API 설계](#api-설계)
5. [React 컴포넌트 구조](#react-컴포넌트-구조)
6. [개발 플로우](#개발-플로우)
7. [배포 가이드](#배포-가이드)

---

## 프로젝트 구조

### 폴더 레이아웃

```
spanish-korean-app/
├── frontend/                 # React 프로젝트 (PWA)
│   ├── src/
│   │   ├── components/
│   │   │   ├── WordCard.jsx
│   │   │   ├── QuizScreen.jsx
│   │   │   ├── CategoryList.jsx
│   │   │   ├── SearchBar.jsx
│   │   │   ├── ResultScreen.jsx
│   │   │   ├── WrongAnswerNotes.jsx
│   │   │   ├── AudioPlayer.jsx       # TTS/mp3 재생 컴포넌트
│   │   │   └── Navigation.jsx
│   │   ├── pages/
│   │   │   ├── HomePage.jsx
│   │   │   ├── QuizPage.jsx
│   │   │   ├── NotesPage.jsx
│   │   │   └── ProfilePage.jsx
│   │   ├── services/
│   │   │   ├── api.js          # API 호출 함수 (Phase 2)
│   │   │   ├── indexedDB.js    # IndexedDB 관리
│   │   │   ├── tts.js          # Web Speech API 래퍼
│   │   │   └── dataExport.js   # 진행상황 내보내기/가져오기
│   │   ├── data/
│   │   │   └── words.json      # Phase 1: 정적 단어 데이터
│   │   ├── App.jsx
│   │   ├── App.css
│   │   └── index.jsx
│   ├── public/
│   │   ├── manifest.json       # PWA 매니페스트
│   │   └── sw.js               # Service Worker
│   ├── package.json
│   ├── .env
│   └── .gitignore
│
├── backend/                  # Laravel 프로젝트 (Phase 2)
│   ├── app/
│   │   ├── Models/
│   │   │   ├── Word.php
│   │   │   ├── Category.php
│   │   │   ├── User.php
│   │   │   ├── UserProgress.php
│   │   │   └── WrongAnswer.php
│   │   ├── Http/
│   │   │   └── Controllers/
│   │   │       ├── WordController.php
│   │   │       ├── QuizController.php
│   │   │       ├── ProgressController.php
│   │   │       └── AuthController.php
│   │   └── Filament/
│   │       └── Resources/
│   │           ├── WordResource.php
│   │           └── CategoryResource.php
│   ├── routes/
│   │   ├── api.php
│   │   └── web.php
│   ├── database/
│   │   ├── migrations/
│   │   │   ├── 2024_create_categories_table.php
│   │   │   ├── 2024_create_words_table.php
│   │   │   ├── 2024_create_user_progress_table.php
│   │   │   └── 2024_create_wrong_answers_table.php
│   │   └── seeders/
│   │       └── WordsTableSeeder.php
│   ├── .env
│   ├── composer.json
│   └── .gitignore
│
├── .gitignore                # 루트 .gitignore
└── README.md                 # 프로젝트 소개
```

---

## 개발 환경 설정

### 필수 설치 프로그램

#### Windows/Mac 공통

1. **Git** (코드 관리)
   ```bash
   # 설치 확인
   git --version
   ```

2. **Node.js** (React용)
   ```bash
   # https://nodejs.org 에서 LTS 버전 다운로드
   node --version
   npm --version
   ```

3. **Herd** (Laravel용 - Phase 2)
   ```bash
   # https://herd.laravel.com 에서 다운로드
   herd --version
   ```

### 프로젝트 초기화

#### Step 1: GitHub Repository 생성

```bash
# github.com에서 새 repository 생성
# 이름: spanish-korean-app
# Public 선택
```

#### Step 2: 로컬 폴더 설정

```bash
# 프로젝트 폴더 생성
mkdir spanish-korean-app
cd spanish-korean-app

# Git 초기화
git init

# .gitignore 파일 생성
cat > .gitignore << 'EOF'
# Frontend
frontend/node_modules/
frontend/.env
frontend/.env.local
frontend/dist/

# Backend
backend/vendor/
backend/.env
backend/.env.local
backend/storage/logs/
backend/bootstrap/cache/

# OS
.DS_Store
Thumbs.db
*.log

# IDE
.vscode/
.idea/
*.swp
EOF

# GitHub 연결
git remote add origin https://github.com/yourusername/spanish-korean-app.git

# 첫 커밋
git add .
git commit -m "Initial commit: Project setup"
git branch -M main
git push -u origin main
```

#### Step 3: React 프로젝트 생성

```bash
# frontend 폴더 생성
npm create vite@latest frontend -- --template react

cd frontend
npm install

# .env 파일 생성
cat > .env << 'EOF'
VITE_API_URL=http://localhost:8000/api
VITE_PHASE=1
EOF
```

#### Step 4: Laravel 프로젝트 생성 (Phase 2)

```bash
# backend 폴더 생성
cd ../backend

composer create-project laravel/laravel .

# .env 파일 설정
cp .env.example .env

# .env 내용 수정
cat > .env << 'EOF'
APP_NAME=SpanishKoreanApp
APP_URL=http://localhost:8000

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=spanish_korean_app
DB_USERNAME=root
DB_PASSWORD=

# Herd에서 자동 생성
EOF

# 앱 키 생성
php artisan key:generate

# 데이터베이스 마이그레이션
php artisan migrate

# Filament 설치 (관리자 패널)
composer require filament/filament
php artisan filament:install --panels=admin
```

### 로컬 개발 서버 실행

#### Phase 1 (React만)

```bash
cd frontend
npm run dev

# 출력:
# ➜  Local:   http://localhost:5173/
```

#### Phase 2 (React + Laravel)

```bash
# 터미널 1 (React)
cd frontend && npm run dev

# 터미널 2 (Laravel)
cd backend && php artisan serve

# 브라우저
# http://localhost:5173 (React)
# http://localhost:8000/api (Laravel)
```

---

## 데이터베이스 스키마

### 1. Categories 테이블

```sql
CREATE TABLE categories (
  id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(255) NOT NULL,              -- '식당에서', '공항에서'
  description TEXT,
  emoji VARCHAR(10),                       -- '🍽️', '✈️'
  icon_url VARCHAR(255),
  slug VARCHAR(255) UNIQUE,                -- 'restaurant', 'airport'
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

**예시 데이터:**
```json
[
  { "id": 1, "name": "인사", "emoji": "👋", "slug": "greeting" },
  { "id": 2, "name": "식당에서", "emoji": "🍽️", "slug": "restaurant" },
  { "id": 3, "name": "공항에서", "emoji": "✈️", "slug": "airport" }
]
```

### 2. Words 테이블

```sql
CREATE TABLE words (
  id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
  category_id BIGINT UNSIGNED NOT NULL,
  spanish VARCHAR(255) NOT NULL,
  korean VARCHAR(255) NOT NULL,
  example_spanish TEXT,                   -- '¿Cómo estás?'
  example_korean TEXT,                    -- '어떻게 지내세요?'
  audio_url VARCHAR(255),                 -- Phase 2: '/audio/words/1.mp3' (nullable)
  difficulty ENUM('easy', 'normal', 'hard') DEFAULT 'normal',
  is_active BOOLEAN DEFAULT TRUE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (category_id) REFERENCES categories(id) ON DELETE CASCADE,
  INDEX idx_category (category_id),
  INDEX idx_spanish (spanish)
);
```

> **참고:** `audio_url`은 Phase 1에서는 NULL 상태.
> Phase 1에서는 Web Speech API로 발음 재생, Phase 2에서 Google TTS mp3로 교체.

**예시 데이터:**
```json
{
  "id": 1,
  "category_id": 1,
  "spanish": "hola",
  "korean": "안녕하세요",
  "example_spanish": "¡Hola, cómo estás?",
  "example_korean": "안녕하세요, 어떻게 지내세요?",
  "audio_url": null,
  "difficulty": "easy"
}
```

### 3. Users 테이블 (Phase 2)

```sql
CREATE TABLE users (
  id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  email_verified_at TIMESTAMP NULL,
  password VARCHAR(255) NOT NULL,
  phone VARCHAR(20),
  is_active BOOLEAN DEFAULT TRUE,
  last_login_at TIMESTAMP NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  INDEX idx_email (email)
);
```

### 4. UserProgress 테이블

> Phase 1: IndexedDB에 저장 / Phase 2: MySQL로 동기화

```sql
CREATE TABLE user_progress (
  id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
  user_id BIGINT UNSIGNED NOT NULL,
  word_id BIGINT UNSIGNED NOT NULL,
  correct_count INT DEFAULT 0,
  wrong_count INT DEFAULT 0,
  last_reviewed_at TIMESTAMP NULL,
  next_review_date TIMESTAMP NULL,           -- Phase 2: Spaced Repetition 용
  difficulty ENUM('easy', 'normal', 'hard') DEFAULT 'normal',
  streak INT DEFAULT 0,                      -- Phase 2: 연속 정답 수
  is_learned BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (word_id) REFERENCES words(id) ON DELETE CASCADE,
  UNIQUE KEY unique_user_word (user_id, word_id),
  INDEX idx_user (user_id),
  INDEX idx_next_review (next_review_date)
);
```

> **Phase 1 IndexedDB 스키마:**
> `correct_count`, `wrong_count`, `last_reviewed_at`만 사용.
> `next_review_date`, `streak`, `is_learned`은 Phase 2에서 활성화.

### 5. WrongAnswers 테이블

> Phase 1: IndexedDB에 저장 / Phase 2: MySQL로 동기화

```sql
CREATE TABLE wrong_answers (
  id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
  user_id BIGINT UNSIGNED NOT NULL,
  word_id BIGINT UNSIGNED NOT NULL,
  quiz_type ENUM('spanish_to_korean', 'korean_to_spanish') NOT NULL,
  user_answer VARCHAR(255),
  correct_answer VARCHAR(255) NOT NULL,
  explanation TEXT,                        -- 오답 설명 (선택사항)
  is_resolved BOOLEAN DEFAULT FALSE,       -- 복습했는지 여부
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
  FOREIGN KEY (word_id) REFERENCES words(id) ON DELETE CASCADE,
  INDEX idx_user (user_id),
  INDEX idx_word (word_id),
  INDEX idx_created (created_at)
);
```

> **변경사항:** `category_name` 컬럼 제거 → `word_id` JOIN으로 카테고리 조회.
> **변경사항:** `quiz_type`에서 `image_match` 제거.

### 마이그레이션 파일 예시

**database/migrations/2024_01_01_create_categories_table.php**

```php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(): void {
        Schema::create('categories', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->text('description')->nullable();
            $table->string('emoji')->nullable();
            $table->string('icon_url')->nullable();
            $table->string('slug')->unique();
            $table->boolean('is_active')->default(true);
            $table->timestamps();
        });
    }

    public function down(): void {
        Schema::dropIfExists('categories');
    }
};
```

### Seeder 예시

**database/seeders/WordsTableSeeder.php**

```php
<?php

namespace Database\Seeders;

use Illuminate\Database\Seeder;
use App\Models\Word;
use App\Models\Category;

class WordsTableSeeder extends Seeder {
    public function run(): void {
        // CSV 파일에서 읽기
        $csv = fopen(storage_path('words.csv'), 'r');
        
        while (($row = fgetcsv($csv)) !== false) {
            $category = Category::where('slug', $row[2])->first();
            
            if ($category) {
                Word::create([
                    'spanish' => $row[0],
                    'korean' => $row[1],
                    'category_id' => $category->id,
                    'example_spanish' => $row[3] ?? null,
                    'example_korean' => $row[4] ?? null,
                    'audio_url' => null,  // Phase 2에서 TTS mp3로 채움
                    'difficulty' => 'normal'
                ]);
            }
        }
        
        fclose($csv);
    }
}
```

---

## API 설계 (Phase 2)

> Phase 1에서는 API를 사용하지 않음. 정적 JSON + IndexedDB로 동작.
> Phase 2에서 아래 API를 구현하여 서버 동기화.

### Base URL
```
http://localhost:8000/api
```

### 1. 단어 관련 API

#### GET /words
**설명:** 전체 단어 목록 조회 (카테고리별)

**Query Parameters:**
```
category_id: number (선택)
search: string (선택)
limit: number (기본값: 100)
offset: number (기본값: 0)
```

**요청:**
```bash
curl "http://localhost:8000/api/words?category_id=1&limit=50"
```

**응답:**
```json
{
  "success": true,
  "data": {
    "words": [
      {
        "id": 1,
        "spanish": "hola",
        "korean": "안녕하세요",
        "category_id": 1,
        "audio_url": null,
        "example_spanish": "¡Hola!",
        "example_korean": "안녕하세요!"
      }
    ],
    "total": 3700,
    "category": { "id": 1, "name": "인사", "emoji": "👋" }
  }
}
```

#### GET /words/{id}
**설명:** 특정 단어 조회

**응답:**
```json
{
  "success": true,
  "data": {
    "id": 1,
    "spanish": "hola",
    "korean": "안녕하세요",
    "category_id": 1,
    "audio_url": null,
    "example_spanish": "¡Hola!",
    "example_korean": "안녕하세요!",
    "difficulty": "easy"
  }
}
```

#### GET /categories
**설명:** 모든 카테고리 조회

**응답:**
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "인사",
      "emoji": "👋",
      "slug": "greeting",
      "word_count": 50
    },
    {
      "id": 2,
      "name": "식당에서",
      "emoji": "🍽️",
      "slug": "restaurant",
      "word_count": 45
    }
  ]
}
```

#### GET /words/search
**설명:** 단어 검색

**Query Parameters:**
```
q: string (필수)
limit: number (기본값: 20)
```

**요청:**
```bash
curl "http://localhost:8000/api/words/search?q=agua"
```

**응답:**
```json
{
  "success": true,
  "data": [
    {
      "id": 2,
      "spanish": "agua",
      "korean": "물",
      "category_id": 2,
      "match_type": "spanish"
    }
  ]
}
```

### 2. 진행상황 관련 API (Phase 2)

#### POST /progress
**설명:** 사용자 학습 진행상황 저장

**요청 본문:**
```json
{
  "word_id": 1,
  "quiz_type": "spanish_to_korean",
  "is_correct": true
}
```

**응답:**
```json
{
  "success": true,
  "data": {
    "user_id": 5,
    "word_id": 1,
    "correct_count": 4,
    "wrong_count": 1,
    "is_learned": false,
    "next_review_date": "2026-03-08T10:30:00Z"
  }
}
```

#### GET /progress
**설명:** 사용자 진행상황 조회 (인증 필요)

**응답:**
```json
{
  "success": true,
  "data": {
    "total_learned": 150,
    "total_learning": 200,
    "categories": [
      {
        "id": 1,
        "name": "인사",
        "learned_count": 30,
        "total_count": 50,
        "progress_percent": 60
      }
    ]
  }
}
```

#### GET /progress/:wordId
**설명:** 특정 단어 진행상황 조회

**응답:**
```json
{
  "success": true,
  "data": {
    "word_id": 1,
    "correct_count": 3,
    "wrong_count": 1,
    "last_reviewed_at": "2026-03-06T10:30:00Z",
    "next_review_date": "2026-03-08T10:30:00Z",
    "difficulty": "easy",
    "streak": 3
  }
}
```

### 3. 오답 노트 API (Phase 2, 회원 전용)

#### POST /wrong-answers
**설명:** 오답 기록 저장

**요청 본문:**
```json
{
  "word_id": 2,
  "quiz_type": "spanish_to_korean",
  "user_answer": "주스",
  "correct_answer": "물"
}
```

**응답:**
```json
{
  "success": true,
  "data": {
    "id": 101,
    "user_id": 5,
    "word_id": 2,
    "created_at": "2026-03-05T14:20:00Z"
  }
}
```

#### GET /wrong-answers
**설명:** 사용자 오답 목록 조회 (인증 필요)

> **변경:** category_name은 word → category JOIN으로 조회

**Query Parameters:**
```
limit: number (기본값: 20)
offset: number (기본값: 0)
category_id: number (선택)
```

**응답:**
```json
{
  "success": true,
  "data": {
    "total": 45,
    "wrong_answers": [
      {
        "id": 101,
        "word_id": 2,
        "spanish": "agua",
        "korean": "물",
        "user_answer": "주스",
        "correct_answer": "물",
        "quiz_type": "spanish_to_korean",
        "category": {
          "id": 2,
          "name": "식당에서",
          "emoji": "🍽️"
        },
        "created_at": "2026-03-05T14:20:00Z"
      }
    ]
  }
}
```

#### DELETE /wrong-answers/{id}
**설명:** 오답 항목 제거

**응답:**
```json
{
  "success": true,
  "message": "오답 항목이 삭제되었습니다"
}
```

### 4. 인증 API (Phase 2)

#### POST /register
**설명:** 회원가입

**요청 본문:**
```json
{
  "name": "김철수",
  "email": "kim@example.com",
  "password": "password123",
  "password_confirmation": "password123"
}
```

**응답:**
```json
{
  "success": true,
  "data": {
    "id": 5,
    "name": "김철수",
    "email": "kim@example.com",
    "token": "eyJ0eXAiOiJKV1QiLCJhbGc..."
  }
}
```

#### POST /login
**설명:** 로그인

**요청 본문:**
```json
{
  "email": "kim@example.com",
  "password": "password123"
}
```

**응답:**
```json
{
  "success": true,
  "data": {
    "id": 5,
    "name": "김철수",
    "email": "kim@example.com",
    "token": "eyJ0eXAiOiJKV1QiLCJhbGc..."
  }
}
```

#### POST /logout
**설명:** 로그아웃 (인증 필요)

**응답:**
```json
{
  "success": true,
  "message": "로그아웃되었습니다"
}
```

### 5. 데이터 마이그레이션 API (Phase 2)

#### POST /migrate-progress
**설명:** IndexedDB에서 서버로 진행상황 마이그레이션 (회원가입 후)

**요청 본문:**
```json
{
  "progress_data": [
    {
      "word_id": 1,
      "correct_count": 3,
      "wrong_count": 1,
      "last_reviewed_at": "2026-03-06T10:30:00Z"
    }
  ],
  "wrong_answers": [
    {
      "word_id": 2,
      "quiz_type": "spanish_to_korean",
      "user_answer": "주스",
      "correct_answer": "물"
    }
  ]
}
```

**응답:**
```json
{
  "success": true,
  "message": "데이터가 동기화되었습니다",
  "data": {
    "migrated_progress": 50,
    "migrated_wrong_answers": 10
  }
}
```

---

## React 컴포넌트 구조

### 1. 데이터 흐름

```
Phase 1 (서버 없음):
App.jsx
├── words.json (정적 단어 데이터)
├── IndexedDB (진행상황 + 오답 로컬 저장)
├── Web Speech API (TTS 발음 재생)
└── State Management (Context API 또는 Zustand)

Phase 2 (서버 추가):
App.jsx
├── API (서버 데이터)
├── IndexedDB (오프라인 캐시)
├── audio_url (mp3) || Web Speech API (fallback)
└── State Management

Components:
├── Navigation (라우팅)
├── HomePage (카테고리 선택)
├── QuizPage (퀴즈 진행)
├── ResultScreen (결과 표시)
├── NotesPage (오답 노트)
└── ProfilePage (진행상황, 내보내기/가져오기)
```

### 2. 주요 컴포넌트

#### App.jsx

```jsx
import React, { useEffect, useState } from 'react';
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import Navigation from './components/Navigation';
import HomePage from './pages/HomePage';
import QuizPage from './pages/QuizPage';
import NotesPage from './pages/NotesPage';
import ProfilePage from './pages/ProfilePage';
import { initializeDB } from './services/indexedDB';

function App() {
  const [isDBReady, setIsDBReady] = useState(false);

  useEffect(() => {
    // IndexedDB 초기화
    initializeDB().then(() => setIsDBReady(true));
  }, []);

  if (!isDBReady) return <div>로딩 중...</div>;

  return (
    <Router>
      <Navigation />
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/quiz/:categoryId" element={<QuizPage />} />
        <Route path="/notes" element={<NotesPage />} />
        <Route path="/profile" element={<ProfilePage />} />
      </Routes>
    </Router>
  );
}

export default App;
```

#### HomePage.jsx

```jsx
import React, { useEffect, useState } from 'react';
import CategoryList from '../components/CategoryList';
import SearchBar from '../components/SearchBar';
import wordsData from '../data/words.json';

function HomePage() {
  const [categories, setCategories] = useState([]);
  const [searchResults, setSearchResults] = useState(null);

  useEffect(() => {
    // Phase 1: 정적 JSON에서 카테고리 추출
    const cats = wordsData.categories;
    setCategories(cats);
  }, []);

  return (
    <div className="home-page">
      <h1>스페인어-한국어 학습앱</h1>
      <SearchBar onSearch={setSearchResults} />
      
      {searchResults ? (
        <div className="search-results">
          {/* 검색 결과 표시 */}
        </div>
      ) : (
        <CategoryList categories={categories} />
      )}
    </div>
  );
}

export default HomePage;
```

#### QuizPage.jsx

```jsx
import React, { useEffect, useState } from 'react';
import { useParams } from 'react-router-dom';
import QuizScreen from '../components/QuizScreen';
import ResultScreen from '../components/ResultScreen';
import wordsData from '../data/words.json';
import { saveProgress, saveWrongAnswer } from '../services/indexedDB';

function QuizPage() {
  const { categoryId } = useParams();
  const [words, setWords] = useState([]);
  const [currentIndex, setCurrentIndex] = useState(0);
  const [results, setResults] = useState(null);
  const [score, setScore] = useState(0);

  useEffect(() => {
    // Phase 1: 정적 JSON에서 카테고리별 단어 로드
    const categoryWords = wordsData.words
      .filter(w => w.category_id === Number(categoryId));
    // 랜덤 10개 선택
    const shuffled = categoryWords.sort(() => Math.random() - 0.5).slice(0, 10);
    setWords(shuffled);
  }, [categoryId]);

  // 같은 카테고리에서 오답 선택지 생성
  const generateChoices = (correctWord) => {
    const categoryWords = wordsData.words
      .filter(w => w.category_id === correctWord.category_id && w.id !== correctWord.id);
    
    // 카테고리 내 단어가 3개 미만이면 다른 카테고리에서 보충
    let pool = [...categoryWords];
    if (pool.length < 3) {
      const otherWords = wordsData.words
        .filter(w => w.id !== correctWord.id && w.category_id !== correctWord.category_id);
      pool = [...pool, ...otherWords];
    }

    // 랜덤 3개 오답 선택
    const wrongChoices = pool.sort(() => Math.random() - 0.5).slice(0, 3);
    
    // 정답 포함하여 셔플
    const allChoices = [correctWord, ...wrongChoices].sort(() => Math.random() - 0.5);
    return allChoices;
  };

  const handleAnswer = async (selectedWord, correctWord, quizType) => {
    const isCorrect = selectedWord.id === correctWord.id;

    // IndexedDB에 진행상황 저장
    await saveProgress(correctWord.id, isCorrect);

    // 오답이면 오답 노트에 저장
    if (!isCorrect) {
      await saveWrongAnswer({
        word_id: correctWord.id,
        quiz_type: quizType,
        user_answer: quizType === 'spanish_to_korean' ? selectedWord.korean : selectedWord.spanish,
        correct_answer: quizType === 'spanish_to_korean' ? correctWord.korean : correctWord.spanish
      });
    }

    if (isCorrect) setScore(score + 1);

    if (currentIndex < words.length - 1) {
      setCurrentIndex(currentIndex + 1);
    } else {
      setResults({
        total: words.length,
        correct: score + (isCorrect ? 1 : 0)
      });
    }
  };

  if (results) {
    return <ResultScreen results={results} />;
  }

  if (words.length === 0) return <div>로딩 중...</div>;

  return (
    <QuizScreen 
      word={words[currentIndex]}
      choices={generateChoices(words[currentIndex])}
      onAnswer={handleAnswer}
      currentIndex={currentIndex}
      total={words.length}
    />
  );
}

export default QuizPage;
```

#### QuizScreen.jsx

```jsx
import React, { useState } from 'react';
import { speakSpanish } from '../services/tts';

function QuizScreen({ word, choices, onAnswer, currentIndex, total }) {
  const [quizType] = useState(
    Math.random() > 0.5 ? 'spanish_to_korean' : 'korean_to_spanish'
  );
  const [selectedAnswer, setSelectedAnswer] = useState(null);

  const getQuestion = () => {
    return quizType === 'spanish_to_korean' 
      ? word.spanish 
      : word.korean;
  };

  const getChoiceText = (choice) => {
    return quizType === 'spanish_to_korean'
      ? choice.korean
      : choice.spanish;
  };

  const handleSelectAnswer = (choice) => {
    setSelectedAnswer(choice);
    
    // 1초 후 다음 문제로
    setTimeout(() => {
      onAnswer(choice, word, quizType);
      setSelectedAnswer(null);
    }, 1000);
  };

  const handlePlayAudio = () => {
    if (word.audio_url) {
      // Phase 2: mp3 재생
      new Audio(word.audio_url).play();
    } else {
      // Phase 1: Web Speech API
      speakSpanish(word.spanish);
    }
  };

  return (
    <div className="quiz-screen">
      <div className="progress">
        {currentIndex + 1} / {total}
      </div>

      <div className="question">
        <button onClick={handlePlayAudio}>🔊 발음 듣기</button>
        <p>{getQuestion()}</p>
      </div>

      <div className="choices">
        {choices.map(choice => (
          <button
            key={choice.id}
            onClick={() => handleSelectAnswer(choice)}
            className={`choice ${
              selectedAnswer?.id === choice.id
                ? choice.id === word.id ? 'correct' : 'wrong'
                : ''
            }`}
            disabled={selectedAnswer !== null}
          >
            {getChoiceText(choice)}
          </button>
        ))}
      </div>
    </div>
  );
}

export default QuizScreen;
```

#### WrongAnswerNotes.jsx

```jsx
import React, { useEffect, useState } from 'react';
import { getWrongAnswers, deleteWrongAnswer } from '../services/indexedDB';
import wordsData from '../data/words.json';

function WrongAnswerNotes() {
  const [wrongAnswers, setWrongAnswers] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    loadWrongAnswers();
  }, []);

  const loadWrongAnswers = async () => {
    try {
      const data = await getWrongAnswers();
      // word_id로 단어 정보 조회 (JOIN 대체)
      const enriched = data.map(answer => {
        const word = wordsData.words.find(w => w.id === answer.word_id);
        const category = wordsData.categories.find(c => c.id === word?.category_id);
        return {
          ...answer,
          spanish: word?.spanish,
          korean: word?.korean,
          category_name: category?.name,
          category_emoji: category?.emoji
        };
      });
      setWrongAnswers(enriched);
    } catch (error) {
      console.error('오답 로드 실패:', error);
    } finally {
      setLoading(false);
    }
  };

  const handleDelete = async (id) => {
    try {
      await deleteWrongAnswer(id);
      setWrongAnswers(wrongAnswers.filter(ans => ans.id !== id));
    } catch (error) {
      console.error('삭제 실패:', error);
    }
  };

  if (loading) return <div>로딩 중...</div>;

  return (
    <div className="wrong-answers">
      <h2>오답 노트 ({wrongAnswers.length})</h2>
      
      {wrongAnswers.map(answer => (
        <div key={answer.id} className="wrong-answer-item">
          <span>{answer.category_emoji} {answer.category_name}</span>
          <h4>{answer.spanish}</h4>
          <p>
            내 답: <strong>{answer.user_answer}</strong> 
            → 정답: <strong>{answer.correct_answer}</strong>
          </p>
          <p className="korean">{answer.korean}</p>
          <button onClick={() => handleDelete(answer.id)}>삭제</button>
        </div>
      ))}
    </div>
  );
}

export default WrongAnswerNotes;
```

### 3. 서비스 레이어

#### services/tts.js (Web Speech API)

```javascript
/**
 * Web Speech API를 사용한 스페인어 TTS
 * Phase 1: 이 파일 사용
 * Phase 2: audio_url이 있으면 mp3 재생, 없으면 이 파일로 fallback
 */

export const speakSpanish = (text) => {
  if (!('speechSynthesis' in window)) {
    console.warn('이 브라우저는 TTS를 지원하지 않습니다');
    return;
  }

  // 이전 발화 중지
  window.speechSynthesis.cancel();

  const utterance = new SpeechSynthesisUtterance(text);
  utterance.lang = 'es-ES';  // 스페인어 (스페인)
  utterance.rate = 0.9;       // 약간 느리게 (학습용)
  utterance.pitch = 1;

  window.speechSynthesis.speak(utterance);
};

export const stopSpeaking = () => {
  if ('speechSynthesis' in window) {
    window.speechSynthesis.cancel();
  }
};
```

#### services/indexedDB.js

```javascript
let db;

// IndexedDB 초기화
export const initializeDB = async () => {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open('SpanishKoreanApp', 1);

    request.onerror = () => reject(request.error);
    request.onsuccess = () => {
      db = request.result;
      resolve(db);
    };

    request.onupgradeneeded = (event) => {
      db = event.target.result;

      // Words 캐시 저장소 (Phase 2: API 데이터 캐시용)
      if (!db.objectStoreNames.contains('words')) {
        db.createObjectStore('words', { keyPath: 'id' });
      }

      // Progress 저장소
      if (!db.objectStoreNames.contains('progress')) {
        db.createObjectStore('progress', { keyPath: 'word_id' });
      }

      // WrongAnswers 저장소
      if (!db.objectStoreNames.contains('wrongAnswers')) {
        const store = db.createObjectStore('wrongAnswers', { keyPath: 'id', autoIncrement: true });
        store.createIndex('createdAt', 'created_at', { unique: false });
      }
    };
  });
};

// 진행상황 저장 (Phase 1: correct_count, wrong_count만 사용)
export const saveProgress = async (wordId, isCorrect) => {
  const tx = db.transaction('progress', 'readwrite');
  const store = tx.objectStore('progress');

  const progress = await new Promise((resolve) => {
    const req = store.get(wordId);
    req.onsuccess = () => resolve(req.result);
    req.onerror = () => resolve(null);
  });

  const updatedProgress = {
    word_id: wordId,
    correct_count: (progress?.correct_count || 0) + (isCorrect ? 1 : 0),
    wrong_count: (progress?.wrong_count || 0) + (isCorrect ? 0 : 1),
    last_reviewed_at: new Date().toISOString()
  };

  store.put(updatedProgress);

  return new Promise((resolve, reject) => {
    tx.oncomplete = () => resolve(updatedProgress);
    tx.onerror = () => reject(tx.error);
  });
};

// 오답 저장
export const saveWrongAnswer = async (data) => {
  const tx = db.transaction('wrongAnswers', 'readwrite');
  const store = tx.objectStore('wrongAnswers');

  store.add({
    ...data,
    created_at: new Date().toISOString()
  });

  return new Promise((resolve, reject) => {
    tx.oncomplete = () => resolve();
    tx.onerror = () => reject(tx.error);
  });
};

// 오답 목록 조회
export const getWrongAnswers = async () => {
  const tx = db.transaction('wrongAnswers', 'readonly');
  const store = tx.objectStore('wrongAnswers');

  return new Promise((resolve, reject) => {
    const req = store.getAll();
    req.onsuccess = () => resolve(req.result);
    req.onerror = () => reject(req.error);
  });
};

// 오답 삭제
export const deleteWrongAnswer = async (id) => {
  const tx = db.transaction('wrongAnswers', 'readwrite');
  const store = tx.objectStore('wrongAnswers');

  store.delete(id);

  return new Promise((resolve, reject) => {
    tx.oncomplete = () => resolve();
    tx.onerror = () => reject(tx.error);
  });
};

// 전체 진행상황 조회
export const getAllProgress = async () => {
  const tx = db.transaction('progress', 'readonly');
  const store = tx.objectStore('progress');

  return new Promise((resolve, reject) => {
    const req = store.getAll();
    req.onsuccess = () => resolve(req.result);
    req.onerror = () => reject(req.error);
  });
};
```

#### services/dataExport.js (진행상황 내보내기/가져오기)

```javascript
import { getAllProgress, getWrongAnswers, saveProgress, saveWrongAnswer } from './indexedDB';

/**
 * Phase 1: 수동 내보내기/가져오기로 기기 간 데이터 이동
 * Phase 2: 서버 동기화로 대체
 */

// 진행상황 + 오답 데이터를 JSON 파일로 내보내기
export const exportData = async () => {
  const progress = await getAllProgress();
  const wrongAnswers = await getWrongAnswers();

  const exportPayload = {
    version: 1,
    exported_at: new Date().toISOString(),
    progress,
    wrong_answers: wrongAnswers
  };

  const blob = new Blob([JSON.stringify(exportPayload, null, 2)], { type: 'application/json' });
  const url = URL.createObjectURL(blob);

  const a = document.createElement('a');
  a.href = url;
  a.download = `spanish-korean-backup-${new Date().toISOString().slice(0, 10)}.json`;
  a.click();

  URL.revokeObjectURL(url);
};

// JSON 파일에서 데이터 가져오기
export const importData = async (file) => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();

    reader.onload = async (e) => {
      try {
        const data = JSON.parse(e.target.result);

        if (!data.version || !data.progress) {
          throw new Error('올바른 백업 파일이 아닙니다');
        }

        // 진행상황 복원
        for (const p of data.progress) {
          await saveProgress(p.word_id, null, p); // 직접 덮어쓰기
        }

        // 오답 복원
        if (data.wrong_answers) {
          for (const wa of data.wrong_answers) {
            await saveWrongAnswer(wa);
          }
        }

        resolve({
          imported_progress: data.progress.length,
          imported_wrong_answers: data.wrong_answers?.length || 0
        });
      } catch (error) {
        reject(error);
      }
    };

    reader.onerror = () => reject(reader.error);
    reader.readAsText(file);
  });
};
```

### 4. API 서비스 (Phase 2)

#### services/api.js

```javascript
/**
 * Phase 2에서 활성화
 * Phase 1에서는 이 파일을 사용하지 않음
 */

const API_URL = import.meta.env.VITE_API_URL || 'http://localhost:8000/api';

// 인증 헤더
const getHeaders = () => {
  const token = localStorage.getItem('auth_token');
  return {
    'Content-Type': 'application/json',
    ...(token && { 'Authorization': `Bearer ${token}` })
  };
};

// 카테고리 조회
export const fetchCategories = async () => {
  const response = await fetch(`${API_URL}/categories`);
  if (!response.ok) throw new Error('카테고리 로드 실패');
  const data = await response.json();
  return data.data;
};

// 단어 조회
export const fetchWords = async (categoryId, limit = 50) => {
  const params = new URLSearchParams();
  if (categoryId) params.append('category_id', categoryId);
  params.append('limit', limit);

  const response = await fetch(`${API_URL}/words?${params}`);
  if (!response.ok) throw new Error('단어 로드 실패');
  const data = await response.json();
  return data.data.words;
};

// 단어 검색
export const searchWords = async (query) => {
  const response = await fetch(`${API_URL}/words/search?q=${query}`);
  if (!response.ok) throw new Error('검색 실패');
  const data = await response.json();
  return data.data;
};

// 진행상황 조회
export const fetchProgress = async () => {
  const response = await fetch(`${API_URL}/progress`, {
    headers: getHeaders()
  });
  if (!response.ok) throw new Error('진행상황 로드 실패');
  const data = await response.json();
  return data.data;
};

// 오답 조회
export const fetchWrongAnswers = async (limit = 20) => {
  const response = await fetch(`${API_URL}/wrong-answers?limit=${limit}`, {
    headers: getHeaders()
  });
  if (!response.ok) throw new Error('오답 로드 실패');
  const data = await response.json();
  return data.data.wrong_answers;
};

// 오답 삭제
export const deleteWrongAnswer = async (id) => {
  const response = await fetch(`${API_URL}/wrong-answers/${id}`, {
    method: 'DELETE',
    headers: getHeaders()
  });
  if (!response.ok) throw new Error('삭제 실패');
  return true;
};

// 로그인
export const login = async (email, password) => {
  const response = await fetch(`${API_URL}/login`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password })
  });
  if (!response.ok) throw new Error('로그인 실패');
  const data = await response.json();
  localStorage.setItem('auth_token', data.data.token);
  return data.data;
};

// 로그아웃
export const logout = async () => {
  const response = await fetch(`${API_URL}/logout`, {
    method: 'POST',
    headers: getHeaders()
  });
  if (!response.ok) throw new Error('로그아웃 실패');
  localStorage.removeItem('auth_token');
  return true;
};

// 데이터 마이그레이션 (IndexedDB → 서버)
export const migrateProgress = async (progressData, wrongAnswers) => {
  const response = await fetch(`${API_URL}/migrate-progress`, {
    method: 'POST',
    headers: getHeaders(),
    body: JSON.stringify({ progress_data: progressData, wrong_answers: wrongAnswers })
  });
  if (!response.ok) throw new Error('마이그레이션 실패');
  const data = await response.json();
  return data.data;
};
```

---

## 개발 플로우

### Phase 1: 프론트엔드 완성 (2-3주) — 서버 없음

#### Week 1: 기본 UI + 퀴즈 로직

- [ ] React 프로젝트 셋업 (Vite + React Router)
- [ ] 정적 words.json 데이터 준비 (CSV → JSON 변환)
- [ ] HomePage (카테고리 선택)
- [ ] QuizScreen (단어 표시 + 4지선다)
  - [ ] 같은 카테고리에서 오답 선택지 생성
  - [ ] spanish_to_korean / korean_to_spanish 랜덤 전환
- [ ] ResultScreen (결과 표시)
- [ ] Navigation (라우팅)

#### Week 2: 데이터 저장 + 오디오

- [ ] IndexedDB 구현
  - [ ] 진행상황 저장 (correct_count, wrong_count)
  - [ ] 오답 기록 저장
- [ ] Web Speech API TTS 구현
  - [ ] 스페인어 발음 재생
  - [ ] audio_url fallback 로직 (Phase 2 대비)
- [ ] 오답 노트 페이지
  - [ ] word_id JOIN으로 카테고리 정보 표시
  - [ ] 삭제 기능
- [ ] 검색 기능 (로컬 필터링)

#### Week 3: PWA + 내보내기/가져오기

- [ ] PWA 설정 (manifest.json, Service Worker)
- [ ] 진행상황 내보내기 (JSON 다운로드)
- [ ] 진행상황 가져오기 (JSON 업로드)
- [ ] ProfilePage (학습 통계, 내보내기/가져오기 UI)
- [ ] 반응형 디자인 + 스타일링

### Phase 2: 백엔드 연동 (2-3주)

#### Week 4-5: Laravel API

- [ ] Laravel 프로젝트 셋업
- [ ] 데이터베이스 마이그레이션
- [ ] Seeder로 3,700개 단어 추가
- [ ] API 구현
  - [ ] WordController (GET /words, GET /categories, GET /words/search)
  - [ ] AuthController (회원가입, 로그인, 로그아웃)
  - [ ] ProgressController (진행상황 CRUD)
  - [ ] WrongAnswerController (오답 CRUD, category JOIN)
  - [ ] MigrationController (IndexedDB → 서버 마이그레이션)
- [ ] React ↔ API 연결
  - [ ] 로그인 상태: 서버 동기화
  - [ ] 비로그인 상태: 기존 IndexedDB 유지

#### Week 6: 고급 기능

- [ ] Spaced Repetition (Leitner System)
  - [ ] next_review_date 계산 (1→3→7→14→30일)
  - [ ] streak, is_learned 활성화
- [ ] Filament 관리자 패널
  - [ ] WordResource (단어 CRUD)
  - [ ] CategoryResource (카테고리 CRUD)
- [ ] Google TTS mp3 생성 + audio_url 업데이트 (선택)

### Phase 3: 최적화 (1주)

- [ ] 성능 최적화 (API 캐싱, 압축)
- [ ] 보안 (CORS, Rate limiting, JWT)
- [ ] Vercel + Hostinger 배포
- [ ] 테스트

---

## 배포 가이드

### Vercel에 React 배포

```bash
cd frontend

# Vercel CLI 설치
npm install -g vercel

# 배포
vercel

# 환경 변수 설정
# .env.production
VITE_API_URL=https://your-backend.hostinger.com/api
VITE_PHASE=2
```

### Hostinger에 Laravel 배포 (Phase 2)

```bash
# 1. SSH 접속
ssh u123456@your-domain.com

# 2. 데이터베이스 생성
# Hostinger hPanel → MySQL Databases

# 3. 파일 업로드 (Git 또는 SFTP)
git clone https://github.com/yourusername/spanish-korean-app.git
cd spanish-korean-app/backend

# 4. 의존성 설치
composer install --optimize-autoloader --no-dev

# 5. .env 설정
cp .env.example .env
nano .env
# DB_HOST, DB_DATABASE, DB_USERNAME, DB_PASSWORD 설정

# 6. 앱 키 생성
php artisan key:generate

# 7. 마이그레이션 실행
php artisan migrate --force

# 8. Seeder 실행
php artisan db:seed
```

### 로컬에서 테스트

```bash
# Phase 1: React만
cd frontend && npm run dev
# http://localhost:5173

# Phase 2: React + Laravel
# Terminal 1
cd frontend && npm run dev
# Terminal 2
cd backend && php artisan serve
# http://localhost:5173 (React)
# http://localhost:8000/api (Laravel)
```

---

## 주의사항

### .env 파일 관리

```
✅ .gitignore에 추가할 것:
.env
.env.local
.env.*.local
node_modules/
vendor/
```

### CORS 설정 (Laravel - Phase 2)

```php
// config/cors.php
'paths' => ['api/*'],
'allowed_methods' => ['*'],
'allowed_origins' => [
    'http://localhost:5173',  // 로컬 개발
    'https://your-frontend.vercel.app'  // 프로덕션
],
```

### Git 커밋 메시지 컨벤션

```
feat: 새 기능 추가
fix: 버그 수정
docs: 문서 수정
style: 코드 포매팅
refactor: 코드 리팩토링
test: 테스트 추가
chore: 의존성 업데이트 등
```

---

## 필요한 패키지

### Frontend (package.json)

```json
{
  "dependencies": {
    "react": "^18.0.0",
    "react-dom": "^18.0.0",
    "react-router-dom": "^6.0.0"
  },
  "devDependencies": {
    "vite": "^latest",
    "@vitejs/plugin-react": "^latest",
    "vite-plugin-pwa": "^latest"
  }
}
```

### Backend (composer.json) - Phase 2

```json
{
  "require": {
    "laravel/framework": "^11.0",
    "filament/filament": "^3.0"
  }
}
```

---

**마지막 업데이트:** 2026-03-06  
**작성자:** Development Guide  
**상태:** Phase 1 준비 중
