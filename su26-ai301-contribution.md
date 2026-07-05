# Contribution [#]: #2623

**Contribution Number:** 2  
**Student:** Hana Ahmed 
**Issue:** https://github.com/carlos-emr/carlos/issues/2623
**Status:** Phase I (Completed)

---

## Why I Chose This Issue

I have already worked on this contributer previous issue and liked it so I was thrilled to find there is more issues that are very intresting and challenging for me to work on
---

## Understanding the Issue

### Problem Description

So basically: this EMR app has a feature that shows a patient's chart (like their blood pressure over time). To load it, you just pass in a patient ID number.
The problem? It checks "are you logged in" but never checks "are you allowed to see this specific patient." So literally any logged-in user — receptionist, random staff, whoever — can just swap the ID number in the URL and pull up someone else's medical chart. No permission check at all.
That's a classic IDOR bug (insecure direct object reference) 


### Expected Behavior

When a user requests a patient's measurement chart via `ScatterPlotChartServlet`, the system should verify — for every request, not just at login — that the requesting user has explicit privilege to view that specific patient's clinical data. If the user lacks that privilege (e.g., no clinical relationship to the patient, insufficient role), the servlet should reject the request with a `403 Forbidden` response and should not render or return any chart data.

### Current Behavior

The servlet only confirms that the user is authenticated (via `LoginFilter`) — it never checks whether the user is *authorized* to view the specific patient referenced by the `demographicNo` parameter. As a result, any logged-in user can substitute any other patient's `demographicNo` into the request and successfully retrieve that patient's measurement chart, regardless of role or clinical relationship. This is a broken access control / IDOR vulnerability.

### Affected Components

-`ScatterPlotChartServlet` — the primary vulnerable file; missing the `SecurityInfoManager.hasPrivilege()` check present in comparable servlets.
- `SecurityInfoManager` — the existing privilege-checking utility that should be invoked here but isn't.
- Likely related/comparable servlets worth auditing for the same gap (to confirm during reproduction — check if sibling chart/report servlets follow the same unguarded pattern).

---

## Reproduction Process

### Environment Setup

I was having a problem in setting and downloading docker because I didn't have it previously.

### Steps to Reproduce

1. step 1: Clonning the enviroments 
2. understanding the extentions I need to download 
3. Finaly analysing the source code and finding the issue

### Reproduction Evidence

- **Commit showing reproduction:** [Link to commit in your fork]
- **Screenshots/logs:** [If applicable]
- **My findings:** [What you discovered during reproduction]

---

## Solution Approach

### Analysis

The root cause is that `ScatterPlotChartServlet.service()` resolves `demographicNo` from either a request parameter or a session-bound `EctSessionBean`, but trusts the request parameter unconditionally whenever it's present — it never validates that the currently authenticated user is authorized to view the specific patient referenced by that ID. The value flows directly into `generateResult()`, which queries `MeasurementDao` and returns real patient data with no intervening permission check. Unlike other parts of the codebase, this servlet never calls `SecurityInfoManager.hasPrivilege()`, which is the established pattern for enforcing per-resource access control elsewhere in the project.

### Proposed Solution

Add a check immediately after `demographicNo` is resolved and before any chart-generation logic runs. Following the pattern already used in this file for other Spring-managed beans (`SpringUtils.getBean(MeasurementDao.class)`), the fix should call `SpringUtils.getBean(SecurityInfoManager.class).hasPrivilege(...)`. If the check fails, the servlet should return `403 Forbidden` immediately and skip chart generation entirely.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** `ScatterPlotChartServlet` renders a patient's measurement chart based on a client-supplied `demographicNo`, without ever verifying that the requesting, authenticated user is authorized to view that specific patient's data — allowing any logged-in user to view any patient's chart by changing the ID.

**Match:** The project's established pattern for authorization checks is `SecurityInfoManager.hasPrivilege(loggedInInfo, object, permission, ...)`, retrieved via `SpringUtils.getBean(...)` — the same dependency-lookup style already used in this file for `MeasurementDao` and `MeasurementTypeDao`. [Once you've opened `SecurityInfoManager.java` and found another servlet/action that calls `hasPrivilege()` correctly, add the specific file/line here as your reference example.]

**Plan:**
1. In `ScatterPlotChartServlet.service()`, after `demographicNo` is finalized, insert a call to `SecurityInfoManager.hasPrivilege()`.
2. If the check fails, call `httpServletResponse.sendError(HttpServletResponse.SC_FORBIDDEN)` and `return` immediately — no chart generation should occur.
3. Add or update tests covering both the authorized and unauthorized cases (see Testing Strategy section).


**Implement:** [Link to your branch/commits as you work]

**Review:** [Self-review checklist - does it follow the project's contribution guidelines?]

**Evaluate:** [How will you verify it works?]

---

## Testing Strategy

### Unit Tests (I have not unit tests currently)

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

I haven't tested anything in this phase

---

## Implementation Notes

### Week [5] Progress

This is my second issue and currently I am just trying to study the issue and analyse it.
I am in Phase 1 second cycle

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** None right now
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- Claude code 
