# Sharing and Loading Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the static invitation more robust for WeChat sharing and avoid loading the map API before it is needed.

**Architecture:** Keep the single-file static-site design. Open Graph data remains in the document head; map loading moves behind an `IntersectionObserver`; image dimensions become HTML metadata so browsers can reserve layout space.

**Tech Stack:** Static HTML, browser DOM APIs, Open Graph metadata, Node.js static syntax checks.

---

### Task 1: Define the failing static checks

**Files:**
- Modify: `index.html:7-19, 362-862, 885-1065`
- Test: one-off Python static validator run from repository root

- [x] **Step 1: Run the failing validator**

```bash
python3 - <<'PY'
from pathlib import Path
import re
html = Path('index.html').read_text(encoding='utf-8')
assert 'og:image:secure_url' in html
assert 'og:image:type' in html
assert not re.search(r'<script\\s+src="https://webapi\\.amap\\.com/maps\\?v=2\\.0', html)
assert 'width="1200" height="991"' in html
PY
```

Expected: fail because the existing page lacks the extra OG fields and intrinsic image dimensions.

### Task 2: Apply share and image metadata

**Files:**
- Modify: `index.html:7-19, 362-862`

- [x] **Step 1: Version and complete the OG image fields**

Use one HTTPS custom-domain URL with `?v=20260623` for `og:image` and `og:image:secure_url`, then add PNG MIME type and descriptive alt text.

- [x] **Step 2: Add intrinsic dimensions to every local image**

Use each file's actual width and height. Change only the first main couple photo to `loading="eager" fetchpriority="high"`; keep all remaining images lazy.

### Task 3: Delay the map API request

**Files:**
- Modify: `index.html:885-973`

- [x] **Step 1: Remove the parser-blocking external map script**

Keep `_AMapSecurityConfig` but do not fetch the AMap API in a static script tag.

- [x] **Step 2: Inject the AMap script once when the map is near the viewport**

Use an `IntersectionObserver` with a 400px bottom root margin; call `initMap` after a successful script load; fall back to immediate loading in browsers without the observer.

### Task 4: Verify the static site

**Files:**
- Test: `index.html`

- [x] **Step 1: Rerun the static validator**

Run the validator from Task 1 and expect no assertion failures.

- [x] **Step 2: Compile the final inline JavaScript**

```bash
node -e 'const fs=require("fs"); const h=fs.readFileSync("index.html","utf8"); const s=[...h.matchAll(/<script(?:\\s[^>]*)?>([\\s\\S]*?)<\\/script>/g)].map(m=>m[1]); new Function(s.at(-1)); console.log("OK");'
```

Expected: `OK`.

### Task 5: Record the validated change

**Files:**
- Modify: `index.html`
- Create: `docs/superpowers/specs/2026-06-23-sharing-and-loading-design.md`
- Create: `docs/superpowers/plans/2026-06-23-sharing-and-loading.md`

- [x] **Step 1: Commit the implementation and its design record**

```bash
git add index.html docs/superpowers
git commit -m "fix: improve sharing metadata and defer map loading"
```
