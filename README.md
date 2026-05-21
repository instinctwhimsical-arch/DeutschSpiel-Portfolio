# 🇩🇪 Deutsch-Spiel: Gamified German Learning Platform

![Badge: React](https://img.shields.io/badge/Frontend-React_18-blue?logo=react)
![Badge: Spring Boot](https://img.shields.io/badge/Backend-Spring_Boot_3-6DB33F?logo=spring)
![Badge: Kotlin](https://img.shields.io/badge/Language-Kotlin-7F52FF?logo=kotlin)
![Badge: PostgreSQL](https://img.shields.io/badge/Database-PostgreSQL_15-4169E1?logo=postgresql)
![Badge: Supabase](https://img.shields.io/badge/BaaS-Supabase-3ECF8E?logo=supabase)

**Deutsch-Spiel**은 플래시카드를 이용한 지루한 기존의 단어 암기 방식을 탈피하여, 역동적인 아케이드 스타일 게임, 전략적 시간 제한, 그리고 강력한 "보스 레이드" 시스템을 도입한 100% 무료/광고 없는 독일어 학습 플랫폼입니다.

**Deutsch-Spiel** is a 100% free, ad-free, fully gamified German language learning web application. It is designed to replace traditional, boring flashcards with dynamic arcade-style games, strategic time limits, and a robust "Boss Raid" progression system.

---

## 🌐 Live Deployments (실배포 주소)
- **Frontend (Vercel):** https://deutsch-spiel-app.vercel.app
- **Backend API (Render):** https://deutschspielapp.onrender.com (Note: Cold start may take up to 50 seconds on the free tier.)

---

## 🚀 Architectural Highlights & Engineering Decisions

### 1. 🎧 Zero-Cost TTS Architecture: Multi-Tier Caching & Circuit Breakers
To prevent exorbitant API billing from external TTS (Text-to-Speech) services and block malicious bot traffic, a highly resilient **Two-Tier Caching System** was implemented.
- **L1 Cache (Frontend - React):** Utilizes `Map` and `sessionStorage` for instant audio playback. Pre-fetches audio asynchronously during the 3-second game countdown (Non-blocking UX).
- **L2 Cache & Circuit Breaker (Backend - Spring Boot):** Hits the Supabase `audio_cache` table using MD5 text hashing before calling the external Google TTS API.
  - Implemented **Rate Limiting** (max 30 requests/min per IP) via `ConcurrentHashMap` to prevent self-DDoS.
  - Implemented a **Daily Circuit Breaker** (`AtomicInteger` + `@Scheduled`) to cap external API calls, ensuring server costs remain strictly at $0.

### 2. 🛡️ Server-Driven Grading & Anti-Cheating System
Client-side grading exposes correct answers in the browser's Network tab. To ensure absolute data integrity:
- **Backend-Driven Evaluation:** The frontend only receives obfuscated template IDs (e.g., `Ich ___ dich`). User inputs are submitted via a generic payload to the backend (`POST /api/quiz/secure-grade`).
- **Session Kickout & Visibility API:** Validates JWT and `X-Session-Id` strictly to prevent multi-device abusing. Any tab-switching during gameplay triggers penalties via the Page Visibility API.

### 3. 🗄️ Database Evolution: NoSQL (Firestore) to RDBMS (PostgreSQL)
Migrated the entire database from Firebase Firestore to Supabase (PostgreSQL) seamlessly within a single day to handle complex linguistic relationships.
- **Complex Modeling:** Enabled structured mapping for German grammar nuances (e.g., `prepositions`, `cases`, `conjugations`, and `plurality`).
- **Data Integrity:** Applied `NULLS NOT DISTINCT` composite unique indexes in PostgreSQL to prevent duplicate word signatures.
- **Audit Logging & Soft Deletes:** Implemented soft deletion (`use_yn`) and an Audit system recording hard deletes into a `deleted_words_log` table with strict UTC timestamp standardization.

### 4. ⚡ Intelligent Interactions & Multi-Blank System
Developed a highly interactive UI for complex German sentence building.
- **Process of Elimination:** Dynamically manages the `isUsed` state of word chips by calculating the total occurrences of duplicate correct answers within a page context.
- **Smart Focus Auto-jump:** Automatically calculates the next empty `globalIdx_blankIdx` composite key, instantly moving the cursor to the next blank upon user input for a seamless flow.
- **Grading Leniency:** Strips unnecessary punctuations (`.`, `?`, `!`) and normalizes whitespaces in the backend to prevent frustrating "typo" failures.

---

## 🎮 Core Gameplay & Gamification

- **Dynamic Timers:** Replaced static timers with dynamically calculated time limits (`Word Count * 4 seconds`). UI triggers visual tension (red blink/pulse) when only 20% of the time remains.
- **Smart Random Conjugation:** The testing algorithm assigns a 60% higher probability to tricky conjugations (`du`, `er/sie/es`) rather than easy ones (`ich`, `wir`), forcing users to actively learn irregular verb changes.
- **3-Strike Hardcore Review System:** Incorrect answers are stored in `wrong_answers`. Words are only removed after the user answers them correctly 3 consecutive times in specific review sessions. One mistake resets the streak to 0.

---

## 🗺️ Future Roadmap: Upcoming Arcade Games & Stages (업데이트 예정 기능)

독일어를 무의식적으로 마스터할 수 있도록 설계된 에듀테인먼트 아케이드 스페셜 게임 및 시스템 파이프라인 목록입니다.
The following feature sets and special arcade game modes are slated for incremental updates to achieve subconscious grammar mastery:

### 1. 🧭 Core Features (핵심 기능)
- **Interactive Tutorial System:** A step-by-step onboarding guide for first-time users to grasp game controls and rules instantly.
- **Wrong Answer Review Arcade:** A dedicated review module where users must repeatedly survive on previously failed questions to drill weak points.
- **CEFR Level Progression (A1.1 to B1.2 Boss Raids):** Comprehensive chapter graduation exams structured as "Boss Fights" at the end of each CEFR curriculum stage.

### 2. 🕹️ 5 Specialized Grammar Arcade Game Modes (문법 특화 아케이드 게임 5종)
Dynamic, repetitive training mini-games covering 13 critical linguistic variations:
1. **The Pronoun Clash (`Wer / Wem / Wen`):** Fast-paced interactive repetition game to automatically distinguish and master German nominative, dative, and accusative pronouns.
2. **Article Matrix (`Der / Die / Das`):** Grid-based reflex game to drill noun genders and case mutations continuously through real-time context and sentence examples.
3. **Auxiliary Rapid-Fire (`Sein vs Haben`):** Split-second decision arcade to build reflex muscle memory for past-tense helping verbs.
4. **Possessive Matrix:** A rapid grid completion game for possessive pronoun tables.
5. **Personal Pronoun Blitz:** High-frequency drill arcade matching personal pronouns to their respective cases.
6. **Adjective Ending Arcade:** Real-time tactical gameplay focused on modifying adjective suffixes across weak, strong, and mixed declensions.
7. **Verb Conjugation Rush:** Speed-running regular, irregular, strong, and past-tense verb conjugations.
8. **Subconscious Interrogatives:** Reverse-engineering prompt sentences (Korean/English to German) using advanced interrogatives (`Wie viel...?`, `Wie oft...?`, `Wann?`) under tight time constraints.
9. **Prepositional Matchmaking (Verb + Preposition):** Sentence-building puzzles designed to fuse verbs seamlessly with their fixed dependent prepositions.
10. **Modal Verb Shift:** Interactive repetition layout tracking the erratic vowel changes of modal verbs.
11. **Geographical Transit (`nach / in / zu`):** Reflex game targeting regional prepositions (e.g., `nach Düsseldorf`, `in die Schweiz`) to eradicate spatial grammar errors.
12. **The Two-Way Preposition Pit (`Wechselpräpositionen`):** Graphical puzzle arcade distinguishing static locations (`Dativ`) from directional movements (`Akkusativ`) alongside strict accusative-only prepositions (`durch`, `für`, `gegen`, `ohne`, `um`).
13. **Hardcore Review Loop:** Arcade mode pulling heavily from cumulative mistake logs.

---

## 🛠️ Tech Stack

### Frontend
- **React.js 18** (Vite)
- **Tailwind CSS & Framer Motion** (Glassmorphism, 3D Animations, Code-driven UX)
- **Context API** (State Management, Multilingual/I18n Context)

### Backend
- **Kotlin 1.9 & Spring Boot 3.2**
- **Spring Data JPA & Spring Security**
- **BFF (Backend-For-Frontend) Pattern:** Aggregating lobby logic, scores, and user data into single, efficient API calls.

### Database & DevOps
- **Supabase (PostgreSQL 15)**
- **GitHub Actions & Vercel** (CI/CD)
- **JWT & Environment Parity**

---

## 💻 Local Development Setup

1. **Clone the repository:**
   ```bash
   git clone [https://github.com/instinctwhimsical-arch/DeutschSpielApp.git](https://github.com/instinctwhimsical-arch/DeutschSpielApp.git)
   cd deutsch-spiel
