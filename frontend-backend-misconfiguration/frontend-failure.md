# Why the Frontend Was Failing

This document explains **why the frontend was showing 403 Forbidden** and why the browser displayed:

```
Backend says:
Error calling backend: TypeError: Failed to fetch
```

So, the **frontend never generated `index.html`**, causing nginx to serve a 403 Forbidden page.

---

# What I Saw

### When I opened:

```
http://localhost:30080
```

I got:

```
403 Forbidden
nginx/1.29.3
```

### Browser console showed:

```
Error calling backend: TypeError: Failed to fetch
```

---

# Root Cause (Frontend)

The frontend container **never created `index.html`**.

As a result, nginx returned **403 Forbidden**, which is the default behavior when:

* The directory exists
* But `index.html` does *not* exist

Here’s what caused it.

---

# Problem 1: `start.sh` was inside a read-only nginx directory

I mounted your ConfigMap into:

```
/usr/share/nginx/html/
```

This directory in the nginx image is **read-only**.

Your `start.sh` script tried to generate the HTML file:

```
sed ... > /usr/share/nginx/html/index.html
```

But nginx logged:

```
chmod: /usr/share/nginx/html/start.sh: Read-only file system
```

So:

* `index.html` was **never created**
* nginx had no index file → returned **403 Forbidden**
* Missing index.html
* Incorrect file permissions
* 403 on root path

---

# The Fix Applied

I corrected the directory structure so that nginx could actually serve files.

### Step 1: Move `start.sh` out of the read-only directory

Instead of putting it in `/usr/share/nginx/html/`, you changed it to:

```
/app/start.sh
```

This directory is writable.

### Step 2: Mount the template file properly

```
/usr/share/nginx/html/index.html.template
```

So nginx can serve from the same directory after generation.

### Step 3: Make `start.sh` runnable

* with proper file permissions

### Step 4: Change container command

```
command: ["/bin/sh", "/app/start.sh"]
```

This ensures the script actually runs.

### Step 5: Script generates the final index.html

`sed` now successfully writes into:

```
/usr/share/nginx/html/index.html
```

### Result

Nginx now has a valid index file → no more 403 Forbidden.

---

Frontend is now **fully working**.