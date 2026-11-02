# broccoli.log

트러블슈팅 기록 블로그 — <https://sungahbak.github.io>

[Jekyll](https://jekyllrb.com/) + [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) 테마. `main`에 push하면 GitHub Actions가 빌드해서 Pages로 배포한다.

## 글 쓰기

`_posts/YYYY-MM-DD-ascii-slug.md` 로 파일을 만든다. 파일명은 URL이 되므로 영문 슬러그를 쓴다.

```markdown
---
title: "제목에 증상을 그대로 적는다"
description: "한 줄 요약. 검색 결과와 목록에 노출된다."
date: 2026-11-02 23:50:00 +0900
categories: [트러블슈팅, 개발환경]   # 최대 2단계
tags: [mcp, claude-code]            # 소문자
error_signatures:                   # 에러 메시지 원문 (검색 색인용으로 쌓는 중)
  - "Needs authentication"
verified: 2026-11-02                # 마지막으로 재현을 확인한 날
---
```

### 글의 골격

```text
1. 문제 상황      증상과 로그 원문
2. 환경          OS/버전/확인 시점
3. 원인을 좁혀나간 과정
   3-1. 오진 ①: ...  / 왜 틀렸나
   3-2. 오진 ②: ...  / 왜 틀렸나
4. 왜 이렇게 되는가   개념으로 내려가는 부분
5. 직접 확인하는 법   재현 명령어
6. 해결
7. 재발 방지
8. 정리          설명 / 예측 / 파괴
```

## 커스터마이즈한 부분

| 파일 | 내용 |
| --- | --- |
| `_includes/post-heatmap.html` | 글 쓴 날을 1년치 격자로 보여준다. `site.posts`만 쓰므로 API 토큰이 필요 없다. `_tabs/about.md`에서 호출. |

## 로컬 미리보기 (선택)

Ruby가 필요하다. 없으면 push해서 Actions 빌드 결과로 확인해도 된다.

```bash
bundle install
bundle exec jekyll s -l
# http://127.0.0.1:4000
```
