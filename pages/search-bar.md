# Explore Omaha — Site-Wide Search Bar
## Add this as a Custom HTML block at the TOP of every page (or in your global header)

```html
<!-- SITE-WIDE SEARCH BAR -->
<div id="eo-search-wrapper" style="width:100%;background:#1a3a5c;padding:14px 20px;box-sizing:border-box;">
  <div style="max-width:980px;margin:0 auto;display:flex;align-items:center;gap:12px;">
    <div style="position:relative;flex:1;">
      <input
        id="eo-search-input"
        type="text"
        placeholder="Search events, articles, restaurants, activities..."
        oninput="eoSearch(this.value)"
        onkeydown="if(event.key==='Escape')eoCloseSearch()"
        style="width:100%;padding:12px 44px 12px 16px;border-radius:8px;border:none;font-size:1rem;outline:none;box-sizing:border-box;background:#fff;color:#1a1a2e;"
      />
      <span onclick="eoSearch(document.getElementById('eo-search-input').value)"
        style="position:absolute;right:14px;top:50%;transform:translateY(-50%);cursor:pointer;font-size:1.1rem;color:#1a3a5c;">&#128269;</span>
    </div>
    <button onclick="eoCloseSearch()" id="eo-search-close" style="display:none;background:transparent;border:1px solid rgba(255,255,255,0.4);color:#fff;padding:10px 16px;border-radius:8px;cursor:pointer;font-size:0.85rem;">
      Clear
    </button>
  </div>

  <!-- RESULTS DROPDOWN -->
  <div id="eo-search-results" style="display:none;max-width:980px;margin:10px auto 0;background:#fff;border-radius:10px;box-shadow:0 8px 24px rgba(0,0,0,0.15);max-height:420px;overflow-y:auto;">
    <div id="eo-results-inner" style="padding:8px 0;"></div>
    <div id="eo-no-results" style="display:none;padding:20px;text-align:center;color:#888;font-size:0.95rem;">
      No results found. Try a different keyword.
    </div>
  </div>
</div>

<script>
// ── Data sources ──────────────────────────────────────────────────────
const EO_BASE_URL = "https://app.base44.com/api/apps/6a30eb6225d9049a67d8f056";

let eoAllData = null;
let eoSearchTimer = null;

async function eoLoadData() {
  if (eoAllData) return eoAllData;
  try {
    const [eventsRes, articlesRes] = await Promise.all([
      fetch(`${EO_BASE_URL}/entities/OmahaFamilyEvent/records?limit=500`),
      fetch(`${EO_BASE_URL}/entities/Article/records?limit=500`)
    ]);
    const events = eventsRes.ok ? (await eventsRes.json()).records || [] : [];
    const articles = articlesRes.ok ? (await articlesRes.json()).records || [] : [];
    eoAllData = { events, articles };
    return eoAllData;
  } catch(e) {
    return { events: [], articles: [] };
  }
}

function eoSearch(query) {
  clearTimeout(eoSearchTimer);
  const q = (query || "").trim().toLowerCase();
  const input = document.getElementById("eo-search-input");
  const closeBtn = document.getElementById("eo-search-close");
  const resultsBox = document.getElementById("eo-search-results");

  if (q.length < 2) {
    resultsBox.style.display = "none";
    closeBtn.style.display = "none";
    return;
  }

  closeBtn.style.display = "block";

  eoSearchTimer = setTimeout(async () => {
    const data = await eoLoadData();
    const results = [];

    // Search events
    data.events.forEach(e => {
      const searchText = [e.event_name, e.description, e.location, e.category, e.age_range].join(" ").toLowerCase();
      if (searchText.includes(q)) {
        results.push({
          type: "event",
          title: e.event_name,
          subtitle: `${e.event_date || ""} · ${e.location || ""}`,
          badge: e.category || "Event",
          badgeColor: "#1a3a5c",
          cost: e.cost || "",
          image: e.image_url || "",
          url: null
        });
      }
    });

    // Search articles
    data.articles.forEach(a => {
      const searchText = [a.title, a.content, a.excerpt, a.category, a.seo_keywords].join(" ").toLowerCase();
      if (searchText.includes(q)) {
        results.push({
          type: "article",
          title: a.title,
          subtitle: a.excerpt ? a.excerpt.substring(0, 80) + "..." : a.category,
          badge: a.category || "Article",
          badgeColor: "#d97706",
          cost: "",
          image: a.featured_image_suggestion || "",
          url: a.slug ? `/blog/${a.slug}` : null
        });
      }
    });

    const inner = document.getElementById("eo-results-inner");
    const noResults = document.getElementById("eo-no-results");
    resultsBox.style.display = "block";

    if (results.length === 0) {
      inner.innerHTML = "";
      noResults.style.display = "block";
      return;
    }

    noResults.style.display = "none";
    inner.innerHTML = results.slice(0, 12).map(r => `
      <div onclick="${r.url ? `window.location.href='${r.url}'` : ''}"
           style="display:flex;align-items:center;gap:12px;padding:10px 16px;cursor:${r.url ? 'pointer' : 'default'};border-bottom:1px solid #f3f4f6;transition:background 0.15s;"
           onmouseover="this.style.background='#f9fafb'" onmouseout="this.style.background='#fff'">
        ${r.image && r.image.startsWith('http') ? `<img src="${r.image}" style="width:48px;height:48px;border-radius:6px;object-fit:cover;flex-shrink:0;" onerror="this.style.display='none'" />` : `<div style="width:48px;height:48px;border-radius:6px;background:#e5e7eb;flex-shrink:0;display:flex;align-items:center;justify-content:center;font-size:1.2rem;">${r.type==='event'?'📅':'📝'}</div>`}
        <div style="flex:1;min-width:0;">
          <div style="font-weight:700;font-size:0.92rem;color:#1a1a2e;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;">${r.title}</div>
          <div style="font-size:0.8rem;color:#6b7280;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;">${r.subtitle}</div>
        </div>
        <div style="display:flex;flex-direction:column;align-items:flex-end;gap:4px;flex-shrink:0;">
          <span style="background:${r.badgeColor};color:#fff;font-size:0.7rem;padding:2px 8px;border-radius:10px;font-weight:700;">${r.badge}</span>
          ${r.cost ? `<span style="font-size:0.78rem;color:#059669;font-weight:600;">${r.cost}</span>` : ''}
        </div>
      </div>
    `).join("") + (results.length > 12 ? `<div style="padding:12px 16px;text-align:center;color:#9ca3af;font-size:0.85rem;">${results.length - 12} more results — refine your search</div>` : "");

  }, 280);
}

function eoCloseSearch() {
  document.getElementById("eo-search-input").value = "";
  document.getElementById("eo-search-results").style.display = "none";
  document.getElementById("eo-search-close").style.display = "none";
  eoAllData = null;
}

// Close on outside click
document.addEventListener("click", function(e) {
  const wrapper = document.getElementById("eo-search-wrapper");
  if (wrapper && !wrapper.contains(e.target)) {
    document.getElementById("eo-search-results").style.display = "none";
  }
});

// Pre-load data on page load for instant results
window.addEventListener("load", () => setTimeout(eoLoadData, 1500));
</script>
```

---

## HOW TO ADD IT

1. Go to your Base44 app builder at https://app.base44.com
2. Open your Explore Omaha app
3. Find your global *Header* component (or the top of your Layout/Shell page)
4. Add a *Custom HTML* block as the FIRST element inside the header
5. Paste the entire code block above
6. Save and publish

## WHAT IT SEARCHES
- All 50 events in your OmahaFamilyEvent database (name, description, location, category)
- All articles in your Article database (title, content, excerpt, keywords)
- Results appear instantly as you type (after 2 characters)
- Shows event image thumbnails, category badge, and cost
- Clicking an article result navigates to the article page

## CUSTOMIZATION
- Change the navy header color: find `background:#1a3a5c` in the wrapper div
- Change number of results shown: find `.slice(0, 12)` and change 12
- Add more entities: duplicate the fetch block in `eoLoadData()` and add a new search loop
