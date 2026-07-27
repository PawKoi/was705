# Lab 6 - Exact Execution Guide (Using Real Lab 1–3 Data)

**Group 4:** Person 1, Person 2, Person 3, Person 4

**Target:** Audiobookshelf v2.35.0 | Course: WAS705NBB
---

> This document tells each person **exactly** what content to put on which slide, which command to run/show, which real finding from Labs 1–3 to cite, and what the interactive task + solution should say - word for word where possible. Copy-paste and adapt tone, don't reinvent.

---
---

## 0. Section-to-Person Assignment (using real names)

| Person | Sections | Source Lab(s) |
|---|---|---|
| **Person 1** | 1. Intro & Developer Roles, 3. Key Threat Events | Lab 1 (overview) + Lab 2 (STRIDE/PASTA) |
| **Person 4** | 2. Applicable Regulations, 5. Recommended Security Tools | Lab 2 (regulatory gaps) + Lab 1/3 (Syft/Grype, Burp, Nikto, sqlmap) |
| **Person 3** | 6. Secure Coding Guidelines, 8. Wrap-Up & Checklist | Lab 2 (mitigations table) + Lab 3 (test results) |
| **Person 2** | 4. Testing Strategies, 7. Comparison with Alternatives | Lab 3 (test methodology) + Lab 1 (competitive landscape) |

*(Swap freely - the point is every section below is fully scripted regardless of who presents it.)*

---
---

## 1. SECTION 1 - Introduction & Developer Roles (Person 1, ~12 min)

### Slide 1: Title + Agenda
- Title: "Audiobookshelf Developer Security Training"
- Subtitle: Group 4 - WAS705NBB
- Agenda: list all 8 section titles

### Slide 2: What is Audiobookshelf
Say exactly this, in your own words:
> Audiobookshelf is a free, open-source, self-hosted audiobook and podcast server - a privacy-respecting alternative to Audible. It's built on Node.js/Express on the backend, Nuxt.js 2 (Vue) on the frontend, with SQLite via Sequelize ORM (24 models) for data, Socket.io for realtime sync, and FFmpeg for audio transcoding, all shipped as a Docker container on Alpine Linux.

### Slide 3: Architecture Diagram
Recreate the DFD from Lab 1 §Architecture:
- 3 external entities: Browser/PWA, Mobile App, Admin User
- Central node: Audiobookshelf Server (Express/Node/Socket.io)
- 3 data stores: SQLite DB, File System (/audiobooks, /podcasts, /config, /metadata), config.json
- FFmpeg subprocess (transcoding)
- 6 outbound HTTPS destinations: Google Books, Audible, Open Library, Audnexus API, iTunes API, **user-supplied RSS feed URLs** (flag this one - it's the attacker-controlled one, sets up Section 3)

### Slide 4: Developer Roles & Responsibilities
| Role | Security Responsibility |
|---|---|
| Backend/API dev | Input validation, auth logic, dependency hygiene |
| Frontend/Vue dev | Output encoding, avoiding XSS via unsanitized rendering |
| DevOps/deployment | Docker hardening, secrets management, TLS/reverse proxy setup |

### Slide 5: Project-Specific Security Challenges (pull directly from your own findings)
1. **Self-hosted, admin-managed deployment** - no forced TLS, admins can disable the SSRF filter (our own Lab 2 finding, ST-23)
2. **User-controlled outbound requests** - RSS feed "Add Podcast" feature is the only fully attacker-controlled outbound destination in the entire DFD
3. **Native subprocess invocation** - FFmpeg runs as a subprocess on every uploaded/transcoded file (CVE-2026-40962, unpatched at assessment time)

### Interactive Task (Slide 6)
**Task:** "Which developer role is primarily responsible for validating a user-supplied RSS feed URL before the server fetches it?"
Options: A) Frontend dev  B) Backend/API dev  C) DevOps  D) End user

**Solution (Slide 7):** B) Backend/API dev - because the SSRF filter and URL allowlist logic (recommended in Lab 2, ST-11/13 mitigation) must live server-side in the code that calls the fetch; it cannot be enforced client-side or by DevOps config alone.

### Narration script skeleton
1. (2 min) What Audiobookshelf is, why it matters
2. (3 min) Walk through architecture diagram
3. (3 min) Developer roles table
4. (3 min) Our 3 project-specific challenges, tie each to a later section ("we'll go deep on RSS/SSRF in Section 3")
5. (2 min) Interactive task + solution

---
---

## 2. SECTION 2 - Applicable Regulations (Person 4, ~12 min)

### Slide 1: Regulatory Landscape for Audiobookshelf
State exactly from Lab 1: GDPR (if EU users), **PIPEDA (applies - this is a Canada/Ontario deployment)**, COPPA (if minors share a deployment), HIPAA (not applicable - no PHI handled).

### Slide 2: Regulation → Control Mapping Table
| Regulation | Requirement | Audiobookshelf Control Status |
|---|---|---|
| GDPR Art. 32 / PIPEDA 4.7 | Security of processing (encryption, access control) | **Gap** - SQLite unencrypted at rest, JWT secret plaintext in config.json |
| GDPR Right to Access | User can request their data | **Gap** - no self-service export |
| GDPR Data Portability | User can export data in machine-readable form | **Gap** - no JSON/CSV export |
| GDPR/PIPEDA Breach Notification | Notify on breach within required window | **Gap** - no automated detection mechanism |
| PIPEDA 4.1.4 | Accountability / audit trail | **Partial** - basic HTTP logs only, no dedicated security audit log |
| Data Minimization / Right to Erasure | Don't over-collect; allow deletion | **No gap** - confirmed compliant |

### Slide 3: Why This Matters for Developers
Explain: these aren't abstract legal requirements - each unchecked gap maps directly to a real code/config change a developer would need to make (e.g., "Right to Access" gap = someone needs to build a `/api/me/export` endpoint; "encryption at rest" gap = migrate SQLite to SQLCipher, per Lab 2's ST-19 mitigation).

### Interactive Task (Slide 4)
**Task:** "Match the regulation requirement to the Audiobookshelf gap: (1) Breach Notification, (2) Encryption at Rest, (3) Data Portability - Match to: (a) No JSON/CSV export, (b) No automated detection mechanism, (c) SQLite stored unencrypted"

**Solution (Slide 5):** 1→b, 2→c, 3→a

### Narration script skeleton
1. (2 min) Which regulations apply and why (PIPEDA is the real one for our deployment)
2. (4 min) Walk the gap table row by row
3. (3 min) Translate 2 gaps into concrete dev tasks
4. (3 min) Interactive task

---
---

## 3. SECTION 3 - Key Threat Events (Person 1, ~15 min - this is your Heavy section)

### Slide 1: Methodology
State: "We used STRIDE across 6 trust boundaries (28 threats total) and PASTA's 7-stage process to model Audiobookshelf's attack surface."

### Slide 2: Trust Boundaries Diagram
Recreate this table as a visual:
| ID | Boundary | Risk |
|---|---|---|
| TB-01 | Client ↔ Server (HTTP/WS) | MITM, credential interception |
| TB-02 | Server ↔ External metadata APIs | Data leakage to 3rd parties |
| TB-03 | Server ↔ RSS Feeds (user URL) | **SSRF - highest risk** |
| TB-04 | Server ↔ FFmpeg subprocess | Command injection, memory corruption |
| TB-05 | Docker container ↔ Host | Path traversal, priv esc |
| TB-06 | Standard user ↔ Admin | Privilege escalation |

### Slide 3: STRIDE Category Summary
| Category | Count | Highest Severity |
|---|---|---|
| Spoofing | 3 | High |
| Tampering | 6 | Critical |
| Repudiation | 3 | Medium |
| Information Disclosure | 7 | Critical |
| Denial of Service | 5 | High |
| Elevation of Privilege | 4 | Critical |

### Slide 4: The 3 Critical Findings (present each with detail)

**Finding 1 - SSRF via RSS Feed Ingestion (ST-11, ST-13)**
- Trust boundary: TB-03
- CWE-918
- Explain: the "Add Podcast" feature lets an authenticated user supply *any* URL as a podcast RSS feed. The server fetches it server-side. Because the SSRF filter is admin-toggleable (ST-23) and axios 0.27.2 has known SSRF bypasses (NO_PROXY bypass, IP-alias bypass - GHSA-3p68-rc4w-qgx5, GHSA-m7pr-hjqh-92cm), an attacker can point the feed URL at `http://127.0.0.1:3000/api/users` or `http://169.254.169.254/latest/meta-data/` (cloud metadata endpoint) to pivot the server into an internal recon tool.
- Risk: Critical (Likelihood: High, Impact: Critical)

**Finding 2 - JWT Signing Secret in Plaintext (ST-04, ST-22)**
- Trust boundary: TB-05
- CWE-321 / CWE-312
- Explain: the JWT secret used to sign every access/refresh token is stored as plaintext inside `config.json` on disk. Anyone with container/host filesystem access (path traversal, backup theft, misconfigured volume mount) can read it and forge arbitrarily-privileged tokens, including admin role claims.
- Risk: Critical (Likelihood: Medium, Impact: Critical)

**Finding 3 - FFmpeg Memory Corruption (ST-14) - CVE-2026-40962**
- Trust boundary: TB-04
- CWE-787
- Explain: FFmpeg 8.0.1-r1 (installed as an Alpine `apk` package - notably **not visible in the npm/package.json SBOM scan**, a blind spot in dependency scanning) has an unpatched memory corruption vulnerability triggerable by a crafted media file. Every uploaded audio file gets passed to this subprocess for transcoding.
- Risk: Critical (Likelihood: Medium, Impact: Critical) - **no patch was available at assessment time**

### Slide 5: Attack Scenario Walkthrough (pick 1, present as a story - use Scenario 1, it's the most concrete)
**Scenario: SSRF Internal Recon**
1. Attacker logs in as a low-privilege standard user
2. Adds a new podcast with feed URL `http://127.0.0.1:3000/api/users`
3. Server-side fetch (axios) resolves and requests this URL - the NO_PROXY bypass defeats the SSRF filter
4. Server returns the internal user list back through the podcast-add response/error
5. Attacker repeats with `http://169.254.169.254/latest/meta-data/` to attempt cloud credential theft if hosted on AWS/GCP/Azure

### Slide 6: Risk Matrix (top rows only - Priority 1s)
| ID | Threat | Likelihood | Impact | Risk | Priority |
|---|---|---|---|---|---|
| ST-11/13 | SSRF via RSS feed | High | Critical | Critical | P1 |
| ST-04/22 | JWT secret plaintext | Medium | Critical | Critical | P1 |
| ST-14 | FFmpeg memory corruption | Medium | Critical | Critical | P1 |

### Interactive Task (Slide 7)
**Task:** "A user adds a podcast subscription with the feed URL `http://169.254.169.254/latest/meta-data/`. What STRIDE category and CWE does this fall under, and which trust boundary is being crossed?"

**Solution (Slide 8):** Information Disclosure (also enables further attacks) / CWE-918 (SSRF) / TB-03 (Server ↔ RSS Feeds). This maps to findings ST-11 and ST-13 in our threat model - the highest-risk item identified across all 28 STRIDE threats.

### Narration script skeleton
1. (2 min) Methodology intro
2. (3 min) Trust boundaries + STRIDE category breakdown
3. (6 min) Deep dive on the 3 critical findings, one at a time
4. (2 min) Attack scenario walkthrough
5. (2 min) Interactive task

---
---

## 4. SECTION 4 - Cybersecurity Testing Strategies (Person 2, ~12 min)

### Slide 1: Manual vs Automated Testing - What We Actually Used
| Type | Tool | Used For |
|---|---|---|
| Automated (SBOM/SCA) | Syft + Grype | Dependency vulnerability discovery (Lab 1) |
| Automated (DAST/fuzzing) | Burp Suite Intruder | 166 SQLi + 221 XSS + 13 path traversal payloads against search endpoint |
| Automated (deep SQLi) | sqlmap | `--level=5 --risk=3` boolean/time/error/UNION-based testing |
| Automated (web server scan) | Nikto | 6,544-item scan in 18 seconds |
| Manual | Burp Repeater | JWT tampering, session replay, IDOR testing, error-handling edge cases |

### Slide 2: Prioritization Framework (Risk-Based)
Explain: we tested in order of the Lab 2 risk matrix, not randomly. P1-critical theoretical findings (SSRF, JWT secret extraction, FFmpeg exploitation) were **deliberately not live-exploited** - they require infrastructure access or malicious file crafting beyond the assessment's authorized scope. Instead we prioritized P2-level, safely-testable items: auth flow, session management, injection testing, error handling.

### Slide 3: Testing Coverage Reality Check (be transparent - this is a strength, not a weakness)
| Lab 2 Priority | Threat | Lab 3 Coverage |
|---|---|---|
| P1 Critical | SSRF via RSS (ST-11,13) | **Not Tested** - vuln lives in outdated axios, still unpatched in 2.35.0 |
| P1 Critical | JWT Secret Plaintext | **Partial** - token handling tested; secret extraction not attempted (needs host access) |
| P1 Critical | FFmpeg Memory Corruption | **Partial** - upload endpoint auth/validation tested; CVE exploit file out of scope |
| P2 High | Auth Endpoint Weaknesses | **Fully Tested** |
| P2 High | Path Traversal | **Tested** - 13 payloads, no vulnerabilities confirmed |

### Slide 4: Why This Approach is Correct
Explain the principle: testing strategy should match authorized scope and safety constraints. Live SSRF exploitation against real infrastructure, or crafting a CVE exploit file, carries real risk (server crash, unintended data exposure) and needs separate authorization - a mature testing strategy documents *what wasn't tested and why*, not just what passed.

### Interactive Task (Slide 5)
**Task:** "We identified 3 P1-Critical threats in our threat model but only fully live-tested P2-level threats in Lab 3. Was this a testing failure? Choose the best justification: A) We ran out of time B) P1 threats required infrastructure access or exploit crafting outside authorized scope, so they remain documented but not live-exploited C) The tools couldn't detect them D) They weren't real risks"

**Solution (Slide 6):** B - Testing prioritization should be risk-based *and* scope/safety-aware. Attempting live SSRF against internal targets or crafting a memory-corruption exploit without separate authorization and a sandboxed environment would be irresponsible; a professional pentest report explicitly documents this boundary rather than hiding it.

### Narration script skeleton
1. (3 min) Manual vs automated, mapped to our actual tool use
2. (3 min) Risk-based prioritization framework
3. (3 min) Coverage reality check table - what we tested vs didn't and why
4. (3 min) Interactive task

---
---

## 5. SECTION 5 - Recommended Security Tools (Person 4, ~15 min - your Heavy section)

### Slide 1: Tool Stack Overview
| Stage | Tool | Purpose |
|---|---|---|
| SBOM generation | Syft | Enumerate all packages (npm + apk) into CycloneDX JSON |
| Vulnerability scanning | Grype | Match SBOM against NVD/GitHub Advisory/Alpine tracker |
| Web proxy/fuzzing | Burp Suite Community | Intercept traffic, Intruder fuzzing, Repeater manual testing |
| Web server scan | Nikto v2.1.5 | Misconfig/header/directory scan |
| Deep SQLi testing | sqlmap v1.6.4 | Boolean/time/error/UNION-based injection testing |

### Slide 2: Live Demo Script - Syft + Grype (show the exact commands)
```bash
# Step 1: Generate SBOM from the container image
syft ghcr.io/advplyr/audiobookshelf:latest -o cyclonedx-json > audiobookshelf_sbom.json

# Step 2: Scan the SBOM for known vulnerabilities
grype sbom:./audiobookshelf_sbom.json -o table > grype_results.txt
```
Narrate while showing: "This generated 130 total vulnerability matches across npm and apk ecosystems. After deduplication: **2 Critical, 28 High, 38 Medium, 22 Low**."

Show the results table (from Lab 1):
| Package | Version | Advisory | Severity |
|---|---|---|---|
| ip | 2.0.0 | GHSA-2p57-rm9w-gvfp | High (EPSS 84.6%) |
| form-data | 4.0.0 | GHSA-fjxv-7rqg-78g4 | Critical |
| FFmpeg | 8.0.1-r1 | CVE-2026-40962 | Critical |
| socket.io-parser | 4.2.4 | GHSA-677m-j7p3-52f9 | High |
| sequelize | 6.35.2 | GHSA-6457-6jrx-69cr | High |
| body-parser | 1.20.1 | GHSA-qwcr-r2fm-qrc7 | High |

### Slide 3: Live Demo Script - Burp Suite
Show/describe: Proxy tab capturing live traffic; Intruder configured against `GET /api/libraries/{id}/search?q=` with the payload set (166 SQLi strings, 221 XSS strings). Show a screenshot of the Intruder results grid - all 200 OK, no anomalous length/response to indicate injection.

### Slide 4: Live Demo Script - sqlmap
```bash
sqlmap -u "http://192.168.40.20:13378/api/libraries/{id}/search?q=test" \
  --headers="Authorization: Bearer <token>" \
  --level=5 --risk=3
```
Narrate: "Ran 6+ minutes testing boolean-blind, time-blind, error-based, and UNION-based techniques. Result: parameter not injectable - confirms Sequelize's parameterized queries are working correctly."

### Slide 5: Live Demo Script - Nikto
```bash
nikto -h http://192.168.40.20:13378
```
Narrate: "Scanned 6,544 items in 18 seconds, 11 findings. Key ones: missing X-Frame-Options header (clickjacking risk), `/config/` directory accessible, inode leakage via ETags. On the positive side: CSP `frame-ancestors 'self'` and `X-Content-Type-Options: nosniff` were both present."

### Slide 6: Tools We Evaluated But Didn't Deploy (and why - shows maturity)
| Tool | Reason Not Used |
|---|---|
| XSStrike | Burp's 221-payload sweep already showed uniform 200 OK with no reflection |
| JWT Tool | Repeater already confirmed 401 on type confusion/expiry/signature tampering |
| Commix | Node.js app showed no observable shell execution paths |
| WFuzz | Burp site map already gave complete endpoint inventory |

### Interactive Task (Slide 7)
**Task:** "Which tool would you use to discover that FFmpeg - installed via Alpine's apk package manager rather than npm - has an unpatched CVE, when a standard `npm audit` would completely miss it?"

**Solution (Slide 8):** Syft + Grype (SBOM-based scanning). This is exactly what happened in our own assessment: `npm audit` only sees `package.json` dependencies, but Syft enumerates *all* packages in the container image, including OS-level apk packages - which is how we caught FFmpeg 8.0.1-r1 / CVE-2026-40962, a vulnerability invisible to standard JavaScript dependency auditing.

### Narration script skeleton
1. (2 min) Tool stack overview
2. (3 min) Syft/Grype demo + results
3. (3 min) Burp Suite demo
4. (2 min) sqlmap demo
5. (2 min) Nikto demo
6. (1 min) Tools not used, and why
7. (2 min) Interactive task

---
---

## 6. SECTION 6 - Secure Coding Guidelines (Person 3, ~15 min - your Heavy section)

### Slide 1: Guideline List (derived directly from Lab 2/3 findings)
1. **Never trust the SSRF filter toggle** - hardcode outbound URL validation, don't make it admin-configurable
2. **Never store signing secrets in application config files** - use environment variables or a secrets manager
3. **Always validate/timeout subprocess inputs** - especially for native binaries like FFmpeg
4. **Implement server-side token revocation** - stateless JWTs need a blacklist or short expiry
5. **Keep dependencies current, and scan the whole image** - not just `package.json`

### Slide 2: Annotated Code Example 1 - SSRF Filter (Guideline 1)
```js
// VULNERABLE (current toggleable pattern - ST-23)
if (config.ssrfFilterEnabled) {
  validateUrl(feedUrl); // admin can disable this in settings
}
await axios.get(feedUrl);

// FIXED - hardcoded, non-configurable, allowlist-based
const { URL } = require('url');
function isSafeFeedUrl(feedUrl) {
  const parsed = new URL(feedUrl);
  const blocked = ['127.0.0.1', 'localhost', '169.254.169.254', '::1'];
  if (blocked.includes(parsed.hostname)) return false;
  // reject private ranges (10.x, 172.16-31.x, 192.168.x) with an IP-range check here
  return true;
}
if (!isSafeFeedUrl(feedUrl)) throw new Error('Feed URL not allowed');
await axios.get(feedUrl, { proxy: false, httpAgent: new http.Agent({ family: 4 }) });
```
Explain: this directly fixes ST-11/ST-13/ST-23 from our Lab 2 threat model - removes the toggle, adds an allowlist check, and disables `axios`'s proxy-following behavior (which was the actual bypass vector, GHSA-3p68-rc4w-qgx5).

### Slide 3: Annotated Code Example 2 - JWT Secret Handling (Guideline 2)
```js
// VULNERABLE (current - ST-04/ST-22)
// config.json: { "jwtSecret": "my-plaintext-secret-123" }
const secret = config.jwtSecret;

// FIXED - read from environment, never written to disk
const secret = process.env.JWT_SIGNING_SECRET;
if (!secret) throw new Error('JWT_SIGNING_SECRET must be set via secrets manager/env');
```
Explain: pairs with Docker/K8s secrets mounted as tmpfs, file permissions 600 if a file must be used at all, and a documented key-rotation plan with a grace period so existing sessions don't all break at once.

### Slide 4: Annotated Code Example 3 - Post-Logout Token Validity (real Lab 3 finding, TC08!)
```js
// VULNERABLE (current behavior - confirmed in our own testing, TC08)
// Access token returned 304/valid for up to 1 hour after logout
// because JWT validation is purely stateless (signature+exp check only)

// FIXED - check against a revocation store
async function verifyAccessToken(token) {
  const payload = jwt.verify(token, secret); // existing check
  const isRevoked = await redisClient.get(`revoked:${payload.jti}`);
  if (isRevoked) throw new Error('Token has been revoked');
  return payload;
}
// On logout:
await redisClient.set(`revoked:${accessToken.jti}`, '1', 'EX', 3600); // TTL = token's remaining life
```
Explain: this is our **own confirmed finding** from Lab 3 (TC08) - not theoretical. We tested it ourselves: after logout, the refresh token was correctly invalidated, but the access token remained valid and usable for up to 1 hour. This fix adds a lightweight revocation check without abandoning the performance benefits of stateless JWTs entirely.

### Slide 5: Contributed Snippets (collect 1 from each teammate - leave placeholder, fill in before recording)
| Contributor | Snippet Topic |
|---|---|
| Person 1 | (from Section 3 threat work - e.g. RSS URL parsing hardening) |
| Person 4 | (from Section 5 tool findings - e.g. dependency pin/upgrade example: axios 0.27.2 → 1.7.0) |
| Person 2 | (from Section 4/7 testing work - e.g. input validation pattern confirmed safe by sqlmap/Burp) |

### Interactive Task (Slide 6)
**Task:** "Look at this code: `if (config.ssrfFilterEnabled) { validateUrl(feedUrl); }`. Identify the vulnerability and name the STRIDE/CWE classification."

**Solution (Slide 7):** The vulnerability is that URL validation is conditional on an admin-configurable setting rather than always enforced - this is ST-23 in our threat model (Tampering, CWE-16: Configuration), and it's what allows ST-11/ST-13 (SSRF, CWE-918) to be exploitable at all if an admin disables the filter. The fix is to make the check unconditional and allowlist-based.

### Narration script skeleton
1. (2 min) Guideline list overview
2. (4 min) SSRF filter code walkthrough
3. (3 min) JWT secret handling walkthrough
4. (3 min) Post-logout token validity walkthrough (emphasize: this is OUR finding, not textbook)
5. (1 min) Contributed snippets
6. (2 min) Interactive task

### Also produce: standalone **Secure Coding Guidelines PDF**
Structure: 1 page per guideline (5 total) with vulnerable/fixed code pairs above, plus the 3 contributed snippets appended at the end.

---
---

## 7. SECTION 7 - Comparison with Alternative Applications (Person 2, ~12 min)

### Slide 1: Candidates
From Lab 1's competitive landscape: **Booksonic** (declining, legacy Subsonic-based) and **Kavita** (actively maintained, ASP.NET Core, broader media focus).

### Slide 2: Architecture Comparison Table
| Aspect | Audiobookshelf | Kavita | Booksonic |
|---|---|---|---|
| Stack | Node/Express + Nuxt/Vue + SQLite | ASP.NET Core | Legacy Subsonic-based (Java) |
| Maintenance | Active | Active | Declining |
| Auth | JWT + refresh rotation, OIDC/SSO w/ PKCE | Built-in + OIDC support | Basic auth model, less modern |
| SSRF-relevant features | RSS feed ingestion (user-controlled URL) - our #1 risk | Similar external metadata fetching, different implementation | Minimal external fetch surface |
| Container hardening | Alpine base, Docker | Similar container practices | Older base images, less frequent updates |

### Slide 3: Threat Exposure Comparison
Explain: Audiobookshelf's **specialization for audio/podcast content** is a double-edged sword - it means a narrower feature surface than Kavita (which handles comics/manga/books too, larger attack surface overall), but the RSS/podcast-feed SSRF vector (ST-11/13) is fairly unique to Audiobookshelf's podcast-focused design; Kavita's metadata-fetching surface has different, if analogous, risk.
Booksonic, being on a legacy/less-maintained codebase, likely carries *more* unpatched dependency risk overall (similar to what we found via Syft/Grype for Audiobookshelf, but with a longer-neglected dependency tree) even though we didn't run a live SBOM scan against it for this lab.

### Slide 4: Mitigation Strategy Differences
Explain: Kavita's more modern .NET stack has some SSRF protections built into `HttpClient` defaults that require more deliberate opt-out to bypass, versus Audiobookshelf's axios-based fetching where the SSRF filter is an explicit, disable-able toggle (our ST-23 finding) - a design choice that shifts responsibility onto admin configuration rather than enforcing it in code.

### Interactive Task (Slide 5)
**Task:** "Compare two authentication flows: Audiobookshelf uses JWT with refresh token rotation and OIDC/PKCE support. Kavita uses a similar JWT + optional OIDC model. Given our Lab 3 finding (TC08 - access tokens remain valid up to 1 hour post-logout due to no server-side blacklist), which design choice would most directly close this gap, and does either project implement it by default?"

**Solution (Slide 6):** A server-side token revocation list (blacklist) or very short-lived access tokens (e.g., 5 minutes) with mandatory refresh would close the gap. Neither project implements a full blacklist by default (as of our testing) - this is a common tradeoff across stateless-JWT architectures, not unique to Audiobookshelf, which reinforces that our TC08 finding reflects an industry-wide pattern in JWT-based auth rather than a one-off implementation mistake.

### Narration script skeleton
1. (2 min) Candidate selection rationale
2. (4 min) Architecture comparison table walkthrough
3. (3 min) Threat exposure comparison, tie back to our own Section 3/5 findings
4. (3 min) Interactive task

---
---

## 8. SECTION 8 - Wrap-Up & Developer Checklist (Person 3, ~10 min)

### Slide 1: Key Takeaways Recap (1 line per section)
1. Audiobookshelf's architecture centers on 6 trust boundaries, with RSS feed ingestion being the single highest-risk surface
2. PIPEDA applies directly to our deployment; several gaps (breach notification, encryption at rest) are unaddressed
3. 3 Critical threats identified: SSRF (RSS), plaintext JWT secret, FFmpeg memory corruption
4. Testing strategy must be risk-based *and* scope-aware - not every theoretical threat should be live-exploited
5. SBOM tools (Syft/Grype) caught what npm audit alone would miss (FFmpeg/apk package)
6. Secure coding fixes exist for all 3 critical findings, and one real bug (TC08 post-logout token validity) was found via our own live testing
7. Comparable projects (Kavita, Booksonic) share similar architectural tradeoffs - this isn't an Audiobookshelf-specific failure pattern

### Slide 2: Developer Security Checklist (final, compiled from all 4 sections)
- [ ] Never make SSRF/security filters admin-toggleable - hardcode the check
- [ ] Never store signing secrets in plaintext config files - use env vars/secrets manager
- [ ] Implement server-side token revocation for logout, not just refresh-token invalidation
- [ ] Run SBOM-based scanning (Syft+Grype), not just `npm audit` - catch OS-level package vulnerabilities
- [ ] Validate and timeout all subprocess calls to native binaries (FFmpeg)
- [ ] Set X-Frame-Options and restrict access to `/config/` at the web server/reverse-proxy layer
- [ ] Enforce HTTPS by default, don't rely solely on optional reverse-proxy setup
- [ ] Encrypt SQLite at rest (SQLCipher) rather than relying on host disk encryption alone
- [ ] Document explicitly what was NOT tested and why, in every security assessment

### Interactive Task: none required for Section 8 (wrap-up), but include a closing reflection prompt: "Which single item on this checklist would you implement first if you were the Audiobookshelf maintainer, and why?" - leave open, no single correct answer, meant to prompt discussion in the video.

### Narration script skeleton
1. (5 min) Recap all 7 prior sections, 1 line each
2. (3 min) Walk the final checklist
3. (2 min) Closing reflection prompt + sign-off

---
---

## 9. Exact Commands Reference Sheet (copy-paste ready for whoever demos live)

```bash
# --- SBOM + Vulnerability Scanning (Section 5) ---
syft ghcr.io/advplyr/audiobookshelf:latest -o cyclonedx-json > audiobookshelf_sbom.json
grype sbom:./audiobookshelf_sbom.json -o table > grype_results.txt

# --- sqlmap deep SQLi test (Section 5) ---
sqlmap -u "http://192.168.40.20:13378/api/libraries/{id}/search?q=test" \
  --headers="Authorization: Bearer <token>" \
  --level=5 --risk=3

# --- Nikto web server scan (Section 5) ---
nikto -h http://192.168.40.20:13378

# --- Test file generation for upload testing (referenced in Lab 3) ---
ffmpeg -f lavfi -i "sine=frequency=1000:duration=1" -c:a libmp3lame test.mp3
```
> Note: run these against your OWN local instance only, never a production/public deployment. This mirrors exactly what the group did in Labs 1–3 with maintainer permission.

---
---

## 10. Real Test Case Reference (for anyone who wants exact wording for slides - pull directly, don't reinvent)

| ID | Area | Result | Use In Section |
|---|---|---|---|
| TC01–TC04 | SQLi (Intruder + sqlmap) | Pass - not injectable | 5, 4 |
| TC05–TC09 | Auth/session/logout | TC08 = Partial (token valid post-logout) | 4, 6 |
| TC10 | Cross-user DELETE | Info - by design, not a vuln | 4 |
| TC11–TC13 | File upload validation | Pass | 4, 6 |
| TC14 | Nikto scan | Finding - missing X-Frame-Options, /config/ exposed | 5 |
| TC15–TC19 | Error handling | Pass - no info leakage | 4, 6 |

---
---

## 11. What NOT to Claim (accuracy guardrails - read before recording)
- Do **not** say you "exploited" SSRF, JWT secret extraction, or the FFmpeg CVE - these were **not live-tested**, only identified via SBOM/threat modeling. Say "identified but not exploited due to scope/safety constraints."
- Do **not** call the cross-user DELETE behavior a vulnerability - the developer confirmed it's intentional shared-library design.
- Do **not** claim v2.35.0 has zero outdated components - Lab 3's scanner limitation contradicts Lab 1/2's deeper SBOM findings; note this discrepancy honestly if asked.
