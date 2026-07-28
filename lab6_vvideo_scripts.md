# Video Scripts 💀💀 All 8 Sections, Full Detail

**Group 4:** Harlin Kaur Taggar, Divyansh Ashishkumar Pandya, Sara Saleh, Pawan Koirala

**Format per section:** 
Assigned Member → 
        Video Title → 
              Objective → 
                Explanation → 
                    Technical Steps/Commands → 
                                  Testing Process → 
                                          Expected Results → 
                                                Screenshots to Capture → 
                                                              Full Speaking Script

---
---

# 🟥 SECTION 1 💀💀 Introduction & Developer Roles

**Assigned Member:** Divyansh Ashishkumar Pandya

**Video Title:** "Audiobookshelf Security Training: Introduction & Developer Roles"

**Type:** Theory | **Length:** 10–12 min

## Objective
Introduce the target application, its architecture, and which developer role owns which security responsibility, setting up context for the rest of the training.

## Explanation
Audiobookshelf is a free, open-source, self-hosted audiobook/podcast server, built on Node.js/Express (backend), Nuxt.js 2/Vue (frontend), SQLite via Sequelize ORM (24 models), Socket.io (realtime), and FFmpeg (transcoding), shipped as a Docker container on Alpine Linux. Understanding this stack matters because every threat discussed later in the training traces back to a specific layer here.

## Technical Steps / Commands
No live commands required for this theory section 💀💀 this is architecture + roles explanation. Optional: show the running container to ground the discussion.
```bash
docker ps
```
(Show `audiobookshelf` container running on port 13378 as the live example throughout.)

## Testing Process
N/A 💀💀 introductory/theory section.

## Expected Results
N/A.

## Screenshots to Capture
- Architecture/DFD diagram slide
- `docker ps` output showing the running container
- Developer roles table slide

## Full Speaking Script

> "Hi everyone, I'm Divyansh, and this is the first section of our security training for Audiobookshelf's development team.
>
> Audiobookshelf is a free, open-source, self-hosted audiobook and podcast server 💀💀 think of it as a privacy-respecting alternative to Audible that you run yourself. Under the hood, it's built on Node.js and Express for the backend, Nuxt.js 2 with Vue for the frontend, SQLite through the Sequelize ORM for data storage 💀💀 that's 24 different models 💀💀 Socket.io for real-time sync across devices, and FFmpeg for audio transcoding. The whole thing ships as a Docker container on an Alpine Linux base.
>
> Here's our container running right now 💀💀 you can see it on port 13378.
>
> [show docker ps]
>
> Architecturally, there are three external entities that talk to the server: the browser or PWA, the mobile app, and admin users. The server itself handles requests through Express and Socket.io, and it talks to three data stores: the SQLite database, the file system 💀💀 which holds audiobooks, podcasts, config, and metadata 💀💀 and a config.json file. It also spawns FFmpeg as a subprocess whenever it needs to transcode audio. And outbound, it reaches six external services: Google Books, Audible, Open Library, Audnexus, iTunes, and 💀💀 this one's important, we'll come back to it 💀💀 user-supplied RSS feed URLs for podcast subscriptions. That last one is the only fully attacker-controlled outbound destination in the entire system, and it's going to come up again in Section 3.
>
> Now, who's responsible for what, security-wise? Backend and API developers own input validation, authentication logic, and keeping dependencies clean. Frontend and Vue developers own output encoding 💀💀 making sure nothing gets rendered unsanitized. And DevOps or deployment folks own Docker hardening, secrets management, and setting up TLS through a reverse proxy.
>
> Based on our own assessment across Labs 1 through 3, we found three project-specific challenges worth flagging up front. First, this is a self-hosted, admin-managed deployment 💀💀 there's no forced TLS, and admins can actually disable the built-in SSRF filter if they want to. Second, that RSS feed feature I mentioned is the one fully user-controlled outbound request in the whole app. And third, FFmpeg runs as a native subprocess on every file that gets uploaded or transcoded, which matters because we found an unpatched CVE in the exact version Audiobookshelf ships.
>
> Quick interactive question for you: which developer role is primarily responsible for validating a user-supplied RSS feed URL before the server fetches it? A) Frontend developer, B) Backend or API developer, C) DevOps, or D) the end user.
>
> [pause]
>
> The answer is B 💀💀 backend or API developer. The reason is that the actual allowlist and validation logic has to live server-side, in the code path that calls the fetch. It can't be enforced by the frontend, and it can't be left to DevOps configuration alone 💀💀 this needs to be in the application logic itself.
>
> That's our intro. In the next sections, we'll go deep on the regulations that apply to this deployment, the specific threats we modeled, and two real mitigations we built and tested ourselves. See you in Section 2."

---
---

# 🟧 SECTION 2 💀💀 Applicable Regulations

**Assigned Member:** Harlin Kaur Taggar

**Video Title:** "Audiobookshelf Security Training: Applicable Regulations"

**Type:** Theory | **Length:** 10–12 min

## Objective
Identify which regulations apply to this deployment and translate each regulatory gap into a concrete developer action item.

## Explanation
From Lab 1/2 analysis: GDPR (if EU users), PIPEDA (applies directly 💀💀 this is a Canada/Ontario deployment), COPPA (if minors share a deployment), HIPAA (not applicable, no PHI). Several compliance gaps were identified during Lab 2's data classification exercise.

## Technical Steps / Commands
No live commands 💀💀 this is a regulatory/compliance explanation section. Optional: show the config.json location referenced in the gap discussion.
```bash
docker exec -it audiobookshelf ls /config
```

## Testing Process
N/A 💀💀 theory section.

## Expected Results
N/A.

## Screenshots to Capture
- Regulation → control mapping table slide
- Data classification table from Lab 2 (passwords, email, JWT secret, etc.)
- `docker exec` output showing the config directory (optional, for context)

## Full Speaking Script

> "Hi, I'm Harlin, and this section covers the regulations that apply to Audiobookshelf and what they actually mean for the people writing the code.
>
> Based on our Lab 1 and Lab 2 analysis, four regulations are relevant depending on deployment context. GDPR applies if there are EU users. PIPEDA applies directly to us, since this is a Canada, Ontario-based deployment. COPPA would apply if minors are sharing a deployment. And HIPAA doesn't apply at all 💀💀 there's no protected health information anywhere in this system.
>
> During our data classification work in Lab 2, we categorized exactly what data Audiobookshelf stores and how. Passwords are bcrypt-hashed 💀💀 that's fine. Email addresses are stored in plaintext and transmitted over HTTPS 💀💀 that's personally identifiable information, so it's confidential. Listening progress is internal-only data. JWT access tokens and refresh tokens are both restricted. And then there are two critical items: the JWT signing secret and the OIDC client secret, both stored in plaintext inside config.json.
>
> When we mapped these against actual regulatory requirements, we found several gaps. Right to access 💀💀 that's a gap, there's no self-service data export. Data portability 💀💀 also a gap, no JSON or CSV export exists. Breach notification 💀💀 gap, there's no automated detection mechanism at all. Encryption at rest 💀💀 gap, it relies entirely on whatever the host operating system provides, nothing built into the app. Access logging is partial 💀💀 there are basic HTTP logs, but no dedicated security audit log. The one thing that is fully compliant: data minimization and right to erasure 💀💀 no gap there.
>
> Here's why this matters for you as a developer, not just as a compliance checkbox: each of these gaps maps to an actual code or config change. The right-to-access gap means someone needs to build a `/api/me/export` endpoint. The encryption-at-rest gap means migrating SQLite to SQLCipher 💀💀 which, by the way, ties directly into the JWT secret problem, since both would go through the same kind of secrets-management fix.
>
> Quick task for you: match each regulation requirement to its Audiobookshelf gap. One 💀💀 Breach Notification. Two 💀💀 Encryption at Rest. Three 💀💀 Data Portability. Match to: A, no JSON or CSV export. B, no automated detection mechanism. C, SQLite stored unencrypted.
>
> [pause]
>
> The answer: One matches B, two matches C, three matches A.
>
> That covers the regulatory landscape. Next up, Section 4, where I'll walk through exactly how we tested this application against real attack techniques."

---
---

# 🟨 SECTION 3 💀💀 Key Threat Events

**Assigned Member:** Divyansh Ashishkumar Pandya

**Video Title:** "Audiobookshelf Security Training: Key Threat Events (STRIDE/PASTA)"

**Type:** Technical | **Length:** 14–15 min

## Objective
Present the full threat model 💀💀 STRIDE categories, trust boundaries, and the 3 critical findings 💀💀 using real analysis from Lab 2.

## Explanation
28 STRIDE threats identified across 6 trust boundaries. PASTA's 7-stage process was also applied. 3 Critical findings: SSRF via RSS feed (ST-11/13), plaintext JWT secret (ST-04/22), FFmpeg memory corruption CVE-2026-40962 (ST-14).

## Technical Steps / Commands
This is a modeling/analysis walkthrough, not a live exploit demo (per Lab 3's documented scope boundary 💀💀 these were NOT live-exploited). Show the risk matrix and attack scenario as diagrams/slides.

No destructive commands are run. If you want a **safe, illustrative** visual, you can show what a malicious feed URL entry would look like in the Add Podcast form field WITHOUT submitting it against any real infrastructure:
```
Example feed URL an attacker might try (DO NOT SUBMIT):
http://127.0.0.1:3000/api/users
```

## Testing Process
N/A for live exploitation (explicitly out of scope, per our own Lab 3 boundary). This section presents the *modeled* threats and *documented* attack scenario narrative only.

## Expected Results
N/A 💀💀 analytical section.

## Screenshots to Capture
- Trust boundary diagram (6 boundaries)
- STRIDE category summary table
- The 3 critical findings, one slide each
- Risk matrix (P1 rows)
- Attack scenario flow diagram (SSRF Internal Recon)

## Full Speaking Script

> "Welcome back, I'm Divyansh again for Section 3 💀💀 this is where we go deep into the actual threats we identified.
>
> We used two methodologies here: STRIDE, which gave us 28 individual threats across 6 trust boundaries, and PASTA's seven-stage risk-centric process.
>
> Let's start with the trust boundaries, because everything else builds on these. TB-01 is the client-to-server boundary over HTTP and WebSocket 💀💀 the risk there is man-in-the-middle attacks and credential interception. TB-02 is server to external metadata APIs 💀💀 risk of data leakage to third parties. TB-03 is server to RSS feeds, where the URL is fully user-controlled 💀💀 this is our highest-risk boundary. TB-04 is server to the FFmpeg subprocess 💀💀 command injection and memory corruption risk. TB-05 is the Docker container to host boundary 💀💀 path traversal and privilege escalation. And TB-06 is standard user to admin 💀💀 privilege escalation through role checks.
>
> Across those boundaries, our 28 STRIDE threats break down like this: 3 spoofing, 6 tampering, 3 repudiation, 7 information disclosure, 5 denial of service, and 4 elevation of privilege. The highest severities cluster in tampering, information disclosure, and elevation of privilege 💀💀 all rated Critical.
>
> Now, the three findings we flagged as Critical priority.
>
> Finding one: SSRF via RSS feed ingestion 💀💀 that's threat IDs ST-11 and ST-13, crossing trust boundary TB-03, mapped to CWE-918. Here's the mechanism: the Add Podcast feature lets any authenticated user supply an arbitrary URL as a feed source, and the server fetches it server-side. The built-in SSRF filter is admin-toggleable 💀💀 so it's not guaranteed to be on 💀💀 and on top of that, the axios version this app uses, 0.27.2, has known SSRF bypass vulnerabilities, including a NO_PROXY bypass and an IP-alias bypass. Put those together, and an attacker could point a podcast feed URL at something like the loopback address to hit internal APIs, or at a cloud metadata endpoint to try to steal cloud credentials.
>
> Finding two: the JWT signing secret, stored as plain text inside config.json 💀💀 that's ST-04 and ST-22, crossing TB-05, CWE-321 and CWE-312. Anyone who gets filesystem access to the container or host 💀💀 through path traversal, a stolen backup, or a misconfigured volume mount 💀💀 can read that secret and forge tokens with any role they want, including admin.
>
> Finding three: FFmpeg memory corruption, tracked as CVE-2026-40962, that's ST-14, crossing TB-04, CWE-787. This one's notable because FFmpeg is installed through Alpine's apk package manager, not through npm 💀💀 which means it's completely invisible to a standard npm audit. Every uploaded audio file gets passed through this subprocess for transcoding, and at the time of our assessment, there was no patch available.
>
> Let me walk through one attack scenario to make this concrete 💀💀 the SSRF internal recon scenario. An attacker logs in as a low-privilege standard user. They add a new podcast subscription with the feed URL pointed at the server's own internal API 💀💀 something like the loopback address on port 3000, hitting the user list endpoint. Because of the axios bypass, the SSRF filter doesn't catch it. The server fetches that URL and the response 💀💀 potentially containing the internal user list 💀💀 comes back through the podcast-add response. From there, the attacker could repeat the process targeting a cloud metadata endpoint to try to extract cloud provider credentials.
>
> On our risk matrix, all three of these landed in the same spot: Critical risk, Priority 1. SSRF is High likelihood, Critical impact. The JWT secret and FFmpeg findings are both Medium likelihood, Critical impact.
>
> Quick task: a user adds a podcast subscription with the feed URL pointing at a cloud metadata endpoint. What STRIDE category and CWE does this fall under, and which trust boundary is crossed?
>
> [pause]
>
> Answer: Information Disclosure, CWE-918 for SSRF, crossing TB-03 💀💀 the server-to-RSS-feed boundary. This maps directly to our ST-11 and ST-13 findings, the single highest-risk item across all 28 threats we identified.
>
> That's our threat model. In Section 4, Harlin's going to show you exactly how we tested this system against live attack techniques 💀💀 and where the line was between what we modeled here and what we actually exploited."

---
---

# 🟩 SECTION 4 💀💀 Cybersecurity Testing Strategies

**Assigned Member:** Harlin Kaur Taggar

**Video Title:** "Audiobookshelf Security Training: Testing Strategies & Live Pentest Walkthrough"

**Type:** Technical | **Length:** 13–15 min

## Objective
Explain manual vs. automated testing, risk-based prioritization, and walk through real test cases executed in Lab 3.

## Explanation
Testing prioritization must be both risk-based AND scope/safety-aware. P1-critical threats from Section 3 were deliberately NOT live-exploited (require infrastructure access or exploit crafting outside authorized scope); P2-level items were fully tested.

## Technical Steps / Commands

```bash
# Burp Suite Intruder 💀💀 search endpoint fuzzing (conceptual, show via UI/screenshot)
# Target: GET /api/libraries/{id}/search?q=<payload>
# Payload sets: 166 SQLi strings, 221 XSS strings, 13 path traversal sequences

# sqlmap deep SQLi test
sqlmap -u "http://192.168.40.20:13378/api/libraries/{id}/search?q=test" \
  --headers="Authorization: Bearer <token>" \
  --level=5 --risk=3

# Nikto web server scan
nikto -h http://192.168.40.20:13378
```

## Testing Process
Show the coverage table (what was tested vs. not, and why) and walk through 4–5 representative test cases from the TC01–TC19 set live or via screenshots of the original Burp/Nikto/sqlmap output.

## Expected Results
| Test | Expected | Actual |
|---|---|---|
| TC01 SQLi (166 payloads) | No DB errors | 200 OK, no injection 💀💀 Pass |
| TC04 sqlmap L5R3 | Not injectable | Confirmed not injectable 💀💀 Pass |
| TC08 Access token replay post-logout | 401 | **304 Not Modified (still valid) 💀💀 Partial, real finding** |
| TC14 Nikto scan | No critical findings | Missing X-Frame-Options, /config/ accessible 💀💀 Finding |

## Screenshots to Capture
- Manual vs automated tool table
- Coverage reality-check table (P1 not tested / P2 fully tested)
- Burp Intruder results grid (200 OK across all payloads)
- sqlmap terminal output (final "not injectable" line)
- Nikto scan summary (11 findings)
- TC08 result row highlighted

## Full Speaking Script

> "Hey, Harlin again for Section 4. Divyansh just walked you through what we found in the threat model 💀💀 now I'll show you how we actually tested for it.
>
> We used a mix of automated and manual testing. For automated dependency scanning, we used Syft and Grype back in Lab 1. For automated fuzzing, Burp Suite's Intruder tool. For deep, dedicated SQL injection testing, sqlmap. For a web server configuration scan, Nikto. And for manual testing 💀💀 session replay, JWT tampering, IDOR checks, error handling edge cases 💀💀 we used Burp's Repeater tool by hand.
>
> Here's the key principle behind our prioritization: we tested in the order of the Lab 2 risk matrix, but 💀💀 and this is important 💀💀 we deliberately did NOT live-exploit the three P1-critical theoretical findings. SSRF against real infrastructure, extracting the JWT secret from the filesystem, or crafting a working exploit for the FFmpeg CVE all require either infrastructure access or malicious file crafting that falls outside our authorized testing scope. Instead, we prioritized what we could safely and fully test: authentication flow, session management, injection testing, and error handling.
>
> Let me be transparent about coverage. SSRF via the RSS feed 💀💀 not tested live, the vulnerability lives in an outdated axios library that's still unpatched in the version we assessed. JWT secret extraction 💀💀 partial, we tested token handling and tampering extensively, but didn't attempt to actually read the secret off disk, since that needs host access. FFmpeg exploitation 💀💀 partial, we tested the upload endpoint's authentication and validation, but didn't craft an actual CVE exploit file. On the other hand, authentication endpoint weaknesses were fully tested, and path traversal was tested with 13 payloads.
>
> Why is this the right approach, and not a testing failure? Because a mature security assessment documents its boundaries. Attempting live SSRF against real infrastructure, or building a working memory-corruption exploit, carries genuine risk 💀💀 server crashes, unintended data exposure 💀💀 and needs separate authorization and a sandboxed environment. Reporting what wasn't tested, and why, is part of doing this responsibly.
>
> Let's look at a few actual results. We ran sqlmap against the search endpoint with level 5, risk 3 💀💀 that's the most aggressive setting 💀💀 testing boolean-blind, time-blind, error-based, and UNION-based injection techniques for over six minutes. Result: parameter not injectable, confirming Sequelize's parameterized queries are doing their job.
>
> [show sqlmap output]
>
> Nikto scanned 6,544 items in just 18 seconds and returned 11 findings 💀💀 the notable ones being a missing X-Frame-Options header, which is a clickjacking risk, and the /config/ directory being accessible when it shouldn't be.
>
> [show Nikto output]
>
> And here's our most interesting real finding 💀💀 test case TC08. We took a valid access token, logged out, and then replayed that same token against a protected endpoint. We expected a 401. What we actually got was the endpoint still responding 💀💀 the access token remained valid and usable for up to one hour after logout, because the JWT validation is purely stateless, with no server-side revocation list. The refresh token was correctly invalidated, which limits how bad this is, but it doesn't eliminate the exposure window entirely.
>
> Quick task for you: we identified three P1-critical threats in our model but only fully live-tested P2-level threats in Lab 3. Was this a testing failure? A) We ran out of time. B) P1 threats required infrastructure access or exploit crafting outside authorized scope, so they remain documented but not live-exploited. C) The tools couldn't detect them. D) They weren't real risks.
>
> [pause]
>
> The answer is B. Good testing strategy is risk-based AND safety-aware 💀💀 you don't just chase every theoretical finding into a live exploit without proper authorization and containment.
>
> That's our testing methodology. Coming up, Sara's going to show you a real mitigation we built for backup-endpoint abuse 💀💀 full implementation, not just theory."

---
---

# 🟦 SECTION 5 💀💀 Recommended Security Tools (Backup Rate Limiting)

**Assigned Member:** Sara Saleh

**Video Title:** "Audiobookshelf Security Training: Rate Limiting Backup Operations"

**Type:** Technical | **Length:** 14–15 min

## Objective
Demonstrate a live-implemented mitigation: proxy-level rate limiting specifically on backup operations, showing how many backups a user can trigger before being blocked, and why this matters.

## Explanation
Audiobookshelf supports automated daily backups and a manual "Backup Now" trigger (Settings → Backups). Like general HTTP traffic, this endpoint has no dedicated rate limiting 💀💀 a malicious or buggy client could trigger repeated backup operations, each of which reads/writes the full SQLite database and metadata to disk, causing disk I/O and CPU exhaustion. This is a config-only mitigation added via a reverse-proxy layer, no application JavaScript modified 💀💀 same pattern used in Section 6, applied to a different, more sensitive endpoint.

> **Before recording:** confirm the exact backup endpoint path for your installed version. Log into the Audiobookshelf web UI → Settings → Backups → click "Backup Now" → open browser DevTools → Network tab → find the outgoing request → copy the exact path (commonly `/api/backups` or similar 💀💀 confirm rather than assume, since paths can change between versions).

## Technical Steps / Commands

### 1. Identify the real endpoint
```bash
# With the Network tab open in your browser, trigger a manual backup and note the exact
# request method + path, e.g.:
# POST /api/backups
```

### 2. Add a dedicated, stricter rate-limiting zone in nginx.conf (same pattern as Section 6, different endpoint + stricter limit)
```nginx
http {
    # ... existing zones from Section 6 ...

    # Backups are expensive (full DB + metadata read/write) 💀💀 much stricter limit
    limit_req_zone $binary_remote_addr zone=backup_limit:10m rate=1r/m;
    limit_req_status 429;

    server {
        listen 8080;

        location /api/backups {
            limit_req zone=backup_limit burst=2 nodelay;
            proxy_pass http://127.0.0.1:80;
            proxy_set_header Host $host;
            proxy_set_header Authorization $http_authorization;
        }

        location / {
            # general rate limiting from Section 6 stays here for all other traffic
            proxy_pass http://127.0.0.1:80;
        }
    }
}
```
Explain: `rate=1r/m` = 1 request per minute sustained, `burst=2` allows 2 quick successive attempts before blocking 💀💀 appropriate for an operation that should realistically happen at most a few times a day, not repeatedly per minute.

### 3. Rebuild and redeploy (same image pipeline as Section 6)
```bash
docker build -t ghcr.io/pawkoi/audiobookshelf-ratelimited:latest .
docker push ghcr.io/pawkoi/audiobookshelf-ratelimited:latest
docker stop audiobookshelf-ratelimited && docker rm audiobookshelf-ratelimited
docker run -d --name audiobookshelf-ratelimited \
  -v <config volume>:/config -v <metadata volume>:/metadata -v <audiobooks volume>:/audiobooks \
  -p 13380:8080 \
  ghcr.io/pawkoi/audiobookshelf-ratelimited:latest
```

## Testing Process
```bash
# Get an auth token first (log in via API or browser, copy Bearer token)

# Test against ORIGINAL (13378) 💀💀 no rate limit, all attempts should succeed
for i in $(seq 1 10); do
  curl -s -o /dev/null -w "%{http_code}\n" -X POST \
    -H "Authorization: Bearer <token>" \
    http://localhost:13378/api/backups
  sleep 1
done

# Test against RATE-LIMITED (13380) 💀💀 should block after 2 quick attempts
for i in $(seq 1 10); do
  curl -s -o /dev/null -w "%{http_code}\n" -X POST \
    -H "Authorization: Bearer <token>" \
    http://localhost:13380/api/backups
  sleep 1
done
```

## Expected Results
| Target | Attempts | Expected outcome |
|---|---|---|
| Original (13378) | 10 rapid backup triggers | All succeed 💀💀 no protection against repeated triggering |
| Rate-limited (13380) | 10 rapid backup triggers | First 2 succeed (burst), remainder return `429` until the 1-per-minute window allows another |

## Screenshots to Capture
- Browser DevTools Network tab showing the real backup endpoint request
- nginx.conf diff showing the new `/api/backups` location block
- Terminal output: original container 💀💀 all 200s
- Terminal output: rate-limited container 💀💀 200s then 429s
- `docker stats` snapshot showing disk/CPU impact difference (optional, if capturable)

## Full Speaking Script

> "Hi, I'm Sara. In this section I'm going to show you a real mitigation I built and tested myself 💀💀 rate limiting specifically on Audiobookshelf's backup feature.
>
> Here's the problem: Audiobookshelf lets any user with the right permission trigger a manual backup through Settings, and there's also a scheduled automated backup option. Each backup operation reads the entire SQLite database and metadata directory and writes a full backup file to disk. That's expensive 💀💀 disk I/O and CPU both spike. And just like the general HTTP traffic we looked at in Pawan's section, there's no rate limiting on this specific endpoint. If someone scripted repeated backup triggers 💀💀 whether maliciously or just by accident, like a broken automation 💀💀 they could cause real resource exhaustion on the server.
>
> First, I needed to find the exact endpoint. I opened the Audiobookshelf web UI, went to Settings, Backups, clicked 'Backup Now,' and watched the Network tab in my browser's developer tools to capture the exact request being made.
>
> [show DevTools capture]
>
> Once I had that, I added a dedicated rate-limiting zone in the same nginx config we're using for general HTTP protection, but scoped specifically to this endpoint and much stricter. I set it to 1 request per minute sustained, with a burst of 2 💀💀 meaning someone gets 2 quick attempts before they're blocked, and then have to wait for the window to refill. That matches how this feature should realistically be used 💀💀 a few times a day at most, not repeatedly within seconds.
>
> [show nginx.conf edit]
>
> I rebuilt the image using the same pipeline Pawan set up 💀💀 build, push to our GHCR registry, pull, and redeploy on the same rate-limited container on port 13380.
>
> Now let's test it. Against the original container on 13378, with no protection, I'm firing 10 backup-trigger requests one per second.
>
> [show terminal 💀💀 all 200s]
>
> Every single one succeeds. No protection at all.
>
> Now the same test against our rate-limited container on 13380.
>
> [show terminal 💀💀 200s then 429s]
>
> The first couple go through 💀💀 that's the burst allowance 💀💀 and then everything after that gets rejected with a 429, until the one-per-minute window opens back up.
>
> This is a config-only fix, same as Pawan's 💀💀 we didn't touch a single line of Audiobookshelf's actual JavaScript. It's entirely enforced at the reverse-proxy layer in front of the unmodified application.
>
> Quick task for you: why should the backup endpoint have a stricter rate limit than general page traffic 💀💀 like the homepage 💀💀 even though both sit behind the same nginx layer?
>
> [pause]
>
> Because the cost per request is completely different. A homepage request is nearly free for the server to handle. A backup operation touches the entire database and file system 💀💀 so even a small number of repeated triggers can do real damage, which is why it needs its own, much tighter limit rather than sharing the general traffic threshold.
>
> Next up, Pawan's going to walk you through the general HTTP rate limiting implementation in more depth, including the full Docker and GHCR pipeline."

---
---

# 🟪 SECTION 6 💀💀 Secure Coding Guidelines (HTTP Rate Limiting)

**Assigned Member:** Pawan Koirala

**Video Title:** "Audiobookshelf Security Training: Implementing HTTP Rate Limiting via Docker/GHCR"

**Type:** Technical | **Length:** 14–15 min

## Objective
Demonstrate the full end-to-end process of identifying a real gap (no global HTTP rate limiting), implementing a config-only fix, building/publishing a custom Docker image, and proving the fix works via load testing and CPU comparison.

## Explanation
From Lab 1/2: `/login` has rate limiting (40 req/600s via express-rate-limit), but no rate limiting exists anywhere else in the app 💀💀 this is the exact CPU-exhaustion risk flagged by the body-parser vulnerability finding (GHSA-qwcr-r2fm-qrc7). Since app JavaScript cannot be modified, the fix is implemented entirely at the infrastructure layer: an nginx reverse proxy added on top of the unmodified official image.

## Technical Steps / Commands

### 1. The gap (show existing container)
```bash
docker ps
# b32d64b67fba   ghcr.io/advplyr/audiobookshelf:latest   "tini -- node index.…"   ...   0.0.0.0:13378->80/tcp   audiobookshelf
```

### 2. Build directory + files
```bash
mkdir -p ~/audiobookshelf-ratelimit && cd ~/audiobookshelf-ratelimit
```

**Dockerfile:**
```dockerfile
FROM ghcr.io/advplyr/audiobookshelf:latest

RUN apk add --no-cache nginx

COPY nginx.conf /etc/nginx/nginx.conf
COPY start.sh /start.sh
RUN chmod +x /start.sh

EXPOSE 8080
ENTRYPOINT ["/start.sh"]
```

**nginx.conf:**
```nginx
worker_processes 1;

events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    sendfile      on;

    limit_req_zone $binary_remote_addr zone=general_limit:10m rate=10r/s;
    limit_req_status 429;

    server {
        listen 8080;
        limit_req zone=general_limit burst=20 nodelay;

        location / {
            proxy_pass         http://127.0.0.1:80;
            proxy_http_version 1.1;
            proxy_set_header   Host $host;
            proxy_set_header   X-Real-IP $remote_addr;
            proxy_set_header   X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header   Upgrade $http_upgrade;
            proxy_set_header   Connection "upgrade";
        }
    }
}
```

**start.sh** (shell script only 💀💀 no app JS touched):
```bash
#!/bin/sh
set -e
tini -- node index.js &
sleep 2
nginx -g "daemon off;"
```

### 3. Build, tag, push, pull, run
```bash
# Build
docker build -t ghcr.io/pawkoi/audiobookshelf-ratelimited:latest .

# Login to own GHCR namespace (personal access token with write:packages scope)
docker login ghcr.io -u pawkoi

# Push
docker push ghcr.io/pawkoi/audiobookshelf-ratelimited:latest

# Pull (proves round-trip through the registry)
docker pull ghcr.io/pawkoi/audiobookshelf-ratelimited:latest

# Find original container's volumes so the new one uses the same data
docker inspect audiobookshelf --format='{{range .Mounts}}{{.Source}} -> {{.Destination}}{{"\n"}}{{end}}'

# Run as a second container on a new port 💀💀 original stays untouched
docker run -d --name audiobookshelf-ratelimited \
  -v <config volume>:/config \
  -v <metadata volume>:/metadata \
  -v <audiobooks volume>:/audiobooks \
  -p 13380:8080 \
  ghcr.io/pawkoi/audiobookshelf-ratelimited:latest
```

## Testing Process

**Basic comparison:**
```bash
echo "=== ORIGINAL (13378) 💀💀 no rate limit ==="
for i in $(seq 1 50); do curl -s -o /dev/null -w "%{http_code}\n" http://localhost:13378/; done | sort | uniq -c

echo "=== NEW (13380) 💀💀 rate limited ==="
for i in $(seq 1 50); do curl -s -o /dev/null -w "%{http_code}\n" http://localhost:13380/; done | sort | uniq -c
```

**Paced traffic (proves legitimate use is unaffected):**
```bash
for i in $(seq 1 50); do curl -s -o /dev/null -w "%{http_code}\n" http://localhost:13380/; sleep 0.1; done | sort | uniq -c
```

**High-volume load test:**
```bash
ab -n 200 -c 20 http://localhost:13378/
ab -n 200 -c 20 http://localhost:13380/
```

**Sustained load + CPU comparison:**
```bash
# Terminal 1 💀💀 log CPU for original
while true; do echo "$(date +%T),$(docker stats audiobookshelf --no-stream --format '{{.CPUPerc}}')" >> old_cpu_log.csv; sleep 1; done

# Terminal 2 💀💀 log CPU for rate-limited
while true; do echo "$(date +%T),$(docker stats audiobookshelf-ratelimited --no-stream --format '{{.CPUPerc}}')" >> new_cpu_log.csv; sleep 1; done

# Terminal 3 💀💀 sustained attack, 30s, 200 concurrent connections
wrk -t4 -c200 -d30s http://localhost:13378/
wrk -t4 -c200 -d30s http://localhost:13380/
```

## Expected Results
| Test | Original (13378) | Rate-limited (13380) |
|---|---|---|
| 50-request burst | 50/50 → 200 | ~26-28/50 → 200, rest → 429 |
| 50-request paced (0.1s apart) | 50/50 → 200 | 50/50 → 200 (legitimate traffic unaffected) |
| 1000-request burst | ~1000/1000 → 200 | ~157-160/1000 → 200, ~840-843 → 429 |
| CPU under sustained load | Sustained ~25-26% | ~10%, stays flat |

## Screenshots to Capture
- `docker ps` showing original container only (before)
- Dockerfile, nginx.conf, start.sh content
- Successful `docker push` output (digest line)
- GHCR package page on github.com showing the pushed image
- `docker ps` showing both containers running side by side
- Terminal: 50-request burst comparison (200s vs 200s+429s)
- Terminal: paced request test (all 200s on rate-limited container)
- `ab` output comparison (Non-2xx responses line)
- CPU log/graph comparing old vs new under sustained `wrk` load

## Full Speaking Script

> "Hey everyone, I'm Pawan. In this section I'm implementing and testing a real fix for a gap we identified back in Lab 1 and Lab 2 💀💀 the lack of general HTTP rate limiting.
>
> Here's the gap: Audiobookshelf does have rate limiting on the login endpoint 💀💀 40 requests per 600 seconds 💀💀 but nowhere else. That ties directly to a vulnerability we found in Lab 1's SBOM scan: the body-parser package has a known issue where a malformed request can cause CPU exhaustion, and with no rate limiting anywhere else in the app, there's nothing stopping someone from just flooding the server with requests.
>
> Here's the constraint I'm working under: I don't own this codebase, and I'm not allowed to modify the application's JavaScript at all. So the entire fix has to live at the infrastructure layer.
>
> My approach: build a new Docker image that layers nginx on top of the official, completely unmodified Audiobookshelf image. Nginx sits in front as a reverse proxy and enforces rate limiting before requests even reach the Node.js application.
>
> Here's my original container, untouched, running on port 13378 💀💀 this stays exactly as it is for the entire video, as our baseline.
>
> [show docker ps]
>
> I built three files. The Dockerfile starts from the official image, installs nginx via apk, and copies in a config file and a startup script.
>
> [show Dockerfile]
>
> The nginx config is where the actual rate limiting logic lives 💀💀 this is pure configuration, zero application code. I defined a rate-limiting zone keyed by client IP, allowing 10 requests per second sustained, with a burst allowance of 20 requests before anything gets rejected with a 429 status.
>
> [show nginx.conf]
>
> And the startup script 💀💀 this is a shell script, not JavaScript 💀💀 it starts the original, completely unmodified app in the background exactly the way the official image normally does, waits a couple seconds for it to bind, then starts nginx in front of it.
>
> [show start.sh]
>
> Now let's build this. I'm tagging it under my own GitHub Container Registry namespace 💀💀 this has nothing to do with the original project's repository, it's entirely my own image, just built on top of their public base image as a starting layer.
>
> [run docker build]
>
> Once it built successfully, I logged into my own GHCR account and pushed it.
>
> [run docker push, show the digest confirmation]
>
> Here it is live on GitHub, under my account's packages tab.
>
> [show GHCR package page]
>
> I pulled it back down to confirm the round trip worked, then ran it as a second, completely separate container, on a new port 💀💀 13380 💀💀 using the exact same data volumes as the original, so it's working with identical data.
>
> [show docker run, then docker ps showing both containers]
>
> Now the actual testing. First, a simple burst 💀💀 50 requests fired as fast as possible against each container.
>
> [show terminal 💀💀 original: all 200s]
>
> Original container: every single request succeeds. Zero protection.
>
> [show terminal 💀💀 new: mix of 200 and 429]
>
> New container: about half succeed, the rest get rejected with 429 💀💀 Too Many Requests. That's nginx enforcing the limit.
>
> But here's the important part 💀💀 I also tested what happens with normal, realistic traffic. If I pace my requests at roughly the allowed rate 💀💀 one every tenth of a second, which matches our 10 requests per second limit 💀💀
>
> [show terminal 💀💀 all 200s even on rate-limited container]
>
> 💀💀 every single request succeeds. This proves the fix isn't just blindly blocking traffic; it specifically targets abusive bursts while leaving normal usage completely untouched.
>
> For a heavier, more realistic test, I used Apache Bench with 200 requests at 20 concurrent connections.
>
> [show ab output 💀💀 Non-2xx responses line]
>
> Look at the 'Non-2xx responses' line 💀💀 zero on the original, but a large chunk rejected on the rate-limited version.
>
> And finally, the metric that matters most for a real DoS scenario 💀💀 CPU usage under sustained load. I logged CPU for both containers every second while running a 30-second sustained attack with 200 concurrent connections using wrk.
>
> [show CPU comparison 💀💀 original spikes and stays elevated, new stays flat and low]
>
> The original container's CPU climbs and stays elevated the entire attack 💀💀 around 25 to 26 percent. The rate-limited container barely moves, staying around 10 percent, because nginx is rejecting the excess load cheaply, before it ever reaches the actual Node.js application and has to do real work.
>
> Quick task for you: looking at this code 💀💀
>
> ```
> limit_req_zone $binary_remote_addr zone=general_limit:10m rate=10r/s;
> limit_req zone=general_limit burst=20 nodelay;
> ```
>
> 💀💀 what does the 'nodelay' keyword actually do, and what would change if we removed it?
>
> [pause]
>
> With nodelay, excess requests inside the burst window are processed immediately, and only requests beyond the full burst capacity get rejected outright. Without nodelay, excess requests would instead be queued and released gradually at the steady rate 💀💀 meaning slower responses instead of instant rejections, which is a smoother experience but takes longer to actually block a flood.
>
> That's the full implementation 💀💀 identified gap, config-only fix, built and published as a real Docker image, and proven with both HTTP-level and CPU-level evidence. Up next, I'll compare Audiobookshelf's overall security posture against similar self-hosted projects."

---
---

# 🟫 SECTION 7 💀💀 Comparison with Alternative Applications

**Assigned Member:** Pawan Koirala

**Video Title:** "Audiobookshelf Security Training: Comparison with Alternative Applications"

**Type:** Theory | **Length:** 10–12 min

## Objective
Compare Audiobookshelf's security posture, architecture, and rate-limiting approach against Kavita and Booksonic.

## Explanation
From Lab 1's competitive landscape: Booksonic (declining, legacy Subsonic-based) vs Kavita (actively maintained, ASP.NET Core, broader media focus). Extend this comparison specifically into rate-limiting/SSRF design choices, tying back to the mitigation just demonstrated in Section 6.

## Technical Steps / Commands
No live commands 💀💀 comparative analysis section. Optional: show each project's public GitHub repo/README side by side as visual reference.

## Testing Process
N/A 💀💀 theory/comparison section.

## Expected Results
N/A.

## Screenshots to Capture
- Architecture comparison table slide
- Threat exposure comparison slide
- Mitigation strategy differences slide

## Full Speaking Script

> "Hi, Pawan again. In this last technical-adjacent section, I want to zoom out and compare Audiobookshelf against two similar self-hosted projects: Kavita and Booksonic.
>
> Booksonic is built on a legacy, Subsonic-based Java stack, and it's in decline 💀💀 not actively maintained the way Audiobookshelf is. Kavita, on the other hand, is actively maintained, built on ASP.NET Core, and has a broader scope 💀💀 it handles comics and manga in addition to books, not just audio and podcasts like Audiobookshelf.
>
> On architecture: Audiobookshelf uses Node and Express with Nuxt and Vue on the frontend and SQLite for data. Kavita uses ASP.NET Core end to end. Both are actively maintained and both support OIDC for authentication. Where they really differ is in SSRF-relevant surface area 💀💀 Audiobookshelf's RSS feed ingestion, which we spent Section 3 covering, is a fairly unique risk tied to its podcast-focused design. Kavita has an analogous external metadata-fetching surface, but implemented differently.
>
> On threat exposure: Audiobookshelf's narrower feature set 💀💀 just audio and podcasts 💀💀 actually means a smaller overall attack surface than Kavita, which handles more media types and therefore more parsers, more format-specific code paths, more places things can go wrong. But the RSS SSRF vector we identified is fairly specific to Audiobookshelf's design choices.
>
> Here's the part that connects directly back to what I just demonstrated in Section 6. Kavita's more modern .NET stack has some SSRF protections built into its default HttpClient behavior 💀💀 an attacker would need to deliberately work around defaults that lean toward safety. Audiobookshelf's approach, using axios, makes the SSRF filter an explicit, admin-toggleable setting 💀💀 which shifts the responsibility onto configuration rather than enforcing it unconditionally in code. That's exactly the pattern I fixed with the nginx rate-limiting layer in Section 6, just applied to a different problem 💀💀 instead of relying on an app-level toggle that could be disabled, the protection lives at the infrastructure layer where it can't be turned off by an admin setting.
>
> Neither Kavita nor Audiobookshelf, as far as our research showed, implements a full server-side JWT revocation list by default 💀💀 which connects back to Harlin's TC08 finding in Section 4, the post-logout token validity gap. That's not a one-off mistake specific to Audiobookshelf; it's a common tradeoff across stateless-JWT architectures industry-wide.
>
> Quick task: given that neither project implements full token revocation by default, and Audiobookshelf's SSRF filter is admin-toggleable while Kavita's protections are closer to default-on, which design pattern would you recommend to a new self-hosted project starting from scratch, and why?
>
> [pause]
>
> The stronger pattern is: security-relevant defaults should not be optional or toggleable by an admin 💀💀 they should be hardcoded and unconditional, the same fix I demonstrated in Section 6. Making critical protections a configuration option, however well-intentioned, creates exactly the kind of gap we found with the SSRF filter.
>
> That wraps up the comparison. Sara's got our final section 💀💀 pulling all of this together into one developer checklist."

---
---

# ⬜ SECTION 8 💀💀 Wrap-Up & Developer Checklist

**Assigned Member:** Sara Saleh

**Video Title:** "Audiobookshelf Security Training: Wrap-Up & Developer Checklist"

**Type:** Theory | **Length:** 8–10 min

## Objective
Recap all 7 prior sections and deliver a final, actionable developer checklist.

## Explanation
Synthesis section 💀💀 collect 1 key takeaway and 1 checklist item from each of the other 3 teammates before recording.

## Technical Steps / Commands
No commands 💀💀 recap/synthesis section.

## Testing Process
N/A.

## Expected Results
N/A.

## Screenshots to Capture
- Key takeaways recap slide (1 line per section)
- Final developer checklist slide

## Full Speaking Script

> "Hi, Sara again, wrapping up our training with a recap and a final checklist.
>
> Quick run-through of everything we covered. Section 1: Audiobookshelf's architecture centers on six trust boundaries, and the RSS feed ingestion path is the single highest-risk surface in the whole system. Section 2: PIPEDA applies directly to our deployment, and we found real gaps in breach notification and encryption at rest. Section 3: three Critical threats 💀💀 SSRF through RSS feeds, a plaintext JWT secret, and an unpatched FFmpeg memory corruption CVE. Section 4: good testing strategy has to be both risk-based and scope-aware 💀💀 we didn't chase every theoretical finding into a live exploit without proper authorization, and along the way we found one real, confirmed gap 💀💀 access tokens staying valid up to an hour after logout. Section 5, which I covered: we built and tested real rate limiting specifically on the backup endpoint, since backup operations are expensive and were completely unprotected. Section 6, Pawan's deep dive: a full working HTTP rate-limiting implementation, built as a Docker image, published to a real container registry, and proven with both request-level and CPU-level evidence. Section 7: comparable projects like Kavita and Booksonic share similar architectural tradeoffs 💀💀 this isn't a uniquely Audiobookshelf problem, it's a pattern worth watching for in general.
>
> Now, the checklist 💀💀 this is what we'd hand directly to the Audiobookshelf development team.
>
> Never make security filters, like the SSRF filter, admin-toggleable 💀💀 hardcode the check so it can't be turned off by configuration.
>
> Never store signing secrets in plaintext config files 💀💀 use environment variables or a proper secrets manager.
>
> Implement server-side token revocation for logout, not just refresh-token invalidation, to close the gap we confirmed in testing.
>
> Run SBOM-based scanning 💀💀 tools like Syft and Grype 💀💀 not just npm audit, since that's the only way to catch OS-level package vulnerabilities like the FFmpeg issue we found.
>
> Validate and timeout all subprocess calls to native binaries.
>
> Add general HTTP rate limiting at the infrastructure layer, and add stricter, endpoint-specific limits on expensive operations like backups 💀💀 exactly what Pawan and I demonstrated live in this training.
>
> Set proper security headers like X-Frame-Options, and restrict access to sensitive directories like /config/ at the web server layer.
>
> Enforce HTTPS by default instead of relying on an optional reverse proxy setup.
>
> And finally 💀💀 document explicitly what wasn't tested, and why, in every security assessment. That transparency is what makes an assessment trustworthy.
>
> One last thing to think about: if you were the Audiobookshelf maintainer, which single item on this checklist would you implement first, and why? There's no single right answer here 💀💀 it's worth thinking about which fix gives the most protection for the least effort.
>
> That's our full training module. Thanks for watching."

---
---

# Quick Reference 💀💀 All Commands Used Across the Training

```bash
# --- Section 1 (context) ---
docker ps

# --- Section 4 (Harlin) ---
sqlmap -u "http://192.168.40.20:13378/api/libraries/{id}/search?q=test" \
  --headers="Authorization: Bearer <token>" --level=5 --risk=3
nikto -h http://192.168.40.20:13378

# --- Section 5 (Sara) ---
# nginx location block for /api/backups with limit_req_zone rate=1r/m, burst=2
docker build -t ghcr.io/pawkoi/audiobookshelf-ratelimited:latest .
docker push ghcr.io/pawkoi/audiobookshelf-ratelimited:latest

# --- Section 6 (Pawan) ---
docker build -t ghcr.io/pawkoi/audiobookshelf-ratelimited:latest .
docker login ghcr.io -u pawkoi
docker push ghcr.io/pawkoi/audiobookshelf-ratelimited:latest
docker pull ghcr.io/pawkoi/audiobookshelf-ratelimited:latest
docker inspect audiobookshelf --format='{{range .Mounts}}{{.Source}} -> {{.Destination}}{{"\n"}}{{end}}'
docker run -d --name audiobookshelf-ratelimited -v <vols> -p 13380:8080 ghcr.io/pawkoi/audiobookshelf-ratelimited:latest
for i in $(seq 1 50); do curl -s -o /dev/null -w "%{http_code}\n" http://localhost:13378/; done | sort | uniq -c
for i in $(seq 1 50); do curl -s -o /dev/null -w "%{http_code}\n" http://localhost:13380/; done | sort | uniq -c
ab -n 200 -c 20 http://localhost:13378/
ab -n 200 -c 20 http://localhost:13380/
wrk -t4 -c200 -d30s http://localhost:13378/
wrk -t4 -c200 -d30s http://localhost:13380/
while true; do echo "$(date +%T),$(docker stats audiobookshelf --no-stream --format '{{.CPUPerc}}')" >> old_cpu_log.csv; sleep 1; done
```

# Accuracy Guardrails 💀💀 Read Before Recording (applies to all 8 videos)
- Do **not** claim SSRF, JWT secret extraction, or the FFmpeg CVE were "exploited" 💀💀 they were identified via threat modeling/SBOM only, explicitly not live-tested. Say "identified but not exploited, due to scope and safety constraints."
- Do **not** call the cross-user DELETE behavior (Lab 3) a vulnerability 💀💀 the developer confirmed it's intentional shared-library design.
- **Confirm the real backup endpoint path** before recording Section 5 💀💀 don't assume `/api/backups` without checking DevTools yourself.
- Be honest that the rate-limiting fixes (Sections 5 and 6) are infrastructure-layer mitigations, not upstream code fixes 💀💀 they protect *your* deployment but don't change the underlying Audiobookshelf source.
