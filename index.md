---
layout: default
title: Home
---

<div id="this-day" class="this-day-box" style="display:none">
  <span class="this-day-label">📅 {{ site.data.blog.this_day_label | default: "This day in history" }}</span>
  <a id="this-day-link" href="#" class="this-day-title"></a>
  <span id="this-day-year" class="this-day-year"></span>
</div>

<div class="list-controls">
  <div class="filter-pills" id="category-filters">
    <button class="filter-pill active" data-filter="">All</button>
    {% for cat in site.data.blog.categories %}
    <button class="filter-pill" data-filter="{{ cat }}">{{ cat }}</button>
    {% endfor %}
  </div>
  <input type="search" id="post-search" class="search-input" placeholder="Search… (press /)" aria-label="Search milestones">
</div>

<p id="no-results" class="no-results" style="display:none">No milestones match your search.</p>

<!-- ─── Cards view ─── -->
<div id="view-cards">
  <ul class="post-list" id="post-list">
    {% for post in site.posts %}
    <li class="post-card" data-category="{{ post.category }}" data-era="{{ post.era }}" data-post-id="{{ post.id }}" data-search="{{ post.title | downcase }} {{ post.subtitle | downcase }} {{ post.category | downcase }} {{ post.era | downcase }} {{ post.location | downcase }}">
      {% if post.image %}
      <a href="{{ post.url | relative_url }}" class="post-card-thumb">
        <img src="{{ post.image | relative_url }}" alt="{{ post.title }}" loading="lazy">
      </a>
      {% else %}
      <div class="post-card-thumb post-card-thumb--empty">
        <span>{{ post.event_date | date: "%Y" }}</span>
      </div>
      {% endif %}
      <div class="post-card-body">
        <div class="post-card-top">
          {% if post.era %}{% assign era_page = site.eras | where: "name", post.era | first %}{% if era_page %}<a href="{{ era_page.url | relative_url }}" class="era-badge">{{ post.era | split: '(' | first | strip }}</a>{% else %}<span class="era-badge">{{ post.era | split: '(' | first | strip }}</span>{% endif %}{% endif %}
          {% if post.category %}<span class="category-badge cat-{{ post.category | downcase | replace: ' & ', '-' | replace: ' ', '-' }}">{{ post.category }}</span>{% endif %}
        </div>
        <div class="post-card-title"><a href="{{ post.url | relative_url }}">{{ post.title }}</a></div>
        {% if post.subtitle %}<p class="post-card-excerpt">{{ post.subtitle }}</p>{% endif %}
        <div class="post-card-meta">
          <span class="post-card-date">{{ post.event_date | default: post.date | date: "%Y" }}</span>
          {% if post.location %}<span>📍 {{ post.location }}</span>{% endif %}
        </div>
      </div>
    </li>
    {% endfor %}
  </ul>
</div>


<script>
(function () {
  const params      = new URLSearchParams(location.search);
  let activeFilter  = params.get('cat') || '';
  let searchQuery   = params.get('q')   || '';

  const searchEl = document.getElementById('post-search');
  const pills    = document.querySelectorAll('#category-filters .filter-pill');
  const noRes    = document.getElementById('no-results');
  const postCards = document.querySelectorAll('#post-list .post-card');

  function updateURL() {
    const p = new URLSearchParams();
    if (activeFilter) p.set('cat', activeFilter);
    if (searchQuery)  p.set('q', searchQuery);
    const newURL = location.pathname + (p.toString() ? '?' + p.toString() : '');
    history.pushState(null, '', newURL);
  }

  function applyFilters() {
    const q = searchQuery.toLowerCase();
    let visible = 0;
    postCards.forEach(card => {
      const text = card.dataset.search || card.textContent.toLowerCase();
      const cat  = card.dataset.category || '';
      const show = (!q || text.includes(q)) && (!activeFilter || cat === activeFilter);
      card.style.display = show ? '' : 'none';
      if (show) visible++;
    });
    noRes.style.display = visible === 0 ? '' : 'none';
  }

  searchEl.value = searchQuery;
  pills.forEach(p => p.classList.toggle('active', p.dataset.filter === activeFilter));
  if (!activeFilter) document.querySelector('.filter-pill[data-filter=""]').classList.add('active');
  applyFilters();

  searchEl.addEventListener('input', () => {
    searchQuery = searchEl.value.trim();
    updateURL();
    applyFilters();
  });

  pills.forEach(pill => {
    pill.addEventListener('click', () => {
      pills.forEach(p => p.classList.remove('active'));
      pill.classList.add('active');
      activeFilter = pill.dataset.filter;
      updateURL();
      applyFilters();
    });
  });

  document.addEventListener('keydown', function (e) {
    if (e.key === '/' && document.activeElement !== searchEl) {
      e.preventDefault();
      searchEl.focus();
      searchEl.select();
    }
    if (e.key === 'Escape' && document.activeElement === searchEl) {
      searchEl.blur();
      searchEl.value = '';
      searchQuery = '';
      updateURL();
      applyFilters();
    }
  });

  // ─── This day in history ─────────────────────────────────────────────────
  const allPosts = [{% for post in site.posts %}{"title":{{ post.title | jsonify }},"url":{{ post.url | relative_url | jsonify }},"date":{{ post.event_date | jsonify }}}{% unless forloop.last %},{% endunless %}{% endfor %}];
  const today = new Date();
  const mm    = String(today.getMonth() + 1).padStart(2, '0');
  const dd    = String(today.getDate()).padStart(2, '0');
  const match = allPosts.find(p => p.date && p.date.slice(5, 10) === mm + '-' + dd);
  if (match) {
    document.getElementById('this-day-link').textContent = match.title;
    document.getElementById('this-day-link').href        = match.url;
    document.getElementById('this-day-year').textContent = match.date.slice(0, 4);
    document.getElementById('this-day').style.display    = '';
  }

  // ─── Read tracking ────────────────────────────────────────────────────────
  const read = JSON.parse(localStorage.getItem('{{ site.data.blog.read_key | default: "history-read" }}') || '[]');
  if (read.length) {
    postCards.forEach(card => {
      if (read.includes(card.dataset.postId)) card.classList.add('post-card--read');
    });
  }
})();
</script>
