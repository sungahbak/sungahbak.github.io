---
layout: default
title: root
tab_id: root
order: 2
---

{% assign root_posts = '' | split: '' %}
{% for post in site.posts %}
  {% assign first_cat = post.categories[0] | downcase %}
  {% if first_cat == 'root' or post.tags contains '개념' or post.tags contains 'root' %}
    {% if post.hidden != true %}
      {% assign root_posts = root_posts | push: post %}
    {% endif %}
  {% endif %}
{% endfor %}

<div class="tab-page-wrap px-xl-1">
  <!-- 상단 타이틀 + 우측 글쓰기 버튼 및 인라인 에디터 -->
  {% include tab-writer.html
    tab_id="root"
    title="root · 개념"
    desc="기술의 핵심 개념을 뿌리부터 차근차근 이해해 보세요."
  %}

  <div id="root-post-list" class="home-post-list">
    {% for post in root_posts %}
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
          <div class="home-post-meta">
            <span class="hp-tab">root</span>
            <span class="hp-sep">/</span>
            <span class="hp-field">{{ field_name }}</span>
            <span class="hp-sep">/</span>
            <time class="hp-date" datetime="{{ post.date | date_to_xmlschema }}">
              {{- post.date | date: '%Y. %m. %d' -}}
            </time>
            <span class="hp-dot">·</span>
            <span class="hp-read">{{ read_min }}분 읽기</span>
          </div>

          <h2 class="home-post-title">{{ post.title }}</h2>
          <p class="home-post-summary">{% include post-summary.html %}</p>
        </a>
      </article>
    {% endfor %}
  </div>

  <p id="tab-empty-msg" class="tab-empty-msg{% if root_posts.size > 0 %} d-none{% endif %}">
    아직 작성된 개념 정리 글이 없습니다.
  </p>

  <nav id="tab-pagination" class="home-pagination" aria-label="페이지 탐색"></nav>
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

  .home-post-list {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }

  .home-post-card {
    background-color: #ffffff;
    border: 1px solid rgba(0, 27, 55, 0.06);
    border-radius: 16px;
    box-shadow: 0 2px 12px rgba(0, 27, 55, 0.04), 0 1px 2px rgba(0, 27, 55, 0.02);
    transition: transform 0.18s ease, box-shadow 0.18s ease, border-color 0.18s ease;
  }

  .home-post-card:hover {
    transform: translateY(-2px);
    border-color: rgba(49, 130, 246, 0.2);
    box-shadow: 0 8px 24px rgba(0, 27, 55, 0.07);
  }

  .home-post-link {
    display: block;
    padding: 1.5rem 1.7rem;
    text-decoration: none;
    color: inherit;
    border-bottom: none !important;
  }

  .home-post-meta {
    display: flex;
    align-items: center;
    flex-wrap: wrap;
    gap: 0.4rem;
    margin-bottom: 0.6rem;
    font-size: 0.84rem;
    line-height: 1.4;
    color: #8b95a1;
  }

  .home-post-meta .hp-tab {
    font-weight: 600;
    color: #3182f6;
  }

  .home-post-meta .hp-field {
    font-weight: 500;
    color: #4e5968;
  }

  .home-post-meta .hp-sep,
  .home-post-meta .hp-dot {
    color: #d1d6db;
  }

  .home-post-title {
    margin: 0 0 0.45rem;
    font-size: 1.25rem;
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
    margin: 0;
    font-size: 0.94rem;
    line-height: 1.62;
    color: #4e5968;
  }

  .tab-empty-msg {
    padding: 2rem 0.2rem;
    font-size: 0.95rem;
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

    let activeFields = [];
    let currentPage = 1;

    function getFiltered() {
      if (activeFields.length === 0) return cards;
      return cards.filter((card) => {
        const fields = (card.dataset.fields || '').split('|').map((s) => s.trim());
        return fields.some((f) => activeFields.includes(f));
      });
    }

    function render() {
      const filtered = getFiltered();
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

    render();
  });
</script>
