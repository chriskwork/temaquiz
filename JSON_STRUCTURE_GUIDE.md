# JSON 구조 완벽 가이드

> 로컬 상태 → IndexedDB → (Phase 2: API → DB) 전체 데이터 흐름

---

## 1️⃣ 전체 데이터 흐름도

### Phase 1 (서버 없음)

```
┌─────────────────────────────────────────────────────────────┐
│                    사용자 브라우저 (PWA)                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  정적 데이터 (words.json)                                    │
│  ├─ categories: Category[]                                  │
│  └─ words: Word[]                                           │
│                  ↓ (앱 시작 시 로드)                           │
│  React State (메모리)                                        │
│  ├─ currentWord: Word                                       │
│  ├─ choices: Word[] (같은 카테고리에서 4지선다)              │
│  ├─ userProgress: Progress[]                                │
│  └─ wrongAnswers: WrongAnswer[]                             │
│                  ↓ (저장)                                     │
│  IndexedDB (브라우저 저장소)                                 │
│  ├─ Store: progress → {word_id, correct_count, ...}        │
│  └─ Store: wrongAnswers → {word_id, user_answer, ...}      │
│                                                              │
│  Web Speech API (TTS)                                       │
│  └─ speakSpanish(word.spanish) → 발음 재생                 │
│                                                              │
│  내보내기/가져오기 (JSON 파일)                               │
│  ├─ exportData() → backup.json 다운로드                     │
│  └─ importData(file) → IndexedDB 복원                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### Phase 2 (서버 추가)

```
┌─────────────────────────────────────────────────────────────┐
│                    사용자 브라우저 (PWA)                      │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  React State (메모리)                                        │
│  ├─ currentWord: Word                                       │
│  ├─ userProgress: Progress[]                                │
│  ├─ wrongAnswers: WrongAnswer[]                             │
│  └─ authToken: string                                       │
│                  ↓ (저장)                                     │
│  IndexedDB (오프라인 캐시)                                   │
│  ├─ Store: words → {id, spanish, korean, ...}              │
│  ├─ Store: progress → {word_id, correct_count, ...}        │
│  └─ Store: wrongAnswers → {word_id, user_answer, ...}      │
│                  ↓ (동기화)                                   │
│  API 요청/응답                                              │
│  ├─ GET /api/words → {success, data: Word[]}               │
│  ├─ POST /api/progress → {success, data: Progress}         │
│  └─ GET /api/wrong-answers → {success, data: WrongAnswer[]}│
│                                                              │
│  오디오 재생                                                 │
│  ├─ audio_url 있으면 → mp3 재생                             │
│  └─ audio_url 없으면 → Web Speech API fallback              │
│                                                              │
└─────────────────────────────────────────────────────────────┘
                          ↓ (HTTPS 전송)
┌─────────────────────────────────────────────────────────────┐
│                    Laravel 백엔드 (Hostinger)                │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Router: /api/words → WordController@index                  │
│  Router: /api/progress → ProgressController@store           │
│  Router: /api/wrong-answers → WrongAnswerController@index   │
│                  ↓                                            │
│  Models (비즈니스 로직)                                      │
│  ├─ Word::where('category_id', $id)->get()                 │
│  ├─ UserProgress::where('user_id', auth()->id())           │
│  └─ WrongAnswer::where('user_id', auth()->id())            │
│                  ↓                                            │
│  MySQL Database (저장소)                                    │
│  ├─ Table: categories                                       │
│  ├─ Table: words                                            │
│  ├─ Table: users                                            │
│  ├─ Table: user_progress                                    │
│  └─ Table: wrong_answers                                    │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## 2️⃣ 핵심 JSON 구조들

### A. Word (단어 데이터)

#### Phase 1: 정적 JSON (words.json)
```json
{
  "categories": [
    { "id": 1, "name": "인사", "emoji": "👋", "slug": "greeting" },
    { "id": 2, "name": "식당에서", "emoji": "🍽️", "slug": "restaurant" }
  ],
  "words": [
    {
      "id": 1,
      "category_id": 1,
      "spanish": "hola",
      "korean": "안녕하세요",
      "example_spanish": "¡Hola, cómo estás?",
      "example_korean": "안녕하세요, 어떻게 지내세요?",
      "difficulty": "easy"
    }
  ]
}
```

> **참고:** Phase 1에서는 `audio_url` 필드가 없음.
> Web Speech API로 발음 재생.

#### React State
```javascript
// components/QuizScreen.jsx
const [currentWord, setCurrentWord] = useState({
  id: 1,
  spanish: "hola",
  korean: "안녕하세요",
  category_id: 1,
  example_spanish: "¡Hola, cómo estás?",
  example_korean: "안녕하세요, 어떻게 지내세요?",
  difficulty: "easy"
});
```

#### Phase 2: API 응답 형식
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
        "example_spanish": "¡Hola, cómo estás?",
        "example_korean": "안녕하세요, 어떻게 지내세요?",
        "difficulty": "easy"
      }
    ],
    "total": 3700,
    "category": {
      "id": 1,
      "name": "인사",
      "emoji": "👋"
    }
  }
}
```

#### Phase 2: IndexedDB 캐시
```javascript
// IndexedDB의 'words' store
{
  id: 1,
  spanish: "hola",
  korean: "안녕하세요",
  category_id: 1,
  audio_url: null,
  example_spanish: "¡Hola, cómo estás?",
  example_korean: "안녕하세요, 어떻게 지내세요?",
  difficulty: "easy",
  synced_at: "2026-03-06T10:00:00Z"
}
```

#### MySQL에 저장되는 형태
```sql
-- words 테이블
INSERT INTO words VALUES (
  1,                                    -- id
  1,                                    -- category_id
  'hola',                              -- spanish
  '안녕하세요',                         -- korean
  '¡Hola, cómo estás?',               -- example_spanish
  '안녕하세요, 어떻게 지내세요?',      -- example_korean
  NULL,                                -- audio_url (Phase 2에서 채움)
  'easy',                              -- difficulty
  1,                                    -- is_active
  '2026-03-06 10:00:00',              -- created_at
  '2026-03-06 10:00:00'               -- updated_at
);
```

---

### B. Category (카테고리)

#### Phase 1: words.json 내 포함
```javascript
// words.json에서 추출
const categories = wordsData.categories;
// [
//   { id: 1, name: "인사", emoji: "👋", slug: "greeting" },
//   { id: 2, name: "식당에서", emoji: "🍽️", slug: "restaurant" }
// ]
```

#### React State
```javascript
const [categories, setCategories] = useState([
  {
    id: 1,
    name: "인사",
    emoji: "👋",
    slug: "greeting",
    word_count: 50
  },
  {
    id: 2,
    name: "식당에서",
    emoji: "🍽️",
    slug: "restaurant",
    word_count: 45
  }
]);
```

#### Phase 2: API 응답
```json
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "인사",
      "emoji": "👋",
      "slug": "greeting",
      "description": "기본 인사말",
      "icon_url": null,
      "word_count": 50
    },
    {
      "id": 2,
      "name": "식당에서",
      "emoji": "🍽️",
      "slug": "restaurant",
      "description": "음식점 관련 표현",
      "icon_url": null,
      "word_count": 45
    }
  ]
}
```

#### MySQL
```sql
INSERT INTO categories VALUES (
  1,
  '인사',
  '기본 인사말',
  '👋',
  null,
  'greeting',
  1,
  '2026-03-06 10:00:00',
  '2026-03-06 10:00:00'
);
```

---

### C. Progress (학습 진행상황)

#### Phase 1: IndexedDB에 저장 (단순 버전)
```javascript
// IndexedDB의 'progress' store (키: word_id)
{
  word_id: 1,
  correct_count: 3,
  wrong_count: 1,
  last_reviewed_at: "2026-03-06T10:30:00Z"
}
```

> **Phase 1에서는** `correct_count`, `wrong_count`, `last_reviewed_at`만 사용.
> `next_review_date`, `streak`, `is_learned`은 Phase 2에서 활성화.

#### React State (Phase 1)
```javascript
const [progress, setProgress] = useState({
  word_id: 1,
  correct_count: 3,
  wrong_count: 1,
  last_reviewed_at: "2026-03-06T10:30:00Z"
});
```

#### Phase 2: IndexedDB (확장 버전)
```javascript
// Phase 2에서 추가되는 필드
{
  word_id: 1,
  correct_count: 3,
  wrong_count: 1,
  last_reviewed_at: "2026-03-06T10:30:00Z",
  next_review_date: "2026-03-08T10:30:00Z",
  difficulty: "easy",
  streak: 3,
  is_learned: false,
  synced_with_server: false,
  synced_at: null
}
```

#### Phase 2: API 요청 (사용자가 답변 제출)
```json
{
  "word_id": 1,
  "quiz_type": "spanish_to_korean",
  "is_correct": true,
  "answered_at": "2026-03-06T10:30:00Z"
}
```

#### Phase 2: API 응답
```json
{
  "success": true,
  "data": {
    "user_id": 5,
    "word_id": 1,
    "correct_count": 4,
    "wrong_count": 1,
    "is_learned": false,
    "next_review_date": "2026-03-08T10:30:00Z",
    "streak": 4,
    "difficulty": "easy"
  }
}
```

#### MySQL
```sql
INSERT INTO user_progress VALUES (
  1,                                    -- id
  5,                                    -- user_id
  1,                                    -- word_id
  4,                                    -- correct_count
  1,                                    -- wrong_count
  '2026-03-06 10:30:00',              -- last_reviewed_at
  '2026-03-08 10:30:00',              -- next_review_date
  'easy',                              -- difficulty
  4,                                    -- streak
  0,                                    -- is_learned
  '2026-03-05 09:00:00',              -- created_at
  '2026-03-06 10:30:00'               -- updated_at
) ON DUPLICATE KEY UPDATE
  correct_count = 4,
  last_reviewed_at = '2026-03-06 10:30:00',
  next_review_date = '2026-03-08 10:30:00';
```

---

### D. WrongAnswer (오답 기록)

#### Phase 1: IndexedDB에 저장
```javascript
// IndexedDB의 'wrongAnswers' store
{
  id: 1,                                     // autoIncrement
  word_id: 2,
  quiz_type: "spanish_to_korean",
  user_answer: "주스",
  correct_answer: "물",
  created_at: "2026-03-05T14:20:00Z"
}
```

> **변경:** `category_name` 저장 안 함.
> 화면 표시 시 `word_id` → words.json에서 JOIN하여 카테고리 정보 조회.

#### React State (화면 표시용, JOIN 후)
```javascript
const [wrongAnswers, setWrongAnswers] = useState([
  {
    id: 1,
    word_id: 2,
    spanish: "agua",              // words.json에서 JOIN
    korean: "물",                 // words.json에서 JOIN
    quiz_type: "spanish_to_korean",
    user_answer: "주스",
    correct_answer: "물",
    category_name: "식당에서",    // words.json → categories에서 JOIN
    category_emoji: "🍽️",        // words.json → categories에서 JOIN
    created_at: "2026-03-05T14:20:00Z"
  }
]);
```

#### Phase 2: API 요청 (오답 저장)
```json
{
  "word_id": 2,
  "quiz_type": "spanish_to_korean",
  "user_answer": "주스",
  "correct_answer": "물"
}
```

#### Phase 2: API 응답 (단건)
```json
{
  "success": true,
  "data": {
    "id": 101,
    "user_id": 5,
    "word_id": 2,
    "quiz_type": "spanish_to_korean",
    "user_answer": "주스",
    "correct_answer": "물",
    "created_at": "2026-03-05T14:20:00Z"
  }
}
```

#### Phase 2: API 응답 (목록 조회, JOIN 포함)
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
        "quiz_type": "spanish_to_korean",
        "user_answer": "주스",
        "correct_answer": "물",
        "category": {
          "id": 2,
          "name": "식당에서",
          "emoji": "🍽️"
        },
        "created_at": "2026-03-05T14:20:00Z",
        "is_resolved": false
      }
    ]
  }
}
```

> **변경:** `category_name` 문자열 대신 `category` 객체로 JOIN하여 반환.

#### MySQL
```sql
INSERT INTO wrong_answers VALUES (
  101,                                  -- id
  5,                                    -- user_id
  2,                                    -- word_id
  'spanish_to_korean',                 -- quiz_type
  '주스',                               -- user_answer
  '물',                                 -- correct_answer
  NULL,                                -- explanation
  0,                                    -- is_resolved
  '2026-03-05 14:20:00',              -- created_at
  '2026-03-05 14:20:00'               -- updated_at
);

-- 카테고리 정보는 JOIN으로 조회
SELECT wa.*, w.spanish, w.korean, c.name as category_name, c.emoji as category_emoji
FROM wrong_answers wa
JOIN words w ON wa.word_id = w.id
JOIN categories c ON w.category_id = c.id
WHERE wa.user_id = 5;
```

---

### E. User & Auth (Phase 2)

#### 회원가입 요청
```json
{
  "name": "김철수",
  "email": "kim@example.com",
  "password": "password123",
  "password_confirmation": "password123"
}
```

#### 로그인 요청
```json
{
  "email": "kim@example.com",
  "password": "password123"
}
```

#### 인증 응답
```json
{
  "success": true,
  "data": {
    "id": 5,
    "name": "김철수",
    "email": "kim@example.com",
    "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
    "token_type": "Bearer"
  }
}
```

#### React State (로컬 저장)
```javascript
const [user, setUser] = useState({
  id: 5,
  name: "김철수",
  email: "kim@example.com"
});

// localStorage에 저장
localStorage.setItem('auth_token', 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...');
```

#### MySQL
```sql
INSERT INTO users VALUES (
  5,
  '김철수',
  'kim@example.com',
  null,
  '$2y$12$...(암호화된 비밀번호)',
  null,
  1,
  '2026-03-06 10:30:00',
  '2026-03-06 10:30:00',
  '2026-03-06 10:30:00'
);
```

---

### F. 내보내기/가져오기 데이터 (Phase 1)

#### 내보내기 JSON 형식
```json
{
  "version": 1,
  "exported_at": "2026-03-06T15:00:00Z",
  "progress": [
    {
      "word_id": 1,
      "correct_count": 5,
      "wrong_count": 2,
      "last_reviewed_at": "2026-03-06T10:30:00Z"
    },
    {
      "word_id": 2,
      "correct_count": 3,
      "wrong_count": 1,
      "last_reviewed_at": "2026-03-05T14:20:00Z"
    }
  ],
  "wrong_answers": [
    {
      "word_id": 2,
      "quiz_type": "spanish_to_korean",
      "user_answer": "주스",
      "correct_answer": "물",
      "created_at": "2026-03-05T14:20:00Z"
    }
  ]
}
```

> **파일명 패턴:** `spanish-korean-backup-2026-03-06.json`
> Phase 2에서 서버 동기화가 추가되면 이 기능은 보조 수단으로 유지.

---

## 3️⃣ 실제 데이터 흐름 예시

### 상황: Phase 1에서 사용자가 퀴즈를 풀고 답변을 제출

#### Step 1: 정적 JSON에서 퀴즈 데이터 로드

```javascript
// QuizPage.jsx
import wordsData from '../data/words.json';

// 카테고리별 단어 필터
const categoryWords = wordsData.words
  .filter(w => w.category_id === Number(categoryId));

// 랜덤 10개 선택
const quizWords = categoryWords
  .sort(() => Math.random() - 0.5)
  .slice(0, 10);
```

#### Step 2: 같은 카테고리에서 4지선다 생성

```javascript
// 같은 카테고리에서 오답 선택지 3개 뽑기
const generateChoices = (correctWord) => {
  const sameCategory = wordsData.words
    .filter(w => w.category_id === correctWord.category_id && w.id !== correctWord.id);
  
  let pool = [...sameCategory];
  // 카테고리 내 단어가 3개 미만이면 다른 카테고리에서 보충
  if (pool.length < 3) {
    const others = wordsData.words
      .filter(w => w.id !== correctWord.id && w.category_id !== correctWord.category_id);
    pool = [...pool, ...others];
  }

  const wrongChoices = pool.sort(() => Math.random() - 0.5).slice(0, 3);
  return [correctWord, ...wrongChoices].sort(() => Math.random() - 0.5);
};
```

#### Step 3: 오디오 재생 (Web Speech API)

```javascript
// services/tts.js
const handlePlayAudio = (word) => {
  if (word.audio_url) {
    // Phase 2: mp3 재생
    new Audio(word.audio_url).play();
  } else {
    // Phase 1: Web Speech API
    const utterance = new SpeechSynthesisUtterance(word.spanish);
    utterance.lang = 'es-ES';
    utterance.rate = 0.9;
    window.speechSynthesis.speak(utterance);
  }
};
```

#### Step 4: IndexedDB에 결과 저장

```javascript
// handleAnswer 함수
const handleAnswer = async (selectedWord, correctWord, quizType) => {
  const isCorrect = selectedWord.id === correctWord.id;

  // IndexedDB에 진행상황 저장
  await saveProgress(correctWord.id, isCorrect);
  // 결과: { word_id: 1, correct_count: 4, wrong_count: 1, last_reviewed_at: "..." }

  // 오답이면 오답 노트에도 저장
  if (!isCorrect) {
    await saveWrongAnswer({
      word_id: correctWord.id,
      quiz_type: quizType,
      user_answer: quizType === 'spanish_to_korean' 
        ? selectedWord.korean 
        : selectedWord.spanish,
      correct_answer: quizType === 'spanish_to_korean' 
        ? correctWord.korean 
        : correctWord.spanish
    });
  }
};
```

#### Step 5: IndexedDB에 저장된 상태

```javascript
// IndexedDB > progress store
{
  word_id: 1,
  correct_count: 4,
  wrong_count: 1,
  last_reviewed_at: "2026-03-06T10:30:00Z"
}

// IndexedDB > wrongAnswers store (오답인 경우)
{
  id: 1,
  word_id: 2,
  quiz_type: "spanish_to_korean",
  user_answer: "주스",
  correct_answer: "물",
  created_at: "2026-03-05T14:20:00Z"
}
```

---

## 4️⃣ Phase 2: 서버 동기화 흐름

### 상황: 회원 로그인 후 퀴즈 답변 제출

```javascript
const handleAnswer = async (isCorrect) => {
  // 1. 즉시 IndexedDB에 저장 (오프라인 지원)
  const updatedProgress = await saveProgress(wordId, isCorrect);

  // 2. 회원이면 서버로도 전송
  const token = localStorage.getItem('auth_token');
  if (token) {
    try {
      await fetch('/api/progress', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${token}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          word_id: wordId,
          quiz_type: 'spanish_to_korean',
          is_correct: isCorrect
        })
      });
    } catch (error) {
      console.error('서버 동기화 실패 - IndexedDB 데이터는 보존됨');
    }
  }
};
```

### Laravel 처리

```php
// ProgressController.php
public function store(Request $request)
{
    $data = $request->validate([
        'word_id' => 'required|integer',
        'is_correct' => 'required|boolean'
    ]);
    
    $progress = UserProgress::updateOrCreate(
        [
            'user_id' => auth()->id(),
            'word_id' => $data['word_id']
        ],
        [
            'correct_count' => DB::raw('correct_count + '.($data['is_correct'] ? 1 : 0)),
            'wrong_count' => DB::raw('wrong_count + '.($data['is_correct'] ? 0 : 1)),
            'last_reviewed_at' => now(),
            'next_review_date' => $this->calculateNextReview($data['is_correct'], $progress)
        ]
    );
    
    return response()->json([
        'success' => true,
        'data' => $progress
    ]);
}
```

---

## 5️⃣ Spaced Repetition 계산 로직 (Phase 2)

### Leitner System 기반 다음 복습 날짜 계산

```javascript
// Phase 2에서 활성화
function calculateNextReview(isCorrect, correctCount) {
  const now = new Date();
  
  let daysToAdd = 1;
  
  if (isCorrect) {
    // 정답: 맞춘 횟수에 따라 다음 복습 시간 결정
    if (correctCount === 1) daysToAdd = 1;      // 1일 후
    else if (correctCount === 2) daysToAdd = 3;  // 3일 후
    else if (correctCount === 3) daysToAdd = 7;  // 7일 후
    else if (correctCount === 4) daysToAdd = 14; // 14일 후
    else daysToAdd = 30;                         // 30일 후 (완전 학습)
  } else {
    daysToAdd = 1; // 오답: 내일 다시
  }
  
  now.setDate(now.getDate() + daysToAdd);
  return now.toISOString();
}

// 사용 예
const nextDate = calculateNextReview(true, 2);
// "2026-03-09T10:30:00Z" (3일 후)
```

### API 응답에 포함

```json
{
  "success": true,
  "data": {
    "word_id": 1,
    "correct_count": 3,
    "wrong_count": 1,
    "next_review_date": "2026-03-13T10:30:00Z",
    "difficulty": "easy",
    "is_learned": false
  }
}
```

---

## 6️⃣ 비회원 → 회원 전환 시 데이터 마이그레이션 (Phase 2)

### IndexedDB에 있던 데이터

```javascript
// IndexedDB progress 전체
[
  { word_id: 1, correct_count: 5, wrong_count: 2, last_reviewed_at: "..." },
  { word_id: 2, correct_count: 3, wrong_count: 1, last_reviewed_at: "..." }
]

// IndexedDB wrongAnswers 전체
[
  { word_id: 2, quiz_type: "spanish_to_korean", user_answer: "주스", correct_answer: "물", ... },
  { word_id: 5, quiz_type: "korean_to_spanish", user_answer: "그라시아", correct_answer: "감사합니다", ... }
]
```

### 회원가입 후 마이그레이션 API 호출

```javascript
async function handleMigrateData(newToken) {
  const progressData = await getAllProgress();       // IndexedDB에서 전체 조회
  const wrongAnswersData = await getWrongAnswers();  // IndexedDB에서 전체 조회
  
  const response = await fetch('/api/migrate-progress', {
    method: 'POST',
    headers: {
      'Authorization': 'Bearer ' + newToken,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      progress_data: progressData,
      wrong_answers: wrongAnswersData
    })
  });
  
  const result = await response.json();
  console.log(`${result.data.migrated_progress}개 진행상황 마이그레이션됨`);
  console.log(`${result.data.migrated_wrong_answers}개 오답 마이그레이션됨`);
}
```

### API 요청 본문 예시

```json
{
  "progress_data": [
    {
      "word_id": 1,
      "correct_count": 5,
      "wrong_count": 2,
      "last_reviewed_at": "2026-03-06T10:30:00Z"
    },
    {
      "word_id": 2,
      "correct_count": 3,
      "wrong_count": 1,
      "last_reviewed_at": "2026-03-05T14:20:00Z"
    }
  ],
  "wrong_answers": [
    {
      "word_id": 2,
      "quiz_type": "spanish_to_korean",
      "user_answer": "주스",
      "correct_answer": "물"
    },
    {
      "word_id": 5,
      "quiz_type": "korean_to_spanish",
      "user_answer": "그라시아",
      "correct_answer": "감사합니다"
    }
  ]
}
```

### Laravel이 처리

```php
// MigrationController.php
public function migrateProgress(Request $request)
{
    $data = $request->validate([
        'progress_data' => 'array',
        'wrong_answers' => 'array'
    ]);
    
    $userId = auth()->id();
    $migratedProgress = 0;
    $migratedWrongAnswers = 0;
    
    // 진행상황 마이그레이션
    foreach ($data['progress_data'] as $progress) {
        UserProgress::updateOrCreate(
            ['user_id' => $userId, 'word_id' => $progress['word_id']],
            $progress
        );
        $migratedProgress++;
    }
    
    // 오답 마이그레이션
    foreach ($data['wrong_answers'] as $answer) {
        WrongAnswer::create([
            'user_id' => $userId,
            ...$answer
        ]);
        $migratedWrongAnswers++;
    }
    
    return response()->json([
        'success' => true,
        'data' => [
            'migrated_progress' => $migratedProgress,
            'migrated_wrong_answers' => $migratedWrongAnswers
        ]
    ]);
}
```

### MySQL에 최종 저장

```sql
-- user_progress에 삽입/업데이트
INSERT INTO user_progress (user_id, word_id, correct_count, wrong_count, ...)
VALUES 
  (5, 1, 5, 2, ...),
  (5, 2, 3, 1, ...)
ON DUPLICATE KEY UPDATE
  correct_count = VALUES(correct_count),
  wrong_count = VALUES(wrong_count);

-- wrong_answers에 삽입
INSERT INTO wrong_answers (user_id, word_id, quiz_type, user_answer, correct_answer, ...)
VALUES
  (5, 2, 'spanish_to_korean', '주스', '물', ...),
  (5, 5, 'korean_to_spanish', '그라시아', '감사합니다', ...);
```

---

## 7️⃣ 검색 기능의 JSON 흐름

### Phase 1: 로컬 검색 (words.json 필터링)

```javascript
// 사용자가 "agua" 검색
const searchWords = (query) => {
  const q = query.toLowerCase();
  return wordsData.words.filter(word =>
    word.spanish.toLowerCase().includes(q) ||
    word.korean.includes(q)
  );
};

// 결과
[
  {
    id: 2,
    spanish: "agua",
    korean: "물",
    category_id: 2,
    example_spanish: "Un vaso de agua, por favor",
    example_korean: "물 한 잔 주세요"
  }
]
```

### Phase 2: API 검색

```
GET /api/words/search?q=agua
```

```json
{
  "success": true,
  "data": [
    {
      "id": 2,
      "spanish": "agua",
      "korean": "물",
      "category_id": 2,
      "category_name": "식당에서",
      "audio_url": "/audio/words/2.mp3",
      "match_type": "spanish",
      "match_position": "exact"
    }
  ]
}
```

---

## 8️⃣ 전체 API 응답 패턴 (Phase 2)

### 성공 응답
```json
{
  "success": true,
  "data": { ... }
}
```

### 에러 응답
```json
{
  "success": false,
  "message": "이메일이 이미 등록되어 있습니다",
  "errors": {
    "email": ["The email has already been taken."]
  }
}
```

### 인증 필요 에러
```json
{
  "success": false,
  "message": "Unauthenticated",
  "status": 401
}
```

### 페이징 응답
```json
{
  "success": true,
  "data": {
    "items": [...],
    "total": 100,
    "per_page": 20,
    "current_page": 1,
    "last_page": 5
  }
}
```

---

## 9️⃣ TypeScript 타입 정의 (선택사항)

```typescript
// types/index.ts

interface Word {
  id: number;
  spanish: string;
  korean: string;
  category_id: number;
  audio_url?: string | null;        // Phase 1: null, Phase 2: mp3 URL
  example_spanish?: string;
  example_korean?: string;
  difficulty: 'easy' | 'normal' | 'hard';
}

interface Category {
  id: number;
  name: string;
  emoji: string;
  slug: string;
  word_count?: number;
}

// Phase 1 Progress (단순 버전)
interface ProgressPhase1 {
  word_id: number;
  correct_count: number;
  wrong_count: number;
  last_reviewed_at: string;
}

// Phase 2 Progress (확장 버전)
interface Progress extends ProgressPhase1 {
  next_review_date: string;
  difficulty: 'easy' | 'normal' | 'hard';
  streak: number;
  is_learned: boolean;
}

interface WrongAnswer {
  id: number;
  word_id: number;
  quiz_type: 'spanish_to_korean' | 'korean_to_spanish';
  user_answer: string;
  correct_answer: string;
  created_at: string;
}

// 화면 표시용 (JOIN 후)
interface WrongAnswerDisplay extends WrongAnswer {
  spanish: string;
  korean: string;
  category_name: string;
  category_emoji: string;
}

interface User {
  id: number;
  name: string;
  email: string;
}

interface AuthToken {
  token: string;
  token_type: 'Bearer';
}

interface ApiResponse<T> {
  success: boolean;
  data?: T;
  message?: string;
}

// 내보내기/가져오기 데이터
interface ExportData {
  version: number;
  exported_at: string;
  progress: ProgressPhase1[];
  wrong_answers: WrongAnswer[];
}
```

---

## 🔟 요약: Phase별 데이터의 여정

### Phase 1 (서버 없음)

```
words.json 로드 (정적 데이터)
  ↓
React State에 단어/카테고리 설정
  ↓
같은 카테고리에서 4지선다 생성
  ↓
사용자 답변
  ↓
IndexedDB에 progress/wrongAnswer 저장
  ↓
[다른 기기로 이동 시]
  ↓
JSON 파일로 내보내기 → 다른 기기에서 가져오기
```

### Phase 2 (서버 추가)

```
사용자 답변
  ↓
IndexedDB에 즉시 저장 (오프라인 지원)
  ↓
로그인 상태면: API 요청 (JSON 전송)
  ↓
Laravel이 받아서 검증
  ↓
MySQL에 INSERT/UPDATE
  ↓
Spaced Repetition 계산 (Leitner System: 1→3→7→14→30일)
  ↓
API 응답 (JSON 반환)
  ↓
React에 표시
```

이 전체 흐름이 **Phase별로 확장 가능한 구조**로 설계되어 있습니다!
