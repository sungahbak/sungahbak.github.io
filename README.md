# sungashell

공부기록 블로그 — <https://sungahbak.github.io>

[Jekyll](https://jekyllrb.com/) + [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy) 테마. `main`에 push하면 GitHub Actions가 빌드해서 Pages로 배포한다.

## 글 쓰기

`_posts/YYYY-MM-DD-ascii-slug.md` 로 파일을 만든다. 파일명은 URL이 되므로 영문 슬러그를 쓴다.

```markdown
---
title: "제목에 증상을 그대로 적는다"
description: "한 줄 요약. 검색 결과와 목록에 노출된다."
date: 2026-11-02 23:50:00 +0900
categories: [개발환경]              # 기술 스택. _data/category_tree.yml 의 이름과 일치시킨다
tags: [트러블슈팅, mcp, claude-code] # 첫 태그는 글 형식(트러블슈팅/구성해설/측정기록), 나머지는 소문자
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
| `_includes/post-heatmap.html` | "최근 1년 동안 N회 기여" 격자. `site.posts`만 쓰므로 API 토큰이 필요 없다. |
| `_layouts/home.html` | 테마 원본(v7.6.0) + 글 목록 위에 위 격자 한 블록 추가. |
| `_includes/category-tree.html` | 오른쪽 패널의 "목차". 트리는 `_data/category_tree.yml`에서 선언한다. |
| `_data/category_tree.yml` | 목차 트리. **글이 0편인 항목은 링크 없이 흐리게** 나온다 = 쓸 거 목록. |
| `_layouts/default.html` | 테마 원본 + 패널에서 `category-tree`를 `update-list` 위에 끼운 한 줄. |
| `_layouts/archives.html` | 아카이브를 **달 단위**로. 달을 클릭하면 날짜 단위로 펼쳐진다 (`<details>`, JS 없음). |

### 카테고리 / 태그 규칙

- **카테고리 = 기술 스택.** `_data/category_tree.yml`에 선언한 이름만 쓴다. 안 쓰면 목차에 안 뜬다.
  2단계를 쓰는 건 `CI/CD`, `IaC`, `모니터링`뿐이다 (예: `[IaC, Terraform]`).
- **글 형식은 태그.** 첫 태그를 `트러블슈팅` / `구성해설` / `측정기록` 중 하나로 둔다.
- **프로젝트도 태그.** ORBIT 관련 글은 카테고리를 기술 스택으로 두고 `orbit` 태그를 붙인다.
  카테고리로 빼면 쿠버네티스 글을 찾는 사람이 ORBIT 밑에 숨은 글을 못 찾는다.
- 목차에 새 항목이 필요하면 `_data/category_tree.yml`에 한 줄 추가한다.

> 정보(about) 탭은 없앴다. 기여 격자는 홈으로 옮겼다.

## 로컬 미리보기 (선택)

Ruby가 필요하다. 없으면 push해서 Actions 빌드 결과로 확인해도 된다.

```bash
bundle install
bundle exec jekyll s -l
# http://127.0.0.1:4000
```

worktree 테스트
