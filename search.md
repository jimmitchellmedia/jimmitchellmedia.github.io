---
---

<!-- Html Elements for Search -->
<div id="search-container">
<input type="search" id="search-input" placeholder="Search...">
<ul id="results-container" class="search-results"></ul>
</div>

<!-- Script pointing to search-script.js -->
<script src="/assets/js/search-script.js" type="text/javascript"></script>

<!-- Configuration -->
<script>
SimpleJekyllSearch({
  searchInput: document.getElementById('search-input'),
  resultsContainer: document.getElementById('results-container'),
  json: '/search.json'
})
</script>