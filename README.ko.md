# Skill & Command Reviewer

[English](README.md) | 한국어

[Claude Code](https://code.claude.com)를 위한 고품질 리뷰어 스킬로, Anthropic의 공식 모범 사례를 기준으로 스킬과 커스텀 명령어를 평가하고 개선합니다.

## 포함된 내용

### 📋 [skill-reviewer](skills/skill-reviewer)

Claude Code 스킬의 **구조적 정확성** (F1-F12)과 **원칙 품질** (P1-P12)을 평가합니다.

**사용 시기:**
- 새로운 스킬 생성 시
- 퍼블리싱 전 기존 스킬 검토 시
- 스킬 저장소에 개선 사항 기여 시
- 설치 전 서드파티 스킬 검증 시

**모드:**
- 모드 1: 자체 검토 — 자신의 스킬 검증
- 모드 2: 외부 검토 — 다른 사람의 스킬 평가
- 모드 3: 자동 PR — 포크, 개선 및 PR 제출
- 모드 4: 생성 — 모범 사례를 따르는 새 스킬 생성

### ⚙️ [command-reviewer](skills/command-reviewer)

Claude Code 커스텀 슬래시 명령어의 **구조적 정확성** (CF1-CF10)과 **원칙 품질** (CP1-CP12)을 평가합니다.

**사용 시기:**
- 커스텀 슬래시 명령어 생성 시
- `.claude/commands/*.md` 파일 검토 시
- 명령어 vs 스킬 형식 결정 시
- 명령어 보안 및 안정성 개선 시

**모드:**
- 모드 1: 자체 검토 — 자신의 명령어 검증
- 모드 2: 외부 검토 — 다른 사람의 명령어 평가
- 모드 3: 생성 — 모범 사례를 따르는 새 명령어 생성
- 모드 4: 자동 PR — 명령어 저장소에 개선 사항 기여

## 설치

### 옵션 1: 두 스킬 모두 설치

```bash
# 이 저장소 클론
git clone https://github.com/nyjin/skill-reviewer.git

# Claude 스킬 디렉토리로 복사
cp -r skill-reviewer/skills/* ~/.claude/skills/
```

### 옵션 2: 개별 스킬 설치

```bash
# skill-reviewer만 설치
cp -r skill-reviewer/skills/skill-reviewer ~/.claude/skills/

# 또는 command-reviewer만 설치
cp -r skill-reviewer/skills/command-reviewer ~/.claude/skills/
```

## 사용법

### 스킬 검토

```
# 자신의 스킬 자체 검토
/skill-reviewer

# 특정 스킬 검토
/skill-reviewer Review the skill at ~/.claude/skills/my-skill

# 새 스킬 생성
/skill-reviewer Create a skill for PDF processing
```

### 명령어 검토

```
# 자신의 명령어 자체 검토
/command-reviewer

# 특정 명령어 검토
/command-reviewer Review the command at .claude/commands/deploy.md

# 새 명령어 생성
/command-reviewer Create a command for git commits
```

## 품질 검사

### 스킬 (F1-F12 + P1-P12)

**형식 (구조적):**
- F1-F4: 프론트매터 (이름, 설명, 트리거)
- F5-F11: 본문 구조 (줄 수, 경로, 참조)
- F12: 스크립트 오류 처리

**원칙 (품질):**
- P1: WHY 설명
- P2: 토큰 효율성
- P3: 자유도 조정
- P4: 일관된 용어
- P5: 구체적 예제
- P6: 피드백 루프
- P7: 일반화
- P8-P12: 메타데이터 품질

### 명령어 (CF1-CF10 + CP1-CP12)

**형식 (구조적):**
- CF1-CF6: 프론트매터 (파일명, YAML, 설명, allowed-tools)
- CF7-CF10: 보안 및 이식성

**원칙 (품질):**
- CP1: WHY 설명
- CP2: 토큰 효율성
- CP3: 자유도 조정
- CP4: 인라인 컨텍스트 (`!` 및 `@`)
- CP5: 출력 형식
- CP6: 피드백 루프
- CP7: 일반화
- CP8: 오류 처리
- CP9: 인자 설계
- CP10: 보안 (최소 권한)
- CP11-CP12: 존재 정당성

## 스코프 프로필

두 리뷰어 모두 스코프에 따라 검사를 조정합니다:

- 📁 **프로젝트**: 단일 저장소, 과적합 허용
- 🏠 **개인**: 개인 사용, 일반화 필수
- 👥 **팀**: 조직 내 공유, + 문서화
- 🌐 **공개**: 오픈소스, 모든 검사 필수

## 기여

개선 사항을 환영합니다! 두 리뷰어 스킬 모두 자체 모범 사례를 따릅니다. 기여 시:

1. 변경 사항에 대해 리뷰어 실행
2. F/P 또는 CF/CP 검사 준수
3. `references/pull-request-guide.md`의 PR 템플릿 사용

## 라이선스

MIT License - [LICENSE](LICENSE) 참조

## 참고 자료

- [Anthropic Agent Skills Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Claude Code Documentation](https://code.claude.com/docs)
