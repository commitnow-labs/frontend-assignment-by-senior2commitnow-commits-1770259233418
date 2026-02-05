
# Wordle(5글자 단어 퍼즐) 게임

## ✏️ 과제의 개요

| 카테고리 | 난이도 | 제한시간 |
|----------|--------|----------|
| frontend | normal | 7d |

### 💻 과제에서 요구하는 개발언어

- react
- nextjs
- javascript
- typescript


## 📜 과제의 내용

> 과제 설명과 요구사항을 참고하여, 구현해 주세요.

### 👀 과제의 설명

# 과제 가이드

## 개요

Wordle(5글자 단어 퍼즐)을 구현합니다.

정답 위치/문자 포함 여부에 따른 **시각 피드백**, **가상 키보드**, **통계/공유**를 포함합니다.

특정 라이브러리 강제 없음(라우팅/상태/애니메이션 등 자유).

### 참고 링크

* 공식 Wordle: [https://www.nytimes.com/games/wordle](https://www.nytimes.com/games/wordle)

### 사용자 시나리오

* 사용자는 `/wordle`에서 5글자를 입력하고 **Enter**로 제출한다.
* 앱은 각 글자를 **정확 일치(hit) / 다른 위치 포함(present) / 미포함(miss)** 으로 판정해 보드와 키보드에 반영한다.
* **중복 문자**는 정답 내 등장 횟수만큼만 일치 처리한다.
* 정답을 맞히면 성공 모달이, 6회 모두 실패하면 실패 모달이 뜬다(정답 공개).
* 결과를 **이모지 그리드(🟩🟨⬛)** 텍스트로 클립보드 공유할 수 있다.
* 플레이 통계(플레이 수/승률/연속 성공 등)는 로컬에 저장·누적된다.
* (옵션) “하드 모드”·“색약 모드”·“일일 퍼즐/랜덤”을 설정에서 토글할 수 있다.

### 필요한 화면

* **게임 보드** `/wordle` 5×6 타일, 현재 행 입력, 가상 키보드(알파벳 + Enter/Backspace), 키 색 동기화.&#x20;
* **결과/통계 모달** 성공/실패 메시지, 시도 횟수, 시도 분포, 연속 성공, 공유 버튼.&#x20;
* **설정(선택)** `/wordle/settings` 또는 모달 하드 모드, 색약 모드(패턴/기호 보조), 일일 퍼즐 vs 랜덤.&#x20;

### 데이터 인터페이스 및 예시

```typescript
// types.ts
export type TileState = 'hit' | 'present' | 'miss';

export interface GuessLetter {
  char: string;      // 단일 대문자
  state: TileState;  // 판정 결과
}

export interface GuessResult {
  letters: GuessLetter[]; // 길이=5
}

export interface GameConfig {
  mode: 'daily' | 'random';
  startDate: string;      // YYYY-MM-DD (daily 기준일)
  maxTries: number;       // 기본 6
  wordLength: number;     // 기본 5
  shareEmoji: { hit: string; present: string; miss: string };
}

export interface GameStats {
  gamesPlayed: number;
  gamesWon: number;
  currentStreak: number;
  maxStreak: number;
  distribution: { [tries: number]: number }; // 예: {1:0,2:1,3:3,...}
}

export interface Dictionary {
  answers: string[];   // 대문자 5글자
  allowlist: string[]; // 제출 검증용(answers 포함)
}

```

```json
// config.json (최소 예시)
{
  "mode": "daily",
  "startDate": "2025-10-01",
  "maxTries": 6,
  "wordLength": 5,
  "shareEmoji": { "hit": "🟩", "present": "🟨", "miss": "⬛" }
}

```

```json
// answers_en.json (최소 예시: 8개)
[
  "ROUTE",
  "PLANT",
  "GUESS",
  "DEBUG",
  "CHAIR",
  "MOUSE",
  "CANDY",
  "BRAVE"
]

```

```json
// allowlist_en.json (최소 예시: answers 포함 30개 내외)
[
  "ROUTE","PLANT","GUESS","DEBUG","CHAIR","MOUSE","CANDY","BRAVE",
  "APPLE","GRAPE","LEMON","BERRY","PEACH","MANGO","ARISE","SMILE",
  "CRANE","TRACE","READY","ALERT","ALOFT","STONE","PRIDE","POINT",
  "SHARE","LEARN","CLOUD","DRIVE","SMART","PIXEL"
]

```

> 구현 시 answers는 정답 후보, allowlist는 제출 단어 검증(사전에 없는 단어 거절)에 사용하세요.`mode: "daily"`일 때는 `(오늘 - startDate)`의 일수로 정답 인덱스를 선택하는 방식을 권장(자유 구현).

​


### 🎯 과제의 요구사항

- 보드/입력: 5×6 보드 렌더링, 물리/가상 키보드 입력, 삭제/제출 동작
- 판정 로직: hit/present/miss 정확, 중복 문자 등장 횟수 처리 정확
- 허용 단어 검증: allowlist에 없는 단어 제출 방지(메시지/진동/애니메이션 등 피드백)
- 게임 종료: 정답/실패 처리, 정답 공개, 재시작 가능
- 키보드 색 동기화: hit > present > miss 우선순위 반영
- 공유 텍스트: 이모지 그리드 + X/6 형식 클립보드 복사
- 통계: 로컬 저장소 기반 플레이 수/승률/연속 성공/시도 분포 누적
- 접근성: 포커스 이동, aria-live로 결과 피드백, 색약 보조(심볼/패턴) 지원(선택)
