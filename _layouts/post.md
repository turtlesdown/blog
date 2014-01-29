---
layout: default
---
<article>
  <title>{{ .page.title }}</title>
  <div class="date-stamp">
    <time>{{ date_to_string .page.date }}</time>
  </div>
  {{ .content }}
</article>