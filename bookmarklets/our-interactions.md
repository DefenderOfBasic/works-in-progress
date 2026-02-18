Glitch is down, here's a copy of the bookmarklet (with my username embedded)

```
javascript:(function() {  let url = document.location.href;  url = url.split('?%27)[0].split(%27#')[0];  if (url.endsWith('/')) {     url = url.substring(0, url.length - 1);  }  let username = url.split('/').pop();  if (!username) { username = prompt("Failed to find username! Enter it manually:"); }  username = username.replace('@', '');  const searchUrl = "https://twitter.com/search?q=(from%3A%40DefenderOfBasic to%3A%40" + encodeURIComponent(username) + ") OR (to%3A%40DefenderOfBasic from%3A%40" + encodeURIComponent(username) + ")&src=typed_query&f=live";  window.open(searchUrl, "_self");})();
```


---

See: https://twitter-bookmarklet.glitch.me/
