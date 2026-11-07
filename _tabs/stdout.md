---
layout: default
title: stdout
tab_id: stdout
order: 3
---

{% assign stdout_items = site.data.stdout %}

<div class="tab-page-wrap px-xl-1">
  <!-- 상단 타이틀 + 관리자 인증 시에만 보이는 글쓰기 버튼 및 인라인 에디터 -->
  {% include tab-writer.html
    tab_id="stdout"
    title="stdout · 잡학 IT 지식"
    desc="알아두면 쓸모 있는 IT 지식을 한 줄씩 가볍게 만나보세요."
  %}

  <!-- yangjunman.com/blog 시리즈 페이지 스타일: 글 개수 + 인라인 검색 바 -->
  <div class="yj-tab-toolbar">
    <p class="yj-tab-count"><strong id="tab-visible-count">{{ stdout_items.size }}</strong>개의 기록</p>
    <div class="yj-tab-search">
      <label for="tab-search-input" class="visually-hidden">기록 검색</label>
      <input
        id="tab-search-input"
        type="search"
        placeholder="지식 검색..."
        class="yj-tab-search-input"
        autocomplete="off"
      >
      <span class="yj-tab-search-icon" aria-hidden="true">
        <i class="fas fa-search"></i>
      </span>
    </div>
  </div>

  <div id="stdout-list" class="stdout-list">
    {% for item in stdout_items %}
      <article class="stdout-row tab-filter-card" data-fields="{{ item.field }}">
        <p class="stdout-row-text">{{ item.content }}</p>
        {% if item.code and item.code != '' %}
          <code class="stdout-row-code">{{ item.code }}</code>
        {% endif %}
        <div class="stdout-row-meta">
          <span class="stdout-row-tab">stdout</span>
          <span class="stdout-row-sep">·</span>
          <span class="stdout-row-field">{{ item.field }}</span>
          <span class="stdout-row-sep">·</span>
          <time class="stdout-row-date">{{ item.date | replace: '-', '. ' }}</time>
          <span class="stdout-row-sep">·</span>
          <span class="stdout-row-read">1분 읽기</span>
        </div>
      </article>
    {% endfor %}
  </div>

  <p id="tab-empty-msg" class="tab-empty-msg d-none">
    선택한 조건에 일치하는 한 줄 지식이 없습니다.
  </p>
</div>

<style>
  .tab-page-wrap {
    margin-top: 1.75rem;
  }

  .tab-clean-header {
    margin-bottom: 1.5rem;
    padding: 0 0.15rem;
  }

  .tab-clean-title {
    margin: 0 0 0.45rem;
    font-size: 1.85rem;
    font-weight: 700;
    letter-spacing: -0.035em;
    color: #191f28;
  }

  .tab-clean-desc {
    margin: 0;
    font-size: 0.96rem;
    color: #6b7684;
    line-height: 1.6;
  }

  .yj-tab-toolbar {
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-wrap: wrap;
    gap: 0.85rem;
    padding-bottom: 1.15rem;
    margin-bottom: 1.5rem;
    border-bottom: 1px solid #e5e8eb;
  }

  .yj-tab-count {
    margin: 0;
    font-size: 0.88rem;
    color: #6b7684;
  }

  .yj-tab-count strong {
    color: #191f28;
    font-weight: 600;
  }

  .yj-tab-search {
    position: relative;
    width: 100%;
    max-width: 250px;
  }

  .yj-tab-search-input {
    width: 100%;
    height: 2.35rem;
    padding: 0 2.2rem 0 0.85rem;
    border: 1px solid #e5e8eb;
    border-radius: 6px;
    background-color: #ffffff;
    font-size: 0.86rem;
    color: #191f28;
    outline: none;
  }

  .yj-tab-search-input:focus {
    border-color: #3182f6;
    box-shadow: 0 0 0 2px rgba(49, 130, 246, 0.12);
  }

  .yj-tab-search-icon {
    position: absolute;
    right: 0.75rem;
    top: 50%;
    transform: translateY(-50%);
    font-size: 0.8rem;
    color: #8b95a1;
    pointer-events: none;
  }

  .stdout-list {
    display: flex;
    flex-direction: column;
    gap: 0.9rem;
    margin-bottom: 2rem;
  }

  .stdout-row {
    background-color: #ffffff;
    border: 1px solid #e5e8eb;
    border-radius: 12px;
    padding: 1.25rem 1.5rem;
    transition: border-color 0.22s ease, box-shadow 0.22s ease;
  }

  .stdout-row:hover {
    border-color: rgba(25, 31, 40, 0.35);
    box-shadow: 0 4px 14px rgba(0, 0, 0, 0.05);
  }

  .stdout-row-text {
    margin: 0;
    font-size: 0.96rem;
    line-height: 1.65;
    color: #191f28;
    font-weight: 500;
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

  .stdout-row-meta {
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    gap: 0.35rem;
    margin-top: 0.75rem;
    font-size: 0.78rem;
    color: #8b95a1;
  }

  .stdout-row-tab {
    font-weight: 700;
    color: #333d4b;
  }

  .stdout-row-field {
    font-weight: 600;
    color: #3182f6;
  }

  .stdout-row-sep {
    color: #d1d6db;
  }

  .stdout-row-read {
    font-weight: 500;
    color: #6b7684;
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
    const searchInput = document.getElementById('tab-search-input');
    const countEl = document.getElementById('tab-visible-count');

    let selectedFields = [];
    let searchQuery = '';

    function applyFilter() {
      let visible = 0;
      cards.forEach((card) => {
        const fields = (card.dataset.fields || '').split('|').map((s) => s.trim());
        const fieldMatch = selectedFields.length === 0 || fields.some((f) => selectedFields.includes(f));
        const textMatch = !searchQuery || card.textContent.toLowerCase().includes(searchQuery);
        const show = fieldMatch && textMatch;
        card.classList.toggle('d-none', !show);
        if (show) visible++;
      });
      if (countEl) countEl.textContent = String(visible);
      if (emptyMsg) {
        emptyMsg.classList.toggle('d-none', visible > 0);
      }
    }

    window.addEventListener('fieldFilterChange', (e) => {
      selectedFields = e.detail.selectedFields || [];
      applyFilter();
    });

    searchInput?.addEventListener('input', () => {
      searchQuery = searchInput.value.trim().toLowerCase();
      applyFilter();
    });
  });
</script>
