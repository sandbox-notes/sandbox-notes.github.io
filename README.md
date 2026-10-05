# Sandbox Notes (블로그 뼈대)

Hugo 기반 한국어/영어 이중 언어 블로그. 테마는 `layouts/`와 `static/css/style.css`에 직접 들어 있어 외부 테마 다운로드가 필요 없다.

## 구조

```
hugo.toml                  # 사이트 설정 (ko 기본, en 추가)
content/ko/posts/*.md      # 한국어 글
content/en/posts/*.md      # 영어 글 (같은 파일명을 쓰면 언어 전환 링크로 연결됨)
layouts/                   # 최소 테마 (라이트/다크 자동)
static/css/style.css
.github/workflows/pages.yml  # GitHub Pages 자동 배포
```

## 처음 한 번

1. Hugo(extended 아님도 가능)를 설치한다. 예: `winget install Hugo.Hugo.Extended`
2. 이 폴더에서 로컬 미리보기: `hugo server -D`  (`-D`는 초안 포함)
3. 브라우저에서 `http://localhost:1313/ko/` 확인.

## 글 올리기 전 체크

- [ ] 글 머리의 `draft: true`를 `false`로 바꿨는가 (초안은 배포되지 않는다)
- [ ] 샘플 파일, 다운로드 링크가 없는가 (해시와 공개 분석 링크만)
- [ ] IOC는 디펭했는가 (`hxxp`, `[.]onion`)
- [ ] 호스트명, 사용자명, 경로, 이메일이 스크린샷/영상/pcap에 없는가
- [ ] 확인한 것과 공개 자료 기반 추정을 구분해서 썼는가
- [ ] 영어 글 번역을 직접 검토했는가
- [ ] 회사 업무와 겹치면 규정을 확인했는가

## 배포 (GitHub Pages)

1. GitHub에 새 저장소를 만들고 이 폴더를 push한다.
2. 저장소 Settings > Pages > Source 를 `GitHub Actions`로 바꾼다.
3. `hugo.toml`의 `baseURL`을 실제 주소로 바꾼다. (워크플로가 `--baseURL`로 덮어쓰므로 필수는 아님)

## 새 글 만들기

같은 파일명으로 두 언어를 만든다.

```
content/ko/posts/<슬러그>.md
content/en/posts/<슬러그>.md
```
