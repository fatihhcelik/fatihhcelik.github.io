---
title: Advisories
icon: fas fa-bullhorn
order: 5
---

<style>
/* Search Controls */
.advisories-controls {
  margin-bottom: 1.5rem;
  display: flex;
  justify-content: flex-end;
}

.search-box {
  position: relative;
  width: 280px;
}

.search-box i {
  position: absolute;
  left: 0.75rem;
  top: 50%;
  transform: translateY(-50%);
  color: var(--text-muted-color, #888);
  font-size: 0.85rem;
}

.search-input {
  width: 100%;
  padding: 0.5rem 0.75rem 0.5rem 2.2rem;
  border-radius: 8px;
  border: 1px solid var(--main-border-color, #444);
  background: var(--search-wrapper-bg, #222);
  color: var(--text-color, #c9d1d9);
  font-size: 0.85rem;
  transition: all 0.2s ease;
  outline: none;
}

.search-input:focus {
  border-color: var(--link-color, #58a6ff);
  box-shadow: 0 0 0 3px rgba(88, 166, 255, 0.15);
}

/* Timeline Layout */
.timeline-section {
  position: relative;
  margin-bottom: 2.5rem;
  padding-left: 2rem;
}

.timeline-section::before {
  content: '';
  position: absolute;
  left: 0.5rem;
  top: 0;
  bottom: 0;
  width: 2px;
  background: var(--timeline-color, #333);
  border-radius: 2px;
}

.timeline-year {
  position: relative;
  font-size: 1.4rem;
  font-weight: 800;
  margin: 0 0 1.5rem 0;
  color: var(--text-color, #c9d1d9);
  display: flex;
  align-items: center;
}

.timeline-year::before {
  content: '';
  position: absolute;
  left: -2rem;
  width: 14px;
  height: 14px;
  background: var(--timeline-node-bg, #969898);
  border: 3px solid var(--link-color, #58a6ff);
  border-radius: 50%;
  box-shadow: 0 0 0 4px var(--body-bg, #0d1117);
}

/* Grid */
.advisories-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 1.5rem;
}

/* Card */
.advisory-card {
  background: var(--card-bg, var(--button-bg, #fff));
  border: 1px solid var(--card-border-color, var(--btn-border-color, #e9ecef));
  border-radius: 12px;
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  box-shadow: 0 2px 6px var(--card-box-shadow, rgba(0,0,0,0.05));
}

.advisory-card.hidden {
  display: none !important;
}

.advisory-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 20px var(--card-box-shadow, rgba(0,0,0,0.15));
  border-color: var(--link-color, #58a6ff);
}

/* Header */
.advisory-header {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  margin-bottom: 1rem;
  gap: 0.6rem;
}

.advisory-cve {
  font-size: 1.2rem;
  font-weight: 700;
  color: var(--text-color, #d8d8d8);
  margin: 0 !important;
  padding: 0 !important;
  font-family: inherit;
  border-bottom: none;
  line-height: 1.2;
}

.advisory-badges {
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  align-items: center;
}

.badge {
  font-size: 0.72rem;
  font-weight: 600;
  padding: 0.2rem 0.55rem;
  border-radius: 999px;
  background: var(--tag-bg, rgba(0, 0, 0, 0.075));
  color: var(--text-color, #333);
  border: 1px solid var(--btn-border-color, #dee2e6);
  white-space: nowrap;
}

/* Product Badge */
.badge.product {
  background: rgba(88, 166, 255, 0.12);
  color: var(--link-color, #58a6ff);
  border-color: rgba(88, 166, 255, 0.3);
}

/* Critical / Red */
.badge.type-rce, .badge.type-command-execution, .badge.type-sqli, .badge.type-command-execution---file-read {
  background: rgba(255, 59, 48, 0.15);
  color: #e53e3e;
  border-color: rgba(255, 59, 48, 0.35);
}

/* Warning / Orange */
.badge.type-xss, .badge.type-ssrf, .badge.type-unrestricted-file-upload {
  background: rgba(221, 107, 32, 0.15);
  color: #dd6b20;
  border-color: rgba(221, 107, 32, 0.35);
}

/* Info / Purple */
.badge.type-information-disclosure, .badge.type-buffer-overflow, .badge.type-heap-buffer-overflow, .badge.type-dos, .badge.type-privilege-escalation, .badge.type-csrf, .badge.type-insecure-deserialization {
  background: rgba(128, 90, 213, 0.15);
  color: #805ad5;
  border-color: rgba(128, 90, 213, 0.35);
}

/* Body */
.advisory-body {
  font-size: 0.88rem;
  line-height: 1.6;
  color: var(--text-muted-color, #a0a0a0);
  flex-grow: 1;
  margin-bottom: 1.5rem;
}

/* Footer */
.advisory-footer {
  margin-top: auto;
  padding-top: 1rem;
  border-top: 1px solid var(--main-border-color, #333);
  display: flex;
  justify-content: flex-end;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.advisory-link {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  font-size: 0.82rem;
  font-weight: 600;
  color: var(--link-color, #58a6ff);
  text-decoration: none !important;
  transition: all 0.2s ease;
  padding: 0.35rem 0.7rem;
  border-radius: 6px;
  border: 1px solid transparent;
}

.advisory-link:hover {
  text-decoration: none !important;
  gap: 0.6rem;
  background: var(--tag-bg, rgba(88, 166, 255, 0.1));
  border-color: var(--main-border-color, #444);
}

.advisory-link i {
  font-size: 0.75rem;
  transition: transform 0.2s ease;
}

.advisory-link:hover i {
  transform: translateX(3px);
}

.advisory-post-link {
  color: #4ade80 !important;
  border: 1px solid rgba(74, 222, 128, 0.2);
}

.advisory-post-link:hover {
  background: rgba(74, 222, 128, 0.1) !important;
  border-color: rgba(74, 222, 128, 0.4);
}

/* Empty State */
.no-results {
  display: none;
  text-align: center;
  padding: 3rem;
  color: var(--text-muted-color, #888);
  font-size: 1rem;
}

</style>

<div class="advisories-controls">
  <div class="search-box">
    <i class="fas fa-search"></i>
    <input type="text" id="advisory-search" class="search-input" placeholder="Search by CVE, Product, Type (e.g. n8n, RCE)..." autocomplete="off">
  </div>
</div>

<div id="no-results" class="no-results">
  <i class="fas fa-file-search" style="font-size: 2rem; margin-bottom: 1rem;"></i>
  <p>No advisories found matching your query.</p>
</div>

<!-- Extract all unique years -->
{% assign years = "" | split: "," %}
{% for advisory in site.data.advisories %}
  {% assign current_year = advisory.cve | slice: 4, 4 %}
  {% unless years contains current_year %}
    {% assign years = years | push: current_year %}
  {% endunless %}
{% endfor %}

{% assign sorted_years = years | sort | reverse %}

<div id="timeline-container">
  {% for year in sorted_years %}
    <div class="timeline-section" data-year="{{ year }}">
      <h2 class="timeline-year">{{ year }}</h2>
      <div class="advisories-grid">
      
        {% for advisory in site.data.advisories %}
          {% assign adv_year = advisory.cve | slice: 4, 4 %}
          
          {% if adv_year == year %}
            {% assign downType = advisory.type | downcase | replace: " ", "-" | replace: "/", "-" %}
            
            <div class="advisory-card" data-search="{{ advisory.cve | downcase }} {{ advisory.product | downcase }} {{ advisory.type | downcase }} {{ advisory.description | downcase }}">
              <div class="advisory-header">
                <h3 class="advisory-cve">{{ advisory.cve }}</h3>
                <div class="advisory-badges">
                  <span class="badge product"><i class="fas fa-box" style="margin-right:3px;"></i>{{ advisory.product }}</span>
                  <span class="badge type-{{ downType }}"><i class="fas fa-bug" style="margin-right:3px;"></i>{{ advisory.type }}</span>
                </div>
              </div>
              
              <div class="advisory-body">
                {{ advisory.description }}
              </div>
              
              {% if advisory.link or advisory.post_link %}
              <div class="advisory-footer">
                {% if advisory.post_link %}
                <a href="{{ advisory.post_link }}" class="advisory-link advisory-post-link">
                  <i class="fas fa-book-open"></i> Read Post
                </a>
                {% endif %}
                
                {% if advisory.link %}
                <a href="{{ advisory.link }}" target="_blank" rel="noopener noreferrer" class="advisory-link">
                  Read Advisory <i class="fas fa-arrow-right"></i>
                </a>
                {% endif %}
              </div>
              {% endif %}
            </div>
          {% endif %}
        {% endfor %}
        
      </div>
    </div>
  {% endfor %}
</div>

<script>
document.addEventListener("DOMContentLoaded", function() {
  const searchInput = document.getElementById('advisory-search');
  const cards = document.querySelectorAll('.advisory-card');
  const sections = document.querySelectorAll('.timeline-section');
  const noResults = document.getElementById('no-results');

  // Small helper to make placeholder text more clear
  searchInput.setAttribute('placeholder', 'Search (e.g. "product:n8n", "type:rce", "CVE-2026")...');

  searchInput.addEventListener('input', function(e) {
    const rawSearch = e.target.value.toLowerCase().trim();
    let totalVisible = 0;
    
    // Parse filters
    let searchProduct = null;
    let searchType = null;
    let generalSearch = rawSearch;
    
    const productMatch = rawSearch.match(/product:([^\s]+)/);
    if (productMatch) {
      searchProduct = productMatch[1];
      generalSearch = generalSearch.replace(productMatch[0], '').trim();
    }
    
    const typeMatch = rawSearch.match(/type:([^\s]+)/);
    if (typeMatch) {
      searchType = typeMatch[1];
      generalSearch = generalSearch.replace(typeMatch[0], '').trim();
    }

    cards.forEach(card => {
      let isVisible = true;
      const searchableText = card.getAttribute('data-search') || "";
      
      // We can grab product and type directly from the elements since we didn't add data-attributes inside liquid above
      const cardProduct = (card.querySelector('.badge.product')?.textContent || "").toLowerCase();
      const cardType = (card.querySelector('.badge[class*="type-"]')?.textContent || "").toLowerCase();
      
      // Check filters
      if (searchProduct && !cardProduct.includes(searchProduct)) isVisible = false;
      if (searchType && !cardType.includes(searchType)) isVisible = false;
      if (generalSearch && !searchableText.includes(generalSearch)) isVisible = false;
      
      if (isVisible) {
        card.classList.remove('hidden');
        totalVisible++;
      } else {
        card.classList.add('hidden');
      }
    });

    // Build timeline aesthetics: hide empty years
    let anySectionVisible = false;
    sections.forEach(section => {
      // Find all cards inside this section that are NOT hidden
      const visibleCardsInYear = section.querySelectorAll('.advisory-card:not(.hidden)');
      if (visibleCardsInYear.length === 0) {
        section.style.display = 'none';
      } else {
        section.style.display = 'block';
        anySectionVisible = true;
      }
    });

    // Show empty state if needed
    if (!anySectionVisible && rawSearch !== "") {
      noResults.style.display = 'block';
    } else {
      noResults.style.display = 'none';
    }
  });
});
</script>