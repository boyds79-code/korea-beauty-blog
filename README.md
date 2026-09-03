# K-Beauty, Explained — 자동 초안 + 사람 검수 블로그

영어권 독자를 대상으로 한 한국 스킨케어/뷰티 블로그입니다. Astro로 만든 정적 사이트이고,
매일 GitHub Actions가 Claude API로 글 초안을 자동 생성해 Pull Request를 올리면,
**사람이 검토하고 PR을 머지해야만** 실제로 사이트에 발행되는 구조입니다.
(초안 자동 생성 ≠ 자동 발행 — 발행은 항상 PR 머지라는 사람의 행동이 트리거합니다.)

이 프로젝트는 `korea-blog`의 구조를 그대로 재사용해서 만든 4번째 블로그입니다
(`korea-blog`, `korea-recipes-blog`, `korea-entertainment-blog`와 동일한 파이프라인/워크플로우).

## 어떻게 작동하나요

1. 매일 정해진 시간에 GitHub Actions(`daily-post.yml`)가 실행됩니다.
2. `topics/queue.yaml`에 남은 주제가 있으면 맨 위 항목을 하나 꺼내 씁니다. (초기 20개용)
3. 큐가 비면 자동으로 "트렌드 모드"로 전환되어, 구글 뉴스에서 최근 K-beauty/한국 스킨케어
   관련 헤드라인을 하나 찾아 그걸 소재로 삼습니다. 다만 이 블로그는 성격상 대부분의 글이
   에버그린 설명글(성분, 루틴, 개념 설명)이라 트렌드 모드는 큐가 소진됐을 때의 안전장치에
   가깝습니다 — 언제든 `topics/queue.yaml`에 새 주제를 추가하는 걸 권장합니다.
4. Claude API를 호출해 SEO에 맞춘 완성된 글 초안(`src/content/blog/*.md`)을 생성합니다.
5. 생성된 파일을 새 브랜치에 커밋하고, **Pull Request를 엽니다.** 이 시점에는 아무것도 사이트에 반영되지 않습니다.
6. 당신이 PR을 열어 내용을 읽고, 필요하면 직접 수정한 뒤, **머지(merge)** 합니다.
7. `main` 브랜치에 머지되는 순간 Vercel이 자동으로 빌드/배포해서 글이 실제로 공개됩니다.

## 이 블로그만의 특징 (다른 세 블로그와 다른 점)

- **특정 브랜드/제품명을 언급하지 않습니다.** 시스템 프롬프트(`scripts/lib/anthropic.mjs`)에서
  성분 종류·루틴 개념 위주로 쓰도록 명시적으로 지시해뒀습니다 — 특정 제품을 추천하는 것처럼
  보이면 애드센스 심사나 신뢰도에 안 좋고, 제품 단종/가격 변동으로 콘텐츠가 금방 낡습니다.
- **의학적 조언이 아니라는 안내가 자연스럽게 들어가도록** 프롬프트에 지시해뒀습니다. 그래도
  검수 시 과장되거나 근거 없는 효능 주장이 없는지 특히 신경 써서 읽어주세요.
- 큐에 있는 초기 20개 주제는 모두 성분/루틴 설명 위주의 에버그린 글감입니다.

## 시작하기 (처음 한 번만)

### 1. 로컬에서 확인
```bash
npm install
npm run dev        # http://localhost:4321 에서 확인
npm run build      # 정적 빌드가 에러 없이 되는지 확인
```

### 2. GitHub 저장소 만들기
새 GitHub 저장소를 만들고 이 프로젝트 전체를 push하세요.
```bash
git init
git add -A
git commit -m "Initial commit"
git branch -M main
git remote add origin <당신의-저장소-URL>
git push -u origin main
```

### 3. Anthropic API 키 발급 및 등록
1. https://console.anthropic.com 에서 API 키를 발급받으세요 (이미 발급받은 키를 재사용해도 됩니다).
2. GitHub 저장소 → **Settings → Secrets and variables → Actions → New repository secret**
   - Name: `ANTHROPIC_API_KEY`
   - Value: 발급받은 키
3. (선택) 모델을 바꾸고 싶으면 같은 화면의 **Variables** 탭에서 `CLAUDE_MODEL` 변수를 추가하세요.

### 3-1. (추천) Gemini API 키로 이미지까지 완전 자동화
1. https://aistudio.google.com/apikey 에서 무료로 API 키를 발급받으세요 (기존 키 재사용 가능).
2. 같은 저장소 **Settings → Secrets and variables → Actions → New repository secret**
   - Name: `GEMINI_API_KEY`
   - Value: 발급받은 키
3. 이걸 등록해두면 PR이 열릴 때 이미지까지 이미 채워져서 옵니다.

### 4. GitHub Actions에 PR 생성 권한 확인
저장소 **Settings → Actions → General → Workflow permissions**에서
"Read and write permissions" + "Allow GitHub Actions to create and approve pull requests"가
켜져 있는지 확인하세요.

또한 **Settings → Secrets and variables → Actions → New repository secret**에
`PAT_TOKEN`을 등록해야 합니다 — 다른 세 블로그에서 이미 만든 Personal Access Token을
그대로 재사용할 수 있습니다 (workflow 권한이 있는 토큰이면 됩니다).

### 5. Vercel 연결
1. https://vercel.com 에서 GitHub 저장소를 import 하세요.
2. Framework Preset은 자동으로 Astro가 잡힙니다. 별다른 설정 없이 Deploy를 누르면 됩니다.
3. 이후로는 `main`에 push(=PR 머지)될 때마다 자동 재배포됩니다.
4. 도메인은 `beauty.hey-driver.com`을 연결하세요 (Route 53에 다른 서브도메인들과 같은 방식으로
   CNAME 레코드를 추가하면 됩니다).

### 6. 첫 실행 테스트
저장소의 **Actions 탭 → Daily draft post → Run workflow** 를 눌러 수동으로 한 번 실행해보세요.

## 애드센스 준비

이 블로그는 `hey-driver.com` 루트 도메인 하위의 서브도메인이라, 루트 도메인이 이미
애드센스에 등록/승인된 상태라면 **별도 신청 없이 자동으로 광고가 활성화됩니다** (2023년 3월
이후 애드센스의 도메인 단위 사이트 관리 정책). `src/layouts/Base.astro`에 인증 스크립트가
이미 들어가 있습니다.

## 이미지는 AI로 생성합니다

`korea-blog`와 완전히 동일한 방식입니다 — `GEMINI_API_KEY`가 있으면 완전 자동, 없으면
PR에 남는 체크리스트를 따라 https://www.bing.com/images/create 에서 수동으로 만들어 넣으면
됩니다. 자세한 내용은 다른 블로그들의 README를 참고하세요 (동일한 파이프라인입니다).

## 주제 큐 관리 (`topics/queue.yaml`)

- 초기 20개가 이미 채워져 있습니다 (성분/루틴 설명 위주). 자유롭게 순서를 바꾸거나, 항목을
  추가/삭제하세요.
- 큐가 소진되면 자동으로 뉴스 트렌드 기반으로 전환됩니다.

## 로컬에서 초안 생성 테스트

```bash
cp .env.example .env
# .env에 ANTHROPIC_API_KEY (그리고 선택적으로 GEMINI_API_KEY) 채우기
set -a && source .env && set +a
npm run generate
```

## 프로젝트 구조

```
src/content/blog/       실제 글(Markdown). 여기 있는 파일 = 발행된 글
src/pages/               라우팅 (index, blog/[slug], about, contact, privacy, rss.xml)
src/layouts/              공통 레이아웃 (Base, BlogPost)
src/components/AdSlot.astro   애드센스 광고 슬롯 컴포넌트
public/images/blog/       자동으로 생성/다운로드된 대표 이미지들
topics/queue.yaml         직접 정한 주제 대기열
topics/used-trends.json   트렌드 모드에서 이미 쓴 헤드라인 기록 (중복 방지용, 자동 생성됨)
scripts/generate-post.mjs 메인 초안 생성 스크립트
scripts/lib/               초안 생성에 쓰이는 하위 모듈들 (topic-sources, anthropic, gemini-images, images, slugify)
scripts/add-photos.mjs    이미지 수동 업로드 자동화 (npm run photos)
.github/workflows/daily-post.yml   매일 실행되는 자동화
```

## 법적 안내

`src/pages/privacy.astro`는 애드센스 심사에 필요한 최소한의 템플릿이며 법률 자문이 아닙니다.
이 블로그는 스킨케어/뷰티 정보를 다루지만 의학적 조언을 제공하지 않으며, 각 글에도 그
취지를 자연스럽게 명시하도록 되어 있습니다. 실제 서비스 지역/이용자에 맞게 검토가 필요할 수
있습니다.
