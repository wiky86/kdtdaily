# 하루 7분 금융×데이터 — 데일리 리딩

유비온 디지털교육센터 · K-Digital Training 「빅데이터를 활용한 디지털금융 전문가 과정」(2026.10.30 – 2027.04.16) 수강생용 데일리 학습자료 사이트.
`index.html` 한 파일로 동작하며, 글은 Supabase(wiky86's Project)에서 읽어옵니다. 로그인 없음.

## 올리는 법

**GitHub Pages**: 저장소에 `index.html`을 올리고 Settings → Pages → Branch `main` / root 선택.
**Netlify**: app.netlify.com/drop 에 이 폴더를 끌어다 놓기.

페이지 안의 키는 Supabase 공개용(읽기 전용) 키입니다. 쓰기는 DB 정책으로 막혀 있습니다.

## DB 구조 (모두 `public` 스키마)

| 테이블 | 용도 |
|---|---|
| `daily_weeks` | 1~24주차 테마, 주제, 금요일 논문 후보(`paper_hint`) |
| `daily_holidays` | 수업이 없는 날 (kind: 공휴일 · 휴강). 글을 발행하지 않음 |
| `daily_jobs` | 직무 지도 카드 (`slug`로 글과 연결) |
| `daily_articles` | 글 본문 |
| `daily_terms` | 용어사전 (글에 연결) |

**공개 규칙**: 글은 `status = 'published'` 이고 `publish_date <= 오늘(한국시간)` 일 때만 보입니다.
미리 써 둔 글은 날짜가 될 때 자동으로 열립니다. 용어도 연결된 글이 공개될 때 함께 보입니다.

## 글 한 편 넣기 (예약 작업이 따르는 형식)

```sql
with a as (
  insert into public.daily_articles
    (slug, publish_date, week_no, corner, title, subtitle, summary, body_md, quiz, tags, reading_min, sources, related_job, status)
  values (
    'w01-mon-financial-market',          -- 영소문자·숫자·하이픈
    '2026-11-02', 1,
    'concept',                           -- concept(월) job(화) code(수) issue(목) paper(금)
    '제목', '부제',
    array['요약 1','요약 2','요약 3'],
    $md$ 마크다운 본문. 수식은 $x$ 또는 $$x$$, 코드는 ```python 블록 $md$,
    '[{"q":"질문","options":["보기1","보기2","보기3"],"answer":1,"explain":"해설"}]',
    array['태그1','태그2'],
    6,
    '[{"title":"출처 제목","url":"https://...","note":"선택"}]',
    null,                                -- 직무 slug 또는 null
    'published'
  ) returning id)
insert into public.daily_terms (term, term_en, definition, article_id)
select t.term, t.en, t.def, a.id from a,
  (values ('용어','English','정의')) as t(term, en, def)
on conflict (term) do nothing;
```
