+++
title = "Load Tests as Code"
outputs = ["Reveal"]
[reveal_hugo]
custom_theme = "reveal-hugo/themes/robot-lung.css"
margin = 0.2
highlight_theme = "color-brewer"
transition = "slide"
transition_speed = "fast"
+++


# Load tests as code
### An introduction to k6

Nicole van der Hoeven

---

# This is pretty cool

## h2

### h3

_Some italicized text_

```js
import http from 'k6/http';
import { sleep, check } from 'k6';

export default function () {
    let res = http.get('https://test.k6.io', {tags: { name: '01_Home' }});
    check(res, {
      'is status 200': (r) => r.status === 200,
      'text verification': (r) => r.body.includes("Collection of simple web-pages suitable for load testing")
    });
    sleep(Math.random() * 5);
}
```

---
