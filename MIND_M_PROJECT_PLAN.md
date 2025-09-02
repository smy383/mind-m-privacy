# 마인드엠(Mind-M) 프로젝트 계획서

## 프로젝트 개요
- **프로젝트명**: 마인드엠(Mind-M)
- **플랫폼**: 크로스 플랫폼 모바일 앱 (Android + iOS)
- **프레임워크**: Flutter
- **핵심 기능**: 감정 관리 및 개선, AI 분석, 구독 서비스, AdMob 광고 통합

## 앱 컨셉 (App Concept)
마인드엠(Mind-M)은 **사용자의 감정상태를 관리하고 기록하며 개선하는 AI 기반 감정 케어 앱**입니다.
- 📝 **일상 메모처럼 쉬운 감정 기록**
- 🤖 **AI가 자동 분석하여 인사이트 제공**
- 📊 **데이터 시각화 및 개선 방향 자동 제시**
- 🎯 **개인화된 목표 설정 및 추천**

---

## 1. 플랫폼 사양 (Platform Specifications)

### 🔧 개발 환경
- **Flutter**: 3.35 (최신 버전)
- **Dart**: 3.8+
- **개발 도구**: 
  - Android Studio (최신)
  - Xcode 15+ (iOS 개발용)
  - VS Code (선택사항)

### 📱 Android 플랫폼
- **minSdkVersion**: 26 (Android 8.0 API Level 26)
- **targetSdkVersion**: 36 (Android 16 API Level 36) ⚡ 업데이트됨
- **compileSdkVersion**: 36 ⚡ 업데이트됨
- **Android Gradle Plugin**: 8.5+
- **NDK 버전**: 27.0.12077973
- **Kotlin 버전**: 2.1.0 ⚡ 업데이트됨
- **Java 버전**: 17 (필수) ⚡ 추가됨

#### 특별 요구사항
- ✅ **16KB 메모리 페이지 지원** (2025년 11월 1일부터 Google Play 필수)
- ✅ 64비트 아키텍처 지원 (arm64-v8a)
- ✅ **Core Library Desugaring** 활성화 (Java 8+ 기능 지원)
- ✅ Google Play 정책 준수

### 🍎 iOS 플랫폼  
- **iOS Deployment Target**: 14.0+
- **Xcode**: 15+ (최신 권장)
- **Swift**: 5.9+
- **아키텍처**: arm64 (iPhone 5s 이상)
- **App Store 정책**: 최신 가이드라인 준수

### 🚀 성능 최적화 예상 효과
- **앱 실행 속도**: 3-30% 향상
- **배터리 효율성**: 4.5% 개선  
- **카메라 실행**: 4.5-6.6% 빠른 시작
- **전반적 성능**: 16KB 페이지 크기로 인한 메모리 최적화

### 📦 핵심 패키지 버전
- `flutter/flutter`: 3.35
- `in_app_purchase`: ^3.2.0+ (구독 기능)
- `google_mobile_ads`: ^5.1.0+ (AdMob 광고)

---

## 설치 과정 및 해결된 이슈들 🔧

### ✅ Flutter 설치 성공 (2025-08-30)
**최종 상태**: Flutter 3.35.2, Android 빌드 및 실행 완료

### 🚨 해결된 주요 문제들

#### 1️⃣ Java 버전 호환성 문제
- **문제**: `Android Gradle plugin requires Java 17 to run. You are currently using Java 11`
- **원인**: Android Gradle Plugin 8.5.0이 Java 17 필수 요구
- **해결책**:
  ```bash
  brew install openjdk@17
  ```
- **설정**: `android/gradle.properties`에 추가
  ```properties
  org.gradle.java.home=/opt/homebrew/opt/openjdk@17
  ```

#### 2️⃣ Android SDK 버전 불일치
- **문제**: 최신 Flutter 플러그인들이 Android SDK 36 요구
- **영향 플러그인**: flutter_local_notifications, in_app_purchase_android 등 7개
- **해결책**: `android/app/build.gradle` 업데이트
  ```gradle
  compileSdk 36
  targetSdkVersion 36
  ```

#### 3️⃣ Kotlin 버전 호환성
- **문제**: speech_to_text 플러그인이 Kotlin 2.1.0과 호환되지 않음
- **해결책**: 
  - Kotlin 버전을 2.1.0으로 통일
  - 호환되지 않는 플러그인 임시 제거
  ```yaml
  # speech_to_text: ^6.6.0  # 임시 비활성화
  ```

#### 4️⃣ Core Library Desugaring 누락
- **문제**: `flutter_local_notifications requires core library desugaring to be enabled`
- **해결책**: `android/app/build.gradle`에 추가
  ```gradle
  compileOptions {
      coreLibraryDesugaringEnabled true
  }
  dependencies {
      coreLibraryDesugaring 'com.android.tools:desugar_jdk_libs:2.0.4'
  }
  ```

#### 5️⃣ Android 리소스 파일 누락
- **문제**: `backup_rules.xml`, `data_extraction_rules.xml` 파일 없음
- **해결책**: XML 파일 생성
  - `android/app/src/main/res/xml/backup_rules.xml`
  - `android/app/src/main/res/xml/data_extraction_rules.xml`

#### 6️⃣ MainActivity 패키지 구조 불일치
- **문제**: `ClassNotFoundException: com.mindm.app.MainActivity`
- **원인**: MainActivity가 잘못된 패키지 경로에 위치
- **해결책**: 
  - 파일 이동: `com/mindm/mind_m/` → `com/mindm/app/`
  - 패키지명 수정: `package com.mindm.app`

#### 7️⃣ deprecated 옵션 제거
- **문제**: `android.bundle.enableUncompressedNativeLibs` 옵션이 deprecated
- **해결책**: gradle.properties에서 해당 옵션 제거

### 🎯 최종 설정값
```properties
# gradle.properties
org.gradle.java.home=/opt/homebrew/opt/openjdk@17
android.enableR8.fullMode=true
org.gradle.daemon=true
org.gradle.parallel=true
org.gradle.caching=true
```

```gradle
# build.gradle
ext.kotlin_version = '2.1.0'
compileSdk 36
targetSdkVersion 36
coreLibraryDesugaringEnabled true
```

## 호환성 검증 결과 ✅
- [x] 크로스 플랫폼 지원 (Android + iOS)
- [x] 구독 기능 완벽 지원
- [x] AdMob 광고 통합 가능
- [x] 16KB 메모리 페이지 지원
- [x] 2025년 Google Play 정책 준수
- [x] iOS App Store 정책 준수
- [x] **Android 빌드 및 실행 성공** 🚀

---

## 2. 핵심 기능 정의 (Core Features)

### 🥇 1순위 핵심 기능

#### 📝 쉬운 사용자 기록 (Easy User Logging)
- **일상 메모 스타일**: 카카오톡 메시지 쓰듯 간편하게
- **다양한 입력 방식**: 
  - 텍스트 입력 (기본)
  - 음성 메모 → 텍스트 변환 (STT)
  - 감정 이모티콘 빠른 선택
- **즉시 저장**: 네트워크 없어도 로컬 저장
- **시간 자동 기록**: 언제 작성했는지 자동 태깅

#### 🤖 AI 자동 분석 (AI Auto Analysis)
- **감정 상태 파악**: 텍스트에서 감정 추출 및 분류
- **패턴 인식**: 시간/요일별 감정 변화 패턴 학습
- **키워드 추출**: 감정에 영향을 주는 주요 요소 식별
- **개선점 자동 도출**: 데이터 기반 맞춤 조언 생성
- **목표 자동 제시**: 개인 상황에 맞는 현실적 목표 설정

#### 🔔 스마트 감정 체크 알림 (Smart Check-in Notifications)
- **개인화된 알림**: 사용자 패턴 학습하여 최적 시간 제안
- **부드러운 접근**: 강요하지 않는 친근한 메시지
- **상황 인식**: 연속으로 기록 누락시 다른 방식으로 접근
- **성취 격려**: 꾸준히 기록할 때 긍정적 피드백

### 🔄 AI 자동화 영역
1. **데이터 시각화**: 그래프, 차트 자동 생성
2. **트렌드 분석**: 주간/월간 감정 변화 요약
3. **개선 로드맵**: 단계별 목표 및 실행 계획 제시
4. **위험 감지**: 부정적 패턴 지속시 조기 경고
5. **긍정 강화**: 좋은 변화 감지시 축하 및 격려

### 🎯 사용자 경험 우선순위
1. **기록의 편리함** - 3초 안에 기록 완료
2. **AI의 정확성** - 개인 특성 반영한 분석
3. **자동화** - 사용자 개입 최소화하여 인사이트 제공

---

## 3. 기술 아키텍처 (Technical Architecture)

### 🏗️ 온디바이스 우선 아키텍처 (On-Device First)
**핵심 철학**: 사용자 데이터를 최대한 디바이스 내에서 처리하여 **프라이버시와 속도** 최적화

#### 📱 로컬 처리 영역 (Device-Local Processing)
- **데이터 저장**: SQLite + 암호화 (사용자 기록, 분석 결과)
- **기본 분석**: 텍스트 전처리, 키워드 추출, 기본 통계
- **UI/UX**: 모든 화면 렌더링 및 상호작용
- **알림 시스템**: 로컬 알림 스케줄링
- **데이터 시각화**: 차트/그래프 생성
- **백업/복원**: 로컬 파일 시스템 활용

#### 🌐 서버 의존 영역 (Server-Required)
- **AI 분석만**: Gemini API를 통한 고급 감정 분석
- **프록시 서버**: Railway 기반 AI API 키 보안 처리

### 🤖 AI 서비스 구조

#### Gemini AI 활용
- **감정 분석**: 텍스트→감정상태 분류
- **패턴 인식**: 시계열 감정 데이터 분석  
- **조언 생성**: 개인화된 개선 방안 제시
- **목표 설정**: 현실적 목표 자동 생성

#### Railway 프록시 서버
- **API 키 보안**: 클라이언트에 노출되지 않도록 서버에서 관리
- **요청 최적화**: 배치 처리, 캐싱으로 비용 절약
- **다중 앱 지원**: 기존 구축된 인프라 활용
- **에러 처리**: 재시도 로직, 폴백 메커니즘

### 🔒 데이터 보안 전략
- **로컬 암호화**: AES-256으로 사용자 데이터 보호
- **전송 암호화**: HTTPS + TLS 1.3
- **최소 데이터 전송**: AI 분석에 필요한 텍스트만 전송
- **익명화**: 개인 식별 정보 제거 후 AI 처리

### ⚡ 성능 최적화
- **오프라인 모드**: 네트워크 없어도 기본 기능 동작
- **지연 로딩**: 필요한 데이터만 메모리에 로드
- **백그라운드 동기화**: AI 분석 결과 비동기 처리
- **캐싱**: 반복 요청 최소화

### 📦 핵심 패키지 추가
- `sqflite`: ^2.3.0+ (로컬 데이터베이스)
- `crypto`: ^3.0.3+ (데이터 암호화)
- `http`: ^1.1.0+ (API 통신)
- `shared_preferences`: ^2.2.2+ (설정 저장)
- `local_auth`: ^2.1.6+ (생체 인증)

---

## 4. 알림 전략 (Notification Strategy)

### 🔔 개인화된 알림 시스템

#### 사용자 설정 옵션
- **알림 간격**: 1시간 ~ 24시간 (1시간 단위로 선택)
- **알림 시간대**: 사용자가 원하는 활동 시간 설정
- **알림 강도**: 
  - 적극적 (매일 정시)
  - 보통 (요일별 차등)
  - 소극적 (느슨한 간격)

#### 📱 스마트 알림 메시지 생성
**최근 메모 기반 개인화된 메시지**

##### 메시지 구조
```
[따뜻한 한마디] + [기록 유도 문구]
```

##### 예시 메시지
- 최근 긍정적 기록: *"어제의 행복한 순간이 아직도 미소를 짓게 하네요 😊 오늘은 어떤 기분인가요?"*
- 최근 힘든 기록: *"힘든 시간도 지나간다는 걸 당신은 알고 있어요 💙 지금 이 순간을 남겨주세요"*
- 기록 없음: *"당신의 하루가 궁금해요 ✨ 지금 순간을 기록해주세요"*
- 성장 감지: *"조금씩 변화하는 모습이 보여요 🌱 오늘의 감정도 들려주세요"*

#### 🎯 알림 전략 세부사항

##### 로컬 알림 스케줄링
- **Flutter Local Notifications** 패키지 활용
- **정확한 시간**: 사용자 설정 시간에 정확히 전송
- **백그라운드 처리**: 앱이 꺼져도 알림 동작
- **배터리 최적화**: 시스템 알림을 활용하여 전력 효율성

##### 메시지 개인화 로직
1. **최근 7일 감정 기록** 분석
2. **감정 트렌드** 파악 (상승/하강/안정)
3. **사용자 선호 키워드** 학습 (로컬 처리)
4. **상황별 메시지** 자동 선택

##### 사용자 경험 플로우
```
알림 도착 → 사용자 터치 → 즉시 메모 작성 화면 → 3초 기록 → 완료
```

#### 📋 알림 설정 UI 구성
- **시간 선택**: 드롭다운 또는 시간 휠
- **간격 설정**: 슬라이더 (1~24시간)
- **활성 시간대**: 시작/종료 시간 설정
- **미리보기**: 설정한 알림 메시지 샘플 보기
- **테스트 알림**: 즉시 알림 받아보기 기능

#### 🔧 기술 구현 사항

##### 필요 패키지
- `flutter_local_notifications`: ^17.2.1+ (로컬 알림)
- `timezone`: ^0.9.2+ (시간대 처리)
- `permission_handler`: ^11.3.1+ (알림 권한)

##### 알림 권한 관리
- **초기 설정시**: 알림 권한 요청
- **거부시**: 앱 내 설정 안내
- **재요청**: 사용자가 원할 때 다시 활성화

##### 개인정보 보호
- **로컬 처리**: 메시지 생성이 디바이스에서 완료
- **데이터 최소화**: 최근 감정 상태만 활용
- **사용자 제어**: 언제든 알림 중단/수정 가능

#### 🎨 알림 디자인 가이드
- **아이콘**: 앱 아이콘 또는 감정별 이모지
- **색상**: 부드러운 파스텔 톤
- **소리**: 선택적, 사용자 설정
- **진동**: 가볍고 부드러운 패턴

---

## 5. 데이터 모델 설계 (Data Model Design)

### 🗄️ SQLite 로컬 데이터베이스 구조

#### 1. 사용자 감정 기록 테이블 (emotion_records)
```sql
CREATE TABLE emotion_records (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    content TEXT NOT NULL,                    -- 사용자가 입력한 메모 내용
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    mood_score INTEGER DEFAULT NULL,          -- 1-10 점수 (선택사항)
    input_type TEXT DEFAULT 'text',          -- 'text', 'voice', 'emoji'
    word_count INTEGER DEFAULT 0,            -- 텍스트 길이 (분석용)
    is_analyzed BOOLEAN DEFAULT FALSE,       -- AI 분석 완료 여부
    encrypted_content BLOB,                  -- 암호화된 원본 데이터
    device_timezone TEXT,                    -- 기록 시점의 시간대
    location_context TEXT DEFAULT NULL       -- 선택적 위치 정보 (일반적 장소만)
);
```

#### 2. AI 분석 결과 테이블 (ai_analysis)
```sql
CREATE TABLE ai_analysis (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    record_id INTEGER NOT NULL,              -- emotion_records FK
    sentiment_score REAL NOT NULL,           -- -1.0 ~ 1.0 (부정↔긍정)
    primary_emotion TEXT,                    -- 주요 감정 (joy, sadness, anger 등)
    secondary_emotions TEXT,                 -- JSON 배열 (보조 감정들)
    keywords TEXT,                           -- JSON 배열 (핵심 키워드)
    suggestions TEXT,                        -- AI 조언/제안사항
    confidence_level REAL DEFAULT 0.8,      -- 분석 신뢰도 0.0~1.0
    processed_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    gemini_response_raw TEXT,                -- 원본 AI 응답 (디버깅용)
    FOREIGN KEY (record_id) REFERENCES emotion_records (id)
);
```

#### 3. 감정 패턴 분석 테이블 (emotion_patterns)
```sql
CREATE TABLE emotion_patterns (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    pattern_type TEXT NOT NULL,              -- 'daily', 'weekly', 'monthly'
    period_start DATE NOT NULL,
    period_end DATE NOT NULL,
    average_sentiment REAL,                  -- 기간 평균 감정
    trend_direction TEXT,                    -- 'improving', 'declining', 'stable'
    peak_emotion TEXT,                       -- 가장 빈번한 감정
    trigger_keywords TEXT,                   -- JSON 배열 (감정 유발 키워드)
    pattern_strength REAL DEFAULT 0.5,      -- 패턴 강도 0.0~1.0
    insights TEXT,                           -- AI가 발견한 인사이트
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

#### 4. 사용자 설정 테이블 (user_settings)
```sql
CREATE TABLE user_settings (
    id INTEGER PRIMARY KEY,
    notification_enabled BOOLEAN DEFAULT TRUE,
    notification_interval INTEGER DEFAULT 8,  -- 시간 간격
    notification_start_time TEXT DEFAULT '09:00',
    notification_end_time TEXT DEFAULT '22:00',
    notification_intensity TEXT DEFAULT 'normal', -- 'low', 'normal', 'high'
    privacy_mode BOOLEAN DEFAULT TRUE,        -- 고급 암호화 사용여부
    backup_enabled BOOLEAN DEFAULT FALSE,    -- 클라우드 백업 설정
    theme_mode TEXT DEFAULT 'auto',          -- 'light', 'dark', 'auto'
    language TEXT DEFAULT 'ko',              -- 언어 설정
    first_setup_completed BOOLEAN DEFAULT FALSE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

#### 5. 알림 메시지 캐시 테이블 (notification_cache)
```sql
CREATE TABLE notification_cache (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    message_text TEXT NOT NULL,
    message_type TEXT NOT NULL,              -- 'positive', 'supportive', 'neutral'
    based_on_sentiment REAL,                -- 어떤 감정 상태에서 생성됐는지
    usage_count INTEGER DEFAULT 0,          -- 사용 횟수 (다양성 위해)
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    last_used_at DATETIME DEFAULT NULL
);
```

#### 6. 목표 및 성취 테이블 (goals_achievements)
```sql
CREATE TABLE goals_achievements (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    goal_type TEXT NOT NULL,                 -- 'daily_record', 'mood_improvement'
    goal_description TEXT NOT NULL,         -- AI가 제안한 목표 설명
    target_value REAL,                      -- 목표 수치 (예: 연속 기록 7일)
    current_value REAL DEFAULT 0,           -- 현재 진행도
    status TEXT DEFAULT 'active',           -- 'active', 'completed', 'paused'
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    completed_at DATETIME DEFAULT NULL,
    reward_message TEXT                     -- 달성시 축하 메시지
);
```

### 🔐 데이터 보안 및 최적화

#### 암호화 전략
- **민감 데이터**: `encrypted_content` 필드에 AES-256 암호화 저장
- **검색 가능**: `content` 필드는 로컬 검색용 (해시 처리)
- **키 관리**: Flutter Secure Storage로 암호화 키 보관

#### 인덱스 최적화
```sql
-- 날짜별 빠른 조회
CREATE INDEX idx_emotion_records_date ON emotion_records(created_at);
-- AI 분석 완료 여부로 필터링
CREATE INDEX idx_emotion_records_analyzed ON emotion_records(is_analyzed);
-- 감정 점수별 정렬
CREATE INDEX idx_ai_analysis_sentiment ON ai_analysis(sentiment_score);
-- 패턴 기간별 조회
CREATE INDEX idx_patterns_period ON emotion_patterns(period_start, period_end);
```

### 📊 데이터 플로우

#### 1. 사용자 기록 플로우
```
사용자 입력 → 로컬 암호화 저장 → 백그라운드 AI 요청 → 결과 저장 → UI 업데이트
```

#### 2. 패턴 분석 플로우
```
일정 기간 데이터 수집 → 로컬 통계 처리 → AI 패턴 분석 요청 → 인사이트 저장
```

#### 3. 알림 생성 플로우
```
최근 7일 데이터 조회 → 로컬 감정 트렌드 분석 → 메시지 템플릿 선택 → 개인화 적용
```

### 🚀 성능 최적화 전략

#### 데이터 정리 정책
- **오래된 원시 AI 응답**: 90일 후 자동 삭제
- **중복 알림 메시지**: 30개 초과시 오래된 것 삭제
- **완료된 목표**: 1년 후 아카이브 테이블로 이동

#### 배치 처리
- **AI 분석**: 최대 5개씩 묶어서 요청 (비용 절약)
- **패턴 분석**: 주 1회 백그라운드에서 자동 실행
- **데이터 백업**: 사용자가 설정한 경우에만 주기적 동기화

---

## 6. 수익 모델 (Revenue Model)

### 💰 간단하고 명확한 수익 구조

#### 무료 버전 (Free Tier)
- **모든 핵심 기능** 이용 가능
  - 감정 기록, AI 분석, 알림, 데이터 시각화
- **광고 표시**
  - **배너 광고**: 하단 고정 (메모 작성 방해 안 되는 위치)
  - **전면 광고**: 앱 시작 시, 분석 결과 확인 후 (자연스러운 타이밍)

#### 프리미엄 구독 ($2.00/월)
- **광고 완전 제거**
- **동일한 기능** (기능 차별화 없음)
- **심플한 가치 제안**: "광고 없는 평온한 감정 기록"

### 📊 AdMob 광고 전략
- **사용자 경험 최우선**: 감정 기록 흐름 방해 금지
- **적절한 타이밍**: 자연스러운 화면 전환 시점
- **광고 빈도**: 과도하지 않게 (1일 최대 5-7회)

---

## 7. 디자인 시스템 (Design System)

### 🎨 글래스모피즘 기반 UI

#### 핵심 디자인 철학
- **투명성과 부드러움**: 뿌연 투명화로 평온한 감정 표현
- **깊이감**: 레이어드 글래스 효과로 공간감 연출
- **미니멀리즘**: 감정에 집중할 수 있는 깔끔한 인터페이스

#### 글래스모피즘 컴포넌트
```dart
// Flutter 글래스모피즘 효과 예시
Container(
  decoration: BoxDecoration(
    gradient: LinearGradient(
      begin: Alignment.topLeft,
      end: Alignment.bottomRight,
      colors: [
        Colors.white.withOpacity(0.15),
        Colors.white.withOpacity(0.05),
      ],
    ),
    borderRadius: BorderRadius.circular(16),
    border: Border.all(
      color: Colors.white.withOpacity(0.2),
      width: 1.5,
    ),
  ),
  child: BackdropFilter(
    filter: ImageFilter.blur(sigmaX: 10, sigmaY: 10),
    child: // 내용
  ),
)
```

#### 컬러 팔레트
- **베이스**: 부드러운 그라데이션 배경 (하늘색 → 보라색)
- **글래스**: 반투명 흰색 (opacity: 0.1~0.3)
- **텍스트**: 고대비 다크그레이 (#2D3748)
- **액센트**: 감정별 파스텔 컬러
  - 긍정: 연한 민트그린 (#A7F3D0)
  - 중성: 연한 라벤더 (#DDD6FE)  
  - 부정: 연한 피치 (#FED7AA)

#### UI 컴포넌트 설계
- **메모 입력창**: 큰 글래스 카드, rounded corners
- **감정 차트**: 투명한 오버레이, 블러 배경
- **버튼**: 서브틀한 글래스 효과, hover 상태
- **알림 카드**: 플로팅 글래스 패널

### 📦 필요 Flutter 패키지
```yaml
dependencies:
  flutter_glassmorphism: ^3.0.0    # 글래스모피즘 효과
  google_mobile_ads: ^5.1.0        # AdMob 광고
  in_app_purchase: ^3.2.0          # 구독 결제
```

### 🎯 UX 원칙
- **감정 우선**: 디자인이 감정 표현을 방해하지 않음
- **직관적 네비게이션**: 3초 규칙 준수
- **접근성**: 색상 대비, 터치 영역 크기 최적화
- **일관성**: 모든 화면에 통일된 글래스모피즘 적용

---

*계획서 작성일: 2025-08-30*  
*다음 단계: 프로젝트 초기 설정 및 개발 시작*