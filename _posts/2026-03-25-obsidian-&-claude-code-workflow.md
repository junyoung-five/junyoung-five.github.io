---
title: "Obsidian + Claude Code로 GitHub Pages 운영하기"
date: 2026-03-25
categories:
  - Study
tags:
  - Obsidian
  - Claude Code
  - GitHub Pages
  - Workflow
---

GitHub Pages로 개인 페이지를 만들면서, 글 작성과 코드 작업을 효율적으로 분리할 수 있는 워크플로우를 정리한다.

핵심은 **Obsidian**과 **Claude Code**의 역할을 나누는 것이다.

## 전체 구조

```
Obsidian (글 작성/수정/미리보기)
        ↕ 같은 로컬 폴더
Claude Code (레이아웃/코드 작업/리서치)
        ↓ git push
GitHub Pages (자동 빌드 & 배포)
```

Obsidian과 Claude Code 모두 동일한 로컬 레포지토리 폴더에 접근한다. 도구만 다를 뿐, 작업 대상은 같은 마크다운 파일이다.

## Obsidian의 역할

### 글 작성 및 편집

Obsidian은 마크다운 에디터로서, 블로그 포스트를 작성하고 수정하는 데 사용한다.

- `_posts/` 폴더에서 직접 새 글 작성
- 실시간 마크다운 렌더링으로 작성 중인 글의 모습을 바로 확인
- 프론트매터(YAML) 편집도 Obsidian 안에서 처리

### 퍼블리시 전 미리보기

GitHub Pages에 push하기 전, Obsidian의 렌더링 뷰에서 글의 구조와 내용을 최종 점검한다. 오탈자, 이미지 누락, 마크다운 문법 오류 등을 사전에 잡을 수 있다.

### Git 연동

**Obsidian Git 플러그인**을 설치하면, Obsidian 안에서 바로 commit & push가 가능하다. 글 작성에만 집중하고 싶을 때 터미널을 오갈 필요가 없다.

## Claude Code의 역할

### 페이지 레이아웃 및 코드 작업

Jekyll 설정(`_config.yml`), 레이아웃 커스터마이징, 테마 수정 등 코드성 작업은 Claude Code에게 맡긴다.

- `_config.yml` 수정
- `_includes/`, `_pages/` 등 구조 변경
- CSS/HTML 커스터마이징
- 새로운 collection이나 카테고리 추가

### 리서치 및 초안 정리

공부한 내용을 포스트로 정리할 때, Claude Code를 활용하여 초안을 작성할 수 있다.

- 특정 주제에 대한 리서치 및 내용 정리
- 코드 예제 포함한 기술 포스트 초안 작성
- 기존 노트를 블로그 포스트 형식으로 변환

### 일괄 작업

여러 파일을 한 번에 수정해야 하는 경우(카테고리 변경, 태그 정리, 프론트매터 일괄 수정 등)에도 Claude Code가 효율적이다.

## 설정 방법

### 1. 로컬에 레포지토리 clone

```bash
cd ~/Documents/GitHub
git clone https://github.com/username/username.github.io.git
```

### 2. Obsidian에서 vault로 열기

Obsidian → "Open another vault" → clone한 폴더 선택

### 3. `.gitignore`에 Obsidian 설정 제외

```
.obsidian/
```

Obsidian의 설정 파일이 레포에 포함되지 않도록 반드시 추가한다.

### 4. 포스트 작성 규칙

파일명은 `YYYY-MM-DD-제목.md` 형식을 따르고, 상단에 프론트매터를 포함한다.

```yaml
---
title: "글 제목"
date: 2026-03-25
categories:
  - Study
tags:
  - 태그
---
```

## 주의사항

- Obsidian의 `[[wikilink]]` 문법은 Jekyll이 인식하지 못한다. 링크는 `[텍스트](URL)` 형식을 사용한다.
- 이미지는 `assets/images/`에 저장하고, 마크다운에서 상대 경로로 참조한다.
- Google Drive 등 클라우드 동기화 폴더에 Git 레포를 두면 `.git/` 충돌이 발생할 수 있으므로, 로컬 경로에 두는 것을 권장한다.

## 정리

| 작업 | 도구 |
|------|------|
| 글 작성/수정 | Obsidian |
| 미리보기/퇴고 | Obsidian |
| 레이아웃/코드 작업 | Claude Code |
| 리서치/초안 정리 | Claude Code |
| 일괄 수정 | Claude Code |
| Git push/배포 | Obsidian Git 플러그인 or 터미널 |

글은 Obsidian으로, 코드는 Claude Code로. 각 도구의 강점에 맞게 역할을 나누면 개인 페이지 운영이 훨씬 수월해진다.
