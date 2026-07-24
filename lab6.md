# Lab 6 – Task Division (Audiobookshelf, 4-Person Group)
# Lab 6 - detailed ([_click me_](https://github.com/PawKoi/was705/blob/main/lab6-detailed.md))

## Section & Workload Assignments

| Person | Sections Owned | Load Type | Video Segments to Record | Slide Deck | Own-Instance Scan (ZAP + Dependency-Check) | Code Snippet for Sec. 6 | Threat Test on Own Instance | Reflection Report |
|---|---|---|---|---|---|---|---|---|
| **Person 1** | 3. Key Threat Events + 1. Intro & Developer Roles | Heavy + Light | 2 | Own 2 sections | ✓ | ✓ | ✓ | Own |
| **Person 2** | 5. Recommended Security Tools + 2. Applicable Regulations | Heavy + Light | 2 | Own 2 sections | ✓ | ✓ | ✓ | Own |
| **Person 3** | 6. Secure Coding Guidelines + 8. Wrap-Up & Checklist | Heavy + Light | 2 | Own 2 sections | ✓ | ✓ | ✓ | Own |
| **Person 4** | 4. Testing Strategies + 7. Comparison with Alternatives | Medium + Medium | 2 | Own 2 sections | ✓ | ✓ | ✓ | Own |

## Shared Deliverables

| Deliverable | Owner | Everyone's Contribution |
|---|---|---|
| Slide Deck (AODA-compliant, merged) | Person 1 assembles final file | Each builds their own 2 sections; rotate accessibility QA (check a teammate's slides for alt text/contrast/fonts) |
| Developer Handout (PDF) | Person 3 | Each adds 1 item from their own section |
| Secure Coding Guidelines (PDF) | Person 3 | Each contributes 1 annotated code snippet from their own instance |
| Comparison with Alternatives (PDF) | Person 4 | Person 1 & 2 share threat/tool findings to strengthen comparison |
| Wrap-Up Checklist | Person 3 | Each submits 1–2 checklist items distilled from their own sections |
| Reflection Report (PDF) | Individual | Everyone writes their own — confirm with instructor if 1/group or 1/student |

## Section Difficulty Reference

| Section | # | Difficulty |
|---|---|---|
| Intro & Developer Roles | 1 | Light |
| Applicable Regulations | 2 | Light |
| Key Threat Events (STRIDE/PASTA) | 3 | Heavy |
| Testing Strategies | 4 | Medium |
| Recommended Security Tools | 5 | Heavy |
| Secure Coding Guidelines | 6 | Heavy |
| Comparison with Alternatives | 7 | Medium |
| Wrap-Up & Checklist | 8 | Light |

## Notes / To Confirm with Instructor
- Confirm whether "same video length per student" means equal total speaking time across each person's 2 sections.
- Confirm whether the Reflection Report is submitted once per group or once per student.


---
---
# DETAILED

---
---

Total: 100 points | 8 sections | 2 hours video | 4 people, own Audiobookshelf instances

---

## 1. Master Assignment Table

| Person | Sections Owned | Difficulty | Est. Prep Time | Video Length Owed | Rubric Points Covered |
|---|---|---|---|---|---|
| **P1** | 1. Intro & Developer Roles, 3. Key Threat Events | Light + Heavy | ~6–7 hrs | 20–30 min | Structure (10), Regulations & Threats (15, shared w/ P2) |
| **P2** | 2. Applicable Regulations, 5. Recommended Security Tools | Light + Heavy | ~6–7 hrs | 20–30 min | Regulations & Threats (15, shared), Testing & Tools (15, shared w/ P4) |
| **P3** | 6. Secure Coding Guidelines, 8. Wrap-Up & Checklist | Heavy + Light | ~6–7 hrs | 20–30 min | Secure Coding (15), Annotated Code (10) |
| **P4** | 4. Cybersecurity Testing Strategies, 7. Comparison w/ Alternatives | Medium + Medium | ~6–7 hrs | 20–30 min | Testing & Tools (15, shared), Comparison (10) |

Every person also contributes to: AODA compliance pass, Reflection Report, Secure Coding contributions, Checklist items. → these balance total effort across all 4.

---

## 2. Per-Section Detailed To-Do Lists

### Section 1 — Introduction & Developer Roles (P1) — Light
| # | Task |
|---|---|
| 1.1 | Write 1-paragraph overview of Audiobookshelf (purpose, tech stack: Node/Express backend, Vue frontend, SQLite) |
| 1.2 | List developer roles relevant to security (backend/API dev, frontend dev, DevOps/deployment) |
| 1.3 | Identify 2–3 security challenges specific to Audiobookshelf (file upload handling, self-hosted deployment exposure, RSS/metadata ingestion) |
| 1.4 | Draft slide(s): title, agenda, roles diagram |
| 1.5 | Write 1 interactive task (e.g. "Which role is responsible for validating uploaded audio metadata?") + solution slide |
| 1.6 | Write narration script (10–15 min) |
| 1.7 | Record video segment (screen + speaker cam) |

### Section 2 — Applicable Regulations (P2) — Light
| # | Task |
|---|---|
| 2.1 | Research which regulations apply (GDPR — user data/listening history; general accessibility law since AODA is already required for the deck itself) |
| 2.2 | Explain impact on dev practices (data minimization, right to deletion, consent for any analytics) |
| 2.3 | Build slide(s) mapping regulation → required control → where it applies in Audiobookshelf |
| 2.4 | Write 1 interactive task ("Match the regulation to the required control") + solution slide |
| 2.5 | Write narration script |
| 2.6 | Record video segment |

### Section 3 — Key Threat Events (P1) — Heavy
| # | Task |
|---|---|
| 3.1 | Build/refresh STRIDE or PASTA model for Audiobookshelf using findings from Labs 1–5 |
| 3.2 | Select 2–3 concrete threat scenarios (e.g. path traversal on upload, auth/session hijacking, malicious RSS feed injection) |
| 3.3 | Attempt/document 1 real test against own instance illustrating a scenario (safely, on your own local instance only) |
| 3.4 | Build slide(s): STRIDE/PASTA table, attack diagram, screenshots/evidence from own instance |
| 3.5 | Write 1 interactive task ("Identify the vulnerability in this code snippet" or scenario-based) + solution slide |
| 3.6 | Write narration script (this section may run slightly longer — budget 15 min) |
| 3.7 | Record video segment |
| 3.8 | Collect threat-testing notes from P2, P3, P4 (each ran 1 test on their own instance) to enrich "real examples" |

### Section 4 — Cybersecurity Testing Strategies (P4) — Medium
| # | Task |
|---|---|
| 4.1 | Explain manual vs. automated testing with pros/cons |
| 4.2 | Build a risk-based prioritization framework (e.g. likelihood × impact matrix) applied to Audiobookshelf features |
| 4.3 | Reference specific tools used in Labs 2–5 as testing methods (setup for Section 5 without duplicating content) |
| 4.4 | Build slide(s): manual vs automated comparison table, prioritization matrix |
| 4.5 | Write 1 interactive task ("Choose the correct mitigation strategy" or prioritization scenario) + solution slide |
| 4.6 | Write narration script |
| 4.7 | Record video segment |

### Section 5 — Recommended Security Tools (P2) — Heavy
| # | Task |
|---|---|
| 5.1 | Run OWASP ZAP scan against own Audiobookshelf instance; capture screenshots/findings |
| 5.2 | Run Dependency-Check against the Node.js dependency tree; capture findings |
| 5.3 | (Optional/if time) Run Burp Suite against a specific API endpoint |
| 5.4 | Collect equivalent scan results from P1, P3, P4's own instances for a 4-way comparison |
| 5.5 | Build slide(s): tool overview, live/recorded scan walkthrough, comparison table of findings across all 4 instances |
| 5.6 | Write 1 interactive task ("Which tool would you use to detect X?") + solution slide |
| 5.7 | Write narration script |
| 5.8 | Record video segment (screen recording of actual tool usage required) |

### Section 6 — Secure Coding Guidelines (P3) — Heavy
| # | Task |
|---|---|
| 6.1 | Identify 3–5 secure coding guidelines specific to Node/Express/Vue/SQLite stack |
| 6.2 | Pull annotated code examples directly from the Audiobookshelf repo (own codebase) — before/after or vulnerable/fixed pairs |
| 6.3 | Collect 1 code snippet contribution from each of P1, P2, P4 |
| 6.4 | Build slide(s): guideline list, annotated code (syntax-highlighted), explanation of why each matters |
| 6.5 | Write 1 interactive task ("Identify the vulnerability in this code snippet") + solution slide |
| 6.6 | Write narration script |
| 6.7 | Record video segment |
| 6.8 | Compile all guidelines + snippets into standalone **Secure Coding Guidelines PDF** (separate submission) |

### Section 7 — Comparison with Alternative Applications (P4) — Medium
| # | Task |
|---|---|
| 7.1 | Select 1–2 comparable self-hosted apps (e.g. Kavita, Booklore, Navidrome, Jellyfin) |
| 7.2 | Compare architecture (stack, auth model, deployment) |
| 7.3 | Compare threat exposure and mitigation strategies side by side |
| 7.4 | Pull in relevant findings from P1's threat work and P2's tool scans to strengthen comparison |
| 7.5 | Build slide(s): comparison table, architecture diagrams |
| 7.6 | Write 1 interactive task ("Compare two authentication flows and identify the more secure one") + solution slide |
| 7.7 | Write narration script |
| 7.8 | Record video segment |

### Section 8 — Wrap-Up & Developer Checklist (P3) — Light
| # | Task |
|---|---|
| 8.1 | Collect 1–2 checklist items from each teammate (drawn from their own sections) |
| 8.2 | Synthesize into a single secure-development checklist |
| 8.3 | Write summary of key takeaways across all 8 sections |
| 8.4 | Build slide(s): final checklist, key takeaways recap |
| 8.5 | Write narration script |
| 8.6 | Record video segment |
| 8.7 | Assemble final merged slide deck from all 4 people's section slides |

---

## 3. Shared / Cross-Cutting Tasks

| Task | Owner | Contributors | Notes |
|---|---|---|---|
| Merge all section slides into 1 deck | P3 | all send finished slides by agreed deadline | must be single file |
| AODA compliance pass (fonts, alt text, contrast) | Rotate: each person checks a teammate's slides, not their own | all | do this *after* merge, not before |
| Export slide deck to PDF | P3 | — | final submission format |
| Own-instance vulnerability scan (ZAP + Dependency-Check) | — | all 4, individually | feeds Section 5 comparison table |
| 1 threat scenario test on own instance | — | all 4, individually | feeds Section 3 real-examples requirement |
| 1 annotated code snippet | — | all 4, individually | feeds Section 6 + Secure Coding Guidelines PDF |
| Developer Handout (PDF) | P3 drafts | all add 1 item from their own section | separate submission |
| Secure Coding Guidelines (PDF) | P3 | all contribute snippets | separate submission |
| Reflection Report (PDF, 1–2 pages) | Individual | each person writes their own | confirm with instructor: 1/group or 1/student |
| Video recording (screen + speaker cam, no AI voice/avatar) | — | each records their own 2 sections | equal length by construction |
| Final file naming & individual submission (no ZIPs) | Whoever submits | all confirm files before submission | check portal requirements |

---

## 4. Full Submission Checklist

| # | Deliverable | Format | Owner/Assembler |
|---|---|---|---|
| 1 | Video Segment – Section 1 | MP4 / link | P1 |
| 2 | Video Segment – Section 2 | MP4 / link | P2 |
| 3 | Video Segment – Section 3 | MP4 / link | P1 |
| 4 | Video Segment – Section 4 | MP4 / link | P4 |
| 5 | Video Segment – Section 5 | MP4 / link | P2 |
| 6 | Video Segment – Section 6 | MP4 / link | P3 |
| 7 | Video Segment – Section 7 | MP4 / link | P4 |
| 8 | Video Segment – Section 8 | MP4 / link | P3 |
| 9 | Slide Deck (AODA-compliant, all 8 sections) | PDF | P3 assembles |
| 10 | Developer Handout | PDF | P3 |
| 11 | Secure Coding Guidelines (project-specific) | PDF | P3 |
| 12 | Reflection Report | PDF, 1–2 pages | Each person, individually |

---

## 5. Suggested Timeline (working backward from due date)

| Phase | Tasks | Who |
|---|---|---|
| Week 1 | Each person runs ZAP + Dependency-Check scan on own instance; runs 1 threat test; drafts own 2 sections' content outline | All |
| Week 1–2 | Draft slides for own 2 sections; write narration scripts; draft interactive task + solution | All |
| Week 2 | Send code snippets to P3 for Secure Coding section; send checklist items to P3 for Section 8; send threat notes to P1 for Section 3; send scan results to P2 for Section 5 | All → P3/P1/P2 |
| Week 2–3 | Record all 8 video segments | All (own sections) |
| Week 3 | Merge slide deck; rotate AODA QA pass; assemble Handout + Secure Coding PDFs | P3 leads, all support |
| Week 3 | Each person writes own Reflection Report | All |
| Final day | Confirm file names, formats, no-ZIP compliance; submit individually | All |

---

## 6. Rubric Coverage Cross-Check

| Rubric Criteria | Points | Primarily Covered By |
|---|---|---|
| Training Structure & Section Planning | 10 | All (shared responsibility, checked in Section 1/8) |
| Coverage of Regulations & Threat Events | 15 | P1 (Sec 3) + P2 (Sec 2) |
| Cybersecurity Testing Strategies & Tools | 15 | P4 (Sec 4) + P2 (Sec 5) |
| Secure Coding Guidelines (Project-Specific) | 15 | P3 (Sec 6) |
| Annotated Code Examples | 10 | P3 (Sec 6), with snippets from all |
| Comparison with Alternative Applications | 10 | P4 (Sec 7) |
| PowerPoint with Tasks & Solutions | 15 | All (each section's task/solution slide) |
| Video Quality & Presentation | 10 | All (individual recording quality) |
| Reflection Report | 10 | All (individual) |
| AODA Compliance & Professionalism | 10 | Rotated QA pass, final by P3 |

---

## 7. Open Questions for Instructor
- [ ] Does "same video length per student" mean total time across both owned sections should match across all 4 people?
- [ ] Is the Reflection Report one per group or one per student?
- [ ] Is a single merged slide deck acceptable, or must each section's slides be traceable to an individual author (e.g. via footer credit)?
