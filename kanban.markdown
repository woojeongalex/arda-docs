---
layout: default
title: 팀 칸반
nav_order: 9
permalink: /kanban/
---

# 팀 칸반 보드

<style>
.kb-board { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; margin: 1rem 0 2rem; }
@media (max-width: 768px) { .kb-board { grid-template-columns: 1fr; } }
.kb-col { background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.1); border-radius: 8px; padding: 10px; }
.kb-col-title { font-weight: 700; font-size: 0.8rem; letter-spacing: 0.05em; text-transform: uppercase; opacity: 0.8; margin: 2px 0 10px 4px; }
.kb-card { border: 1px solid rgba(255,255,255,0.12); border-left-width: 4px; border-radius: 6px; padding: 8px 10px; margin-bottom: 8px; background: rgba(0,0,0,0.25); }
.kb-card-title { font-size: 0.82rem; line-height: 1.45; margin-bottom: 5px; }
.kb-meta { display: flex; flex-wrap: wrap; gap: 5px; align-items: center; }
.kb-owner { font-size: 0.66rem; font-weight: 700; padding: 1px 8px; border-radius: 999px; color: #111; }
.kb-tag { font-size: 0.66rem; padding: 1px 7px; border-radius: 4px; border: 1px solid rgba(255,255,255,0.2); opacity: 0.85; }
.kb-due { font-size: 0.66rem; opacity: 0.65; }
.kb-note { font-size: 0.7rem; opacity: 0.6; margin-top: 4px; }
.kb-legend { display: flex; flex-wrap: wrap; gap: 8px; margin-bottom: 0.5rem; }
.kb-count { opacity: 0.6; font-weight: 400; }
details.kb-rules { margin-top: 1.5rem; }
details.kb-rules summary { cursor: pointer; font-size: 0.78rem; opacity: 0.6; }
details.kb-rules summary:hover { opacity: 1; text-decoration: underline; }
/* 카드 상세 펼침 (2026-09-03) — detail 있는 카드는 클릭하면 상세가 열린다 */
details.kb-card { margin: 0 0 8px; }
details.kb-card summary { cursor: pointer; list-style: none; }
details.kb-card summary::-webkit-details-marker { display: none; }
details.kb-card summary:hover .kb-card-title { text-decoration: underline; }
.kb-more { font-size: 0.64rem; padding: 1px 7px; border-radius: 4px; background: rgba(122,162,247,0.18); border: 1px solid rgba(122,162,247,0.45); }
details.kb-card[open] .kb-more { opacity: 0.5; }
.kb-detail { font-size: 0.74rem; line-height: 1.65; opacity: 0.85; margin-top: 8px; padding-top: 8px; border-top: 1px dashed rgba(255,255,255,0.15); white-space: pre-line; }
</style>

<div class="kb-legend">
{% for member in site.data.kanban %}{% assign m = member[1] %}
  <span class="kb-owner" style="background: {{ m.color }};">{{ m.name | default: m.owner }} · {{ m.domain }}</span>
{% endfor %}
</div>

<div class="kb-board">
{% assign columns = "todo:할 일,doing:진행 중,done:완료" | split: "," %}
{% for col in columns %}
  {% assign parts = col | split: ":" %}
  {% assign status = parts[0] %}
  <div class="kb-col">
    {% assign total = 0 %}
    {% for member in site.data.kanban %}{% assign m = member[1] %}{% for card in m.cards %}{% if card.status == status %}{% assign total = total | plus: 1 %}{% endif %}{% endfor %}{% endfor %}
    <div class="kb-col-title">{{ parts[1] }} <span class="kb-count">({{ total }})</span></div>
    {% for member in site.data.kanban %}
      {% assign m = member[1] %}
      {% for card in m.cards %}
        {% if card.status == status %}
        {% if card.detail %}
        <details class="kb-card" style="border-left-color: {{ m.color }};">
          <summary>
            <div class="kb-card-title">{{ card.title }}</div>
            <div class="kb-meta">
              <span class="kb-owner" style="background: {{ m.color }};">{{ m.name | default: m.owner }}</span>
              {% if card.feature %}<span class="kb-tag">{{ card.feature }}</span>{% endif %}
              {% if card.due %}<span class="kb-due">~{{ card.due }}</span>{% endif %}
              <span class="kb-more">상세 ▾</span>
            </div>
            {% if card.note %}<div class="kb-note">{{ card.note }}</div>{% endif %}
          </summary>
          <div class="kb-detail">{{ card.detail | strip }}</div>
        </details>
        {% else %}
        <div class="kb-card" style="border-left-color: {{ m.color }};">
          <div class="kb-card-title">{{ card.title }}</div>
          <div class="kb-meta">
            <span class="kb-owner" style="background: {{ m.color }};">{{ m.name | default: m.owner }}</span>
            {% if card.feature %}<span class="kb-tag">{{ card.feature }}</span>{% endif %}
            {% if card.due %}<span class="kb-due">~{{ card.due }}</span>{% endif %}
          </div>
          {% if card.note %}<div class="kb-note">{{ card.note }}</div>{% endif %}
        </div>
        {% endif %}
        {% endif %}
      {% endfor %}
    {% endfor %}
  </div>
{% endfor %}
</div>

<details markdown="1" class="kb-rules">
<summary>📖 사용 규칙 보기 (카드 편집 방법 · 충돌 방지)</summary>

카드는 `_data/kanban/<자기 GitHub 아이디>.yml`에서만 편집한다 — 한 사람이 파일 하나를 소유하므로 4명이 동시에 작업해도 git 충돌이 나지 않는다.

1. **자기 파일만 수정한다.** 본인 소유 파일 외에는 손대지 않는다. 남의 카드에 할 말이 있으면 팀 채널로.
2. **카드 형식**은 파일 안 기존 항목을 복사해서 쓴다. `status`는 `todo | doing | done` 세 값만.
   ```yaml
   - title: "카드 제목"
     feature: "D3"        # 기능 번호 또는 주차 (선택)
     status: todo         # todo | doing | done
     done: 2026-09-02     # 완료일 — done으로 바꿀 때 함께 기입 (개발 로그 자동 집계용)
     due: 2026-09-04      # 마감 (선택)
     note: ""             # 한 줄 메모 (선택 — 제목·메모·상세에 ¦ 문자는 금지)
     detail: |            # 상세 요약 (선택) — 개발 로그에서 클릭해 들어가는 상세 페이지에 표시
       무엇을 어떻게 바꿨는지 2~4줄
   ```
3. **push 전 `git pull --rebase`.** 서로 다른 파일이라 rebase가 항상 깨끗하게 통과한다. 충돌이 났다면 파일 소유자 버전 우선, `push --force` 금지.
4. **포스트(`_posts/`)도 같은 원칙**: 파일명을 `YYYY-MM-DD-<아이디>-<주제>.markdown`으로 만들어 한 파일을 한 사람만 만지게 한다.
5. **`done`으로 바꿀 때 `done: 날짜`를 함께 기입한다** — [개발 로그]({{ site.baseurl }}/devlog/) 완료 타임라인이 이 날짜로 자동 집계된다. 카드 정리(삭제)는 주간 다이제스트(금요일) 이후 각자 판단 — 단, 삭제하면 개발 로그에서도 사라지므로 **기록을 남기려면 유지**를 권장.
6. 팀 채널에 공유된 `Arda-칸반-클로드-지시서.md`를 Claude에게 읽히면 위 전부를 말로 대신 시킬 수 있다.

</details>
