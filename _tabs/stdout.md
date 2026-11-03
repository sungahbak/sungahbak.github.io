---
layout: default
title: stdout
tab_id: stdout
order: 3
---

{% assign stdout_items = site.data.stdout %}

<div class="tab-page-wrap px-xl-1">
  <!-- 상단 타이틀 + 우측 글쓰기 버튼 및 인라인 에디터 -->
  {% include tab-writer.html
    tab_id="stdout"
    title="stdout · 잡학 IT 지식"
    desc="알아두면 쓸모 있는 IT 지식을 한 줄씩 가볍게 만나보세요"
  %}

  <div id="stdout-list" class="stdout-list">
    {% for item in stdout_items %}
      <article class="stdout-row tab-filter-card" data-fields="{{ item.field }}">
        <div class="stdout-row-meta">
          <span class="stdout-row-field">{{ item.field }}</span>
          <span class="stdout-row-sep">·</span>
          <time class="stdout-row-date">{{ item.date | replace: '-', '. ' }}</time>
        </div>
        <p class="stdout-row-text">{{ item.content }}</p>
        {% if item.code and item.code != '' %}
          <code class="stdout-row-code">{{ item.code }}</code>
        {% endif %}
      </article>
    {% endfor %}
  </div>

  <p id="tab-empty-msg" class="tab-empty-msg d-none">
    선택한 분야에 아직 등록된 한 줄 지식이 없습니다.
  </p>
</div>

<style>
  .tab-page-wrap {
    margin-top: 1.75rem;
  }

  .tab-clean-header {
    margin-bottom: 1.75rem;
    padding: 0 0.15rem;
  }

  .tab-clean-title {
    margin: 0 0 0.45rem;
    font-size: 1.7rem;
    font-weight: 700;
    letter-spacing: -0.03em;
    color: #191f28;
  }

  .tab-clean-desc {
    margin: 0;
    font-size: 0.98rem;
    color: #6b7684;
    line-height: 1.55;
  }

  .stdout-list {
    display: flex;
    flex-direction: column;
    gap: 0.85rem;
    margin-bottom: 2rem;
  }

  .stdout-row {
    background-color: #ffffff;
    border: 1px solid rgba(0, 27, 55, 0.06);
    border-radius: 16px;
    padding: 1.25rem 1.5rem;
    box-shadow: 0 2px 12px rgba(0, 27, 55, 0.04), 0 1px 2px rgba(0, 27, 55, 0.02);
  }

  .stdout-row-meta {
    display: flex;
    align-items: center;
    gap: 0.35rem;
    margin-bottom: 0.45rem;
    font-size: 0.82rem;
    color: #8b95a1;
  }

  .stdout-row-field {
    font-weight: 600;
    color: #3182f6;
  }

  .stdout-row-sep {
    color: #d1d6db;
  }

  .stdout-row-text {
    margin: 0;
    font-size: 0.96rem;
    line-height: 1.62;
    color: #333d4b;
  }

  .stdout-row-code {
    display: inline-block;
    margin-top: 0.55rem;
    padding: 0.2rem 0.55rem;
    border-radius: 6px;
    background-color: #f6f7f9;
    color: #4e5968;
    font-size: 0.8rem;
  }

  .tab-empty-msg {
    padding: 2rem 0.2rem;
    font-size: 0.95rem;
    color: #8b95a1;
  }
</style>

<script>
  document.addEventListener('DOMContentLoaded', () => {
    const cards = Array.from(document.querySelectorAll('.tab-filter-card'));
    const emptyMsg = document.getElementById('tab-empty-msg');

    window.addEventListener('fieldFilterChange', (e) => {
      const selected = e.detail.selectedFields || [];
      let visible = 0;
      cards.forEach((card) => {
        const fields = (card.dataset.fields || '').split('|').map((s) => s.trim());
        const show = selected.length === 0 || fields.some((f) => selected.includes(f));
        card.classList.toggle('d-none', !show);
        if (show) visible++;
      });
      if (emptyMsg) {
        emptyMsg.classList.toggle('d-none', visible > 0);
      }
    });
  });
</script>

