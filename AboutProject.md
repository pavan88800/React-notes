# Interview Prep: Intro & AirOps

## 1. Opening Intro (~45 sec)

_Use when asked: "Tell me about yourself."_

> Hi, I'm Pavan. I'm a Software Engineer at GeekyAnts with about five years of experience in frontend development, mainly building web applications with React, TypeScript, and Redux.
>
> Currently I'm working on AirOps, an aviation operations platform that helicopter operators use to run their day-to-day operations: flight planning and booking, crew management, flight reports, logbooks, fuel, expenses, and billing.
>
> On the frontend, I build and maintain features across these modules. One area I've worked on closely is the Web Planner, a central screen where operations teams view and manage scheduled flights. Since it's a data-heavy screen, a lot of my work there has been around state management, reusable components, and keeping it fast and reliable.
>
> I work closely with backend and QA to take features from development through to production, including debugging live issues when they come up.

---

## 2. Project Deep-Dive (~60 sec)

_Use when asked: "Tell me about your current project."_

> AirOps is used by helicopter operators to run their daily operations: planning and booking flights, assigning crew, recording flight reports and logbooks, and tracking fuel, expenses, and billing.
>
> One of the core screens is the Web Planner, where ops teams see and manage all scheduled flights in one place. From a frontend perspective, it's challenging because it's very data-heavy. A single view pulls together flights, crew, and aircraft, and changes need to reflect quickly and accurately.
>
> So a lot of my work is around structuring API and state handling cleanly with Redux, building reusable components that different modules can share, and keeping these heavy screens performant. Because this is operational data, reliability matters a lot, so I work closely with backend and QA to make sure what we ship is correct before it reaches users.

---

## 3. Cheat Sheet

| Topic            | Key points                                                                                                                   |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| **Product**      | Aviation ops platform for helicopter operators                                                                               |
| **Modules**      | Flight planning & booking, crew management, flight reports, logbooks, fuel, expenses, billing                                |
| **Web Planner**  | Central screen to view and manage scheduled flights                                                                          |
| **Stack**        | React, TypeScript, JavaScript, Redux                                                                                         |
| **My role**      | Features, reusable components, performance, accessibility, production issues                                                 |
| **Senior angle** | Link business workflow → engineering decisions: data-heavy UI, state/API handling, performance, reliability, cross-team work |

---

## 4. Things to Avoid

- ❌ "software engineering at" → ✅ "software engineer at"
- ❌ "five plus year" → ✅ "five years"
- ❌ Listing all 7 modules as your work. Pick 1–2 you actually owned.
- ❌ Ending on filler ("Overall, I've gained good experience…"). End on something concrete.
- ❌ Made-up numbers. Interviewers will dig into them.

---

## 5. Challenge Stories (STAR)

_Use when asked: "Tell me about a challenging problem you solved."_

> ⚠️ **These are templates.** Keep only what actually happened, and replace every `[bracket]` with real details. Prepare **2 stories well** rather than 4 halfway.

---

### Story 1: Performance (Web Planner)

> **Situation:** In AirOps, the Web Planner shows all scheduled flights along with crew and aircraft details. As operators added more flights, the screen became slow. Scrolling lagged, and updating one flight caused the whole view to re-render.
>
> **Task:** I was responsible for figuring out why it was slow and fixing it without breaking existing workflows, since ops teams use this screen all day.
>
> **Action:** First, I used React DevTools Profiler to find what was re-rendering. The main issue was that many components subscribed to large parts of the Redux store, so any change re-rendered everything. I restructured the selectors using memoized selectors so each component only got the data it needed. I wrapped heavy row components in `React.memo`, stabilized callbacks with `useCallback`, and [added list virtualization / pagination] so we weren't rendering every flight at once. I tested with QA against realistic data volumes before release.
>
> **Result:** The Planner became noticeably smoother. [Real outcome: e.g., "render time dropped from X to Y" or "the ops team stopped reporting lag."] It also gave us a pattern for other data-heavy screens in the app.

**Signal:** profiling before fixing, understanding _why_ React re-renders.

---

### Story 2: Production Bug

> **Situation:** After a release, operators reported that [flight reports / logbook entries] were showing wrong or missing data on the web app. Since this is operational data used for compliance and billing, it was high priority.
>
> **Task:** I was asked to find the root cause and ship a fix quickly without introducing new issues.
>
> **Action:** First, I reproduced the issue using the same account and data setup the operator had. Then I checked the network tab and compared the API response with what the UI showed. The API data was correct, so the problem was on the frontend. It turned out that [a Redux state update was overwriting data / a stale cache was shown after navigation / a date conversion was using the wrong timezone]. I fixed the logic, added a check for that edge case, and coordinated with QA to test the specific scenario plus related flows. We deployed a hotfix the same day.
>
> **Result:** The issue was resolved for all operators. To prevent it happening again, I [added test cases / added a reusable utility for date handling / documented the edge case for the team].

**Signal:** calm, methodical debugging (reproduce → isolate → fix → verify → prevent).

---

### Story 3: Reusable Component

> **Situation:** Many AirOps modules, like flights, crew, fuel, and expenses, needed data tables. Each module had built its own version, so the UI was inconsistent and the same bugs had to be fixed in several places.
>
> **Task:** I took ownership of building one shared table component that all modules could use.
>
> **Action:** I first listed what each module needed: sorting, filtering, pagination, row actions, loading and empty states. Then I designed a generic component in React and TypeScript with typed column definitions, so each module only had to pass its columns and data. I kept styling and behavior consistent, made it keyboard-accessible, and [added virtualization for large datasets]. I migrated [one or two modules] first, got feedback from the team, and then rolled it out to the rest.
>
> **Result:** New tables now take [hours instead of days] to build, the UI is consistent across modules, and a bug fix in the table fixes it everywhere.

**Signal:** thinking about the whole codebase and team productivity, not just one ticket.

---

### Story 4: Complex Feature (Crew Assignment)

> **Situation:** Operators needed to assign crew to flights, but there are many rules involved: crew must be qualified for the aircraft type, available at that time, and within duty-time limits. If a wrong assignment slips through, it's an operational and safety risk.
>
> **Task:** I built the crew assignment flow on the frontend.
>
> **Action:** I worked with the backend team to decide which validations would run where. Instant checks, like availability and qualifications, ran on the frontend for quick feedback, while the backend remained the final authority. On the UI, I built a form that shows only eligible crew, with clear warning and error messages explaining _why_ someone can't be assigned. I managed the form state and validation in [React Hook Form / Redux], handled conflicts when two users edited the same flight, and tested edge cases with QA, like overlapping flights and last-minute changes.
>
> **Result:** Operators could assign crew faster and with fewer errors, and the clear error messages [reduced support questions / reduced back-and-forth with the ops team].

**Signal:** understanding the business domain and working across frontend and backend.

---

### Likely Follow-up Questions

- "How did you measure the improvement?"
- "What would you do differently now?"
- "Why did you choose [X] over [Y]?"
- "What was the hardest part?"
