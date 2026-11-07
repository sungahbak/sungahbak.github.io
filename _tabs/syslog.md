---
layout: default
title: syslog
tab_id: syslog
order: 1
---

{% assign syslog_posts = '' | split: '' %}
{% for post in site.posts %}
  {% assign first_cat = post.categories[0] | downcase %}
  {% assign is_root = false %}
  {% if first_cat == 'root' or post.tags contains '개념' or post.tags contains 'root' %}
    {% assign is_root = true %}
  {% endif %}
  {% if first_cat == 'syslog' or is_root == false %}
    {% if post.hidden != true %}
      {% assign syslog_posts = syslog_posts | push: post %}
    {% endif %}
  {% endif %}
{% endfor %}

<div class="tab-page-wrap px-xl-1">
  <!-- 상단 타이틀 + 관리자 인증 시에만 보이는 글쓰기 버튼 및 인라인 에디터 -->
  {% include tab-writer.html
    tab_id="syslog"
    title="syslog · 트러블슈팅"
    desc="시스템의 원리를 이해하고 문제를 해결한 트러블슈팅 기록입니다."
  %}

  <!-- yangjunman.com/blog 시리즈 페이지 스타일: 글 개수 + 인라인 검색 바 -->
  <div class="yj-tab-toolbar">
    <p class="yj-tab-count"><strong id="tab-visible-count">{{ syslog_posts.size }}</strong>개의 글</p>
    <div class="yj-tab-search">
      <label for="tab-search-input" class="visually-hidden">글 검색</label>
      <input
        id="tab-search-input"
        type="search"
        placeholder="글 검색..."
        class="yj-tab-search-input"
        autocomplete="off"
      >
      <span class="yj-tab-search-icon" aria-hidden="true">
        <i class="fas fa-search"></i>
      </span>
    </div>
  </div>

  <div id="syslog-post-list" class="home-post-list">
    {% for post in syslog_posts %}
      {% assign first_cat = post.categories[0] | downcase %}
      {% if first_cat == 'syslog' or first_cat == 'root' or first_cat == 'stdout' %}
        {% assign field_list = post.categories | slice: 1, post.categories.size %}
      {% else %}
        {% assign field_list = post.categories %}
      {% endif %}
      {% if field_list.size > 0 %}
        {% assign field_name = field_list | join: ' · ' %}
        {% assign field_data = field_list | join: '|' %}
      {% else %}
        {% assign field_name = '일반' %}
        {% assign field_data = '일반' %}
      {% endif %}

      {% assign words = post.content | strip_html | number_of_words %}
      {% assign read_min = words | divided_by: 180 | at_least: 1 %}

      <article class="home-post-card tab-filter-card" data-fields="{{ field_data }}">
        <a href="{{ post.url | relative_url }}" class="home-post-link">
          <div class="home-post-body">
            <h2 class="home-post-title">{{ post.title }}</h2>
            <p class="home-post-summary">{% include post-summary.html %}</p>
          </div>

          <div class="home-post-meta">
            <span class="hp-tab">syslog</span>
            <span class="hp-dot">·</span>
            <span class="hp-field">{{ field_name }}</span>
            <span class="hp-dot">·</span>
            <time class="hp-date" datetime="{{ post.date | date_to_xmlschema }}">
              {{- post.date | date: '%Y. %m. %d' -}}
            </time>
            <span class="hp-dot">·</span>
            <span class="hp-read">{{ read_min }}분 읽기</span>
          </div>
        </a>
      </article>
    {% endfor %}
  </div>

  <p id="tab-empty-msg" class="tab-empty-msg d-none">
    선택한 조건에 일치하는 글이 없습니다.
  </p>

  <nav id="tab-pagination" class="home-pagination" aria-label="페이지 탐색"></nav>
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

  .home-post-list {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .home-post-card {
    background-color: #ffffff;
    border: 1px solid #e5e8eb;
    border-radius: 12px;
    transition: border-color 0.22s ease, box-shadow 0.22s ease;
  }

  .home-post-card:hover {
    border-color: rgba(25, 31, 40, 0.35);
    box-shadow: 0 4px 14px rgba(0, 0, 0, 0.05);
  }

  .home-post-link {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    padding: 1.35rem 1.5rem;
    text-decoration: none;
    color: inherit;
    border-bottom: none !important;
  }

  .home-post-title {
    margin: 0;
    font-size: 1.15rem;
    font-weight: 700;
    line-height: 1.42;
    letter-spacing: -0.025em;
    color: #191f28;
    transition: color 0.16s ease;
  }

  .home-post-link:hover .home-post-title {
    color: #3182f6;
  }

  .home-post-summary {
    margin: 0.5rem 0 0;
    font-size: 0.88rem;
    line-height: 1.65;
    color: #6b7684;
  }

  .home-post-meta {
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    gap: 0.4rem;
    margin-top: 0.9rem;
    font-size: 0.78rem;
    color: #8b95a1;
  }

  .home-post-meta .hp-tab {
    font-weight: 700;
    color: #333d4b;
  }

  .home-post-meta .hp-field {
    font-weight: 600;
    color: #3182f6;
  }

  .home-post-meta .hp-dot {
    color: #d1d6db;
  }

  .home-post-meta .hp-read {
    font-weight: 500;
    color: #6b7684;
  }

  .tab-empty-msg {
    padding: 2.5rem 0.5rem;
    font-size: 0.92rem;
    color: #8b95a1;
  }

  .home-pagination {
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 0.3rem;
    margin: 2.25rem 0 1.5rem;
    user-select: none;
  }

  .hp-page-btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    min-width: 2.2rem;
    height: 2.2rem;
    padding: 0 0.5rem;
    border-radius: 999px;
    font-size: 0.9rem;
    font-weight: 500;
    color: #6b7684;
    background-color: transparent;
    border: none;
    cursor: pointer;
    transition: background-color 0.15s ease, color 0.15s ease;
  }

  .hp-page-btn:hover:not(.disabled):not(.active) {
    background-color: #f2f4f6;
    color: #191f28;
  }

  .hp-page-btn.active {
    background-color: #f2f4f6;
    color: #191f28;
    font-weight: 700;
  }

  .hp-page-btn.disabled {
    color: #d1d6db;
    cursor: default;
  }
</style>

<script>
  document.addEventListener('DOMContentLoaded', () => {
    const PER_PAGE = 6;
    const cards = Array.from(document.querySelectorAll('.tab-filter-card'));
    const emptyMsg = document.getElementById('tab-empty-msg');
    const paginationEl = document.getElementById('tab-pagination');
    const searchInput = document.getElementById('tab-search-input');
    const countEl = document.getElementById('tab-visible-count');

    let activeFields = [];
    let searchQuery = '';
    let currentPage = 1;

    function getFiltered() {
      return cards.filter((card) => {
        const fields = (card.dataset.fields || '').split('|').map((s) => s.trim());
        const fieldMatch = activeFields.length === 0 || fields.some((f) => activeFields.includes(f));
        const textMatch = !searchQuery || card.textContent.toLowerCase().includes(searchQuery);
        return fieldMatch && textMatch;
      });
    }

    function render() {
      const filtered = getFiltered();
      if (countEl) countEl.textContent = String(filtered.length);
      const totalPages = Math.max(1, Math.ceil(filtered.length / PER_PAGE));
      if (currentPage > totalPages) currentPage = 1;

      cards.forEach((c) => c.classList.add('d-none'));

      if (filtered.length === 0) {
        emptyMsg.classList.remove('d-none');
        paginationEl.innerHTML = '';
        return;
      }

      emptyMsg.classList.add('d-none');
      const start = (currentPage - 1) * PER_PAGE;
      filtered.slice(start, start + PER_PAGE).forEach((c) => c.classList.remove('d-none'));

      let html = `<button type="button" class="hp-page-btn ${currentPage === 1 ? 'disabled' : ''}" data-page="${currentPage - 1}" ${currentPage === 1 ? 'disabled' : ''}>&lt;</button>`;
      for (let p = 1; p <= totalPages; p++) {
        html += `<button type="button" class="hp-page-btn ${p === currentPage ? 'active' : ''}" data-page="${p}">${p}</button>`;
      }
      html += `<button type="button" class="hp-page-btn ${currentPage === totalPages ? 'disabled' : ''}" data-page="${currentPage + 1}" ${currentPage === totalPages ? 'disabled' : ''}>&gt;</button>`;
      paginationEl.innerHTML = html;

      paginationEl.querySelectorAll('button[data-page]').forEach((btn) => {
        btn.addEventListener('click', () => {
          const target = Number(btn.dataset.page);
          if (target >= 1 && target <= totalPages && target !== currentPage) {
            currentPage = target;
            render();
            window.scrollTo({ top: 0, behavior: 'smooth' });
          }
        });
      });
    }

    window.addEventListener('fieldFilterChange', (e) => {
      activeFields = e.detail.selectedFields || [];
      currentPage = 1;
      render();
    });

    searchInput?.addEventListener('input', () => {
      searchQuery = searchInput.value.trim().toLowerCase();
      currentPage = 1;
      render();
    });

    render();
  });
</script>
