---

### 1. Revised 5-Sentence Flow (with specialties embedded)

1. A user opens Mashwary, sets pickup/destination, and applies specialties — female-driver filter, scheduled trip, parent-approval mode, or elderly voice-request — before confirming.
2. The backend matches an available driver (respecting gender preference where possible) and shows driver info and the safety-camera disclosure before the user books.
3. During the trip, the passenger watches the live map, shares live location with a trusted contact, and the in-car camera records under role-based access rules.
4. Parents see the car's live location (but not the teen's device) via 2FA-style approval; elderly users' rides are booked by a linked family member through voice or basic message.
5. On completion, the trip is rated and logged, coupons are credited for long trips, and all footage/metadata is retained only per the privacy policy.

---

### 2. Feature Classification: Prototype (Now) vs. Future

| Feature | Difficulty | Ship in Prototype Now? | Why / Dependency |
|---|---|---|---|
| Registration/login (name, email, phone, country, location) | Low | ✅ Yes | Pure form + auth; already sketched |
| Pickup/destination + live map tracking | Medium | ✅ Yes | Google Maps/Mapbox API; core demo value |
| Driver info display + female-driver filter | Medium | ✅ Yes | Needs a seeded driver DB; filter is a simple query flag |
| Live location sharing with trusted contact | Low–Medium | ✅ Yes | Shareable link + map; high demo impact |
| Arabic/English toggle (i18n) | Low | ✅ Yes | Static string tables; strong differentiator |
| Scheduled trips | Low | ✅ Yes | Timestamp field + queue |
| Rating/review after trip | Low | ✅ Yes | Simple CRUD |
| In-car safety camera *concept* (mock feed + access-rule screen) | Low (mock) | ✅ Yes (mock only) | Real streaming is heavy; show the privacy-rule UI |
| Parental approval (2FA-style) | Medium | ⚠️ Partial | Mock the approve/deny flow; real 2FA later |
| Elderly voice/basic-message ride request | Medium | ⚠️ Partial | Show UI + canned voice note; no real STT |
| Coupon/gamified long-trip rewards | Low–Medium | ⚠️ Optional | Simple counter is easy; ad-based games are not |
| Real in-car camera streaming + storage | High | ❌ Future | Hardware, bandwidth, legal retention policy |
| Real driver onboarding/KYC/background checks | High | ❌ Future | Legal/compliance, partnerships |
| Female Training Center (FTC) integration | High | ❌ Future | Organizational partnership, not software |
| Live payments | High | ❌ Future | Payment gateway + licensing |
| Real-time matching algorithm at scale | High | ❌ Future | Needs real driver supply |

**Rule of thumb:** anything needing *hardware, licensing, real supply, or money movement* is future. Anything that is *UI + seeded data + a public API* ships in the prototype.

---

### 3. What You Must Know About Your Own Project

| Category | What to know | Your current status |
|---|---|---|
| **Core value proposition** | One sentence: "Safe, private, bilingual ride-booking with driver choice and trusted-person tracking." | ✅ Stated in abstract |
| **Primary users** | Women, teens 16+, elderly, parents | ✅ Defined (p.7) |
| **Killer differentiators** | Female-driver option, safety camera + privacy rules, parental controls, full Arabic | ✅ Strong — lead with these |
| **Must-have vs nice-to-have** | Must: booking, map, tracking, share, i18n. Nice: coupons, gamification | ⚠️ Not yet separated in doc |
| **Technical stack** | Web frontend, map API, auth, DB, i18n library | ❌ Not specified — decide before building |
| **Data & privacy model** | Who sees camera footage, retention period, consent flow | ⚠️ Mentioned as principle, no policy written |
| **Feasibility boundaries** | No real drivers, no hardware, no payments in v1 | ⚠️ Implied, not stated |
| **SDG mapping** | SDG 8, 11 (+ Gender Equality, Quality Education via FTC) | ✅ Stated (p.6) |
| **Business model** | Commission? coupons? ads? | ❌ Canvas section is empty |
| **Competitive gap** | Uber/Careem: weak Arabic, no female-driver pick, vague privacy | ✅ Implied — make explicit |
| **Evaluation plan** | Test cases, survey, usability feedback | ⚠️ Template only — tables empty |
| **Known contradictions** | Sections 2.3–2.12 + Section III still contain **"Pet Finder"** lost-pet content | ❌ Must fix before submission |

**The three things you must decide before coding:** (1) map provider, (2) auth method, (3) the exact privacy rule for camera access. Everything else is UI polish.

---

### 4. Clean Formal 5-Sentence Summary (final)

1. Mashwary is a bilingual web platform where users set a pickup point and destination and optionally apply specialties such as a female-driver filter, scheduled timing, parental approval, or elderly voice-assisted booking.
2. The system matches an available driver, presents driver information and the safety-camera disclosure, and confirms the ride.
3. Throughout the trip, the passenger follows the live map, shares live location with a trusted contact, and the in-car camera records under defined role-based privacy rules.
4. Parents monitor the car's location without accessing the teenager's device, while family members can book on behalf of elderly users.
5. The prototype will demonstrate booking, live tracking, location sharing, driver filtering, and bilingual support using seeded data, while real camera streaming, payments, driver onboarding, and the Female Training Center remain future phases.
