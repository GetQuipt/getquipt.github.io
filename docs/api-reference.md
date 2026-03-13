---
title: API Reference
nav_order: 5
---

<div id="swagger"></div>

<link rel="stylesheet" href="https://unpkg.com/swagger-ui-dist/swagger-ui.css">
<link rel="stylesheet" href="{{ '/assets/css/custom.scss' | relative_url }}">

<script src="https://unpkg.com/swagger-ui-dist/swagger-ui-bundle.js"></script>
<script>
    SwaggerUIBundle({ url: 'openapi.yaml', dom_id: '#swagger', deepLinking: true, docExpansion: 'none', onComplete: () => {
        if (window.location.hash) {
          setTimeout(() => {
            const hash = decodeURIComponent(window.location.hash);
            const tagName = hash.startsWith('#/') ? hash.slice(2) : null;
            if (tagName) {
              const id = 'operations-tag-' + tagName.replace(/\s+/g, '_');
              const el = document.getElementById(id);
              if (el) el.scrollIntoView({ behavior: 'smooth' });
            }
          }, 500);
        }
      }
    });
</script>
