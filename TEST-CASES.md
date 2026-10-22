# ShiftSwap: Test Cases & Bug Reports

Manual testing of the ShiftSwap prototype.

- **Environment:** Google Chrome (desktop), `index.html` opened locally, data in `localStorage`
- **Test data:** `ShiftSwap.Project.xlsx` (29 employees, 6 departments) and the 4 default shifts in `index.html`
- **Before each run:** DevTools (F12) → Console → `localStorage.clear()`, then refresh

**Summary:** 19 test cases · 15 pass · 3 fail (known open bugs) · 1 not testable yet · 6 bugs found and fixed

---

## 1. Shift board & workflow

| ID | Test case | Steps | Expected result | Status |
| --- | --- | --- | --- | --- |
| TC-01 | Board loads default shifts | Open the app | 4 shift cards appear, each with department, day, time and full name of the poster | ✅ Pass |
| TC-02 | Claim an open shift | Click **Claim** on an *Open* shift | Status becomes *Claimed*, card shows "Claimed by Traian", **Unclaim** button appears | ✅ Pass |
| TC-03 | Unclaim a claimed shift | Claim a shift, then click **Unclaim** | Status returns to *Open*, "Claimed by" line disappears | ✅ Pass |
| TC-04 | Employees can't approve | Manager view **off**, claim a shift | No **Approve** button is shown | ✅ Pass |
| TC-05 | Manager approves a claim | Claim a shift, turn Manager view **on**, click **Approve** | Status becomes *Approved*, card turns grey, only **Unapprove** is left | ✅ Pass |
| TC-06 | Manager unapproves | Approve a shift, click **Unapprove** | Status returns to *Open* and the claim is cleared | ✅ Pass (see BUG-10) |
| TC-07 | State survives refresh | Approve a shift, refresh the page | The shift is still *Approved* | ✅ Pass |
| TC-08 | Tabs switch | Click **Schedule**, then **My Shifts**, then **ShiftSwap** | The right panel shows each time, active tab is highlighted | ✅ Pass |

## 2. Saved data (`localStorage`)

| ID | Test case | Steps | Expected result | Status |
| --- | --- | --- | --- | --- |
| TC-09 | Corrupted saved data | Console: `localStorage.shifts = "{broken"`, refresh | App still shows the 4 default shifts; a warning is logged in the Console | ✅ Pass |
| TC-10 | Old data after an update | Save shifts, change `defaultShifts` and bump `DATA_VERSION`, refresh | Old saved shifts are dropped, the new defaults load | ✅ Pass |

## 3. Test data (Excel)

| ID | Test case | Steps | Expected result | Status |
| --- | --- | --- | --- | --- |
| TC-11 | Department names match | Compare department names in *Employees* and *Schedule* | Identical spelling in both sheets (6 departments) | ✅ Pass (after BUG-04 fix) |
| TC-12 | Every employee is scheduled | Compare names in both sheets | All 29 employees appear in *Schedule* under their own department, nobody extra | ✅ Pass |
| TC-13 | Email format | Check every email | Every email follows `First.Last@gmail.com` | ✅ Pass (after BUG-05 fix) |
| TC-14 | Shift format | Check every schedule cell | Every cell is `OFF` or like `8AM-4PM` | ✅ Pass |
| TC-15 | App shifts match schedule | Compare the 4 default shifts with *Schedule* | Each poster really works that shift, in that department | ✅ Pass (after BUG-06 fix) |

## 4. Known gaps (open)

| ID | Test case | Steps | Expected result | Actual result | Status |
| --- | --- | --- | --- | --- | --- |
| TC-16 | Only the claimer can unclaim | User A claims a shift, user B clicks **Unclaim** | B is not allowed | Anyone can unclaim | ❌ Fail (BUG-07) |
| TC-17 | Manager actions need a manager | As an employee, tick **Manager view** | Not allowed without a manager login | Anyone can become manager | ❌ Fail (BUG-08) |
| TC-18 | Data is shown as text, not run as code | Console: save a shift whose `dept` is `<img src=x onerror=alert(1)>`, refresh | Department shows as plain text | The HTML runs | ❌ Fail (BUG-09) |
| TC-19 | Same first name, different people | Log in as Lucas Martin (E015), open *My Shifts* | Only Lucas Martin's shifts, never Lucas Baker's (E025) | Logins and *My Shifts* not built yet | ⏳ Not testable yet |

---

## Bug reports

| ID | Bug | Severity | Status |
| --- | --- | --- | --- |
| BUG-01 | Old saved data hides code changes | High | ✅ Fixed |
| BUG-02 | Corrupted saved data gives an empty board | High | ✅ Fixed |
| BUG-03 | Extra `}` stops the whole app | Critical | ✅ Fixed |
| BUG-04 | "House Keeping" vs "Housekeeping" | Medium | ✅ Fixed |
| BUG-05 | Typo in an employee email | Low | ✅ Fixed |
| BUG-06 | Default shifts don't match the schedule | Medium | ✅ Fixed |
| BUG-07 | Anyone can unclaim anyone's shift | High | 🔓 Open |
| BUG-08 | Manager view has no access control | High | 🔓 Open (needs logins) |
| BUG-09 | Shift data is inserted as raw HTML | High | 🔓 Open |
| BUG-10 | Unapprove also removes the claim | Low | 💬 Design decision |

### BUG-01: Old saved data hides code changes
- **Steps:** Use the app (data is saved), change the shifts or statuses in the code, refresh.
- **Expected:** The app shows the updated shifts.
- **Actual:** The browser keeps showing the old saved values; buttons seem to "not change anything".
- **Cause:** `getShifts()` always preferred whatever was in `localStorage`, even if it came from an older version of the code.
- **Fix:** Added a `DATA_VERSION`. When it changes, old saved data is removed. Verified by TC-10.

### BUG-02: Corrupted saved data gives an empty board
- **Steps:** Console: `localStorage.shifts = "{broken"`, refresh.
- **Expected:** The app recovers.
- **Actual:** Empty board; Console shows `SyntaxError` from `JSON.parse`.
- **Fix:** Wrapped reading in `try / catch`; on error, reset to the default shifts and log a warning. Verified by TC-09.

### BUG-03: Extra `}` stops the whole app
- **Steps:** Open the app after editing `getShifts()`.
- **Expected:** Shifts appear.
- **Actual:** Header and tabs show, but no shifts. Console: `Uncaught SyntaxError: Unexpected token '}'`.
- **Cause:** A leftover closing brace from the old version of the function, so the browser could not run any of the script.
- **Fix:** Removed the brace, then re-ran TC-01 to TC-10.

### BUG-04: "House Keeping" vs "Housekeeping"
- **Where:** `ShiftSwap.Project.xlsx`, sheet *Schedule*, cell C11.
- **Expected:** Same department name as in *Employees* and the app ("Housekeeping").
- **Actual:** "House Keeping", later "HouseKeeping" (capital K). Filtering by department would miss all 6 Housekeeping employees.
- **Fix:** Renamed to "Housekeeping". Verified by TC-11.

### BUG-05: Typo in an employee email
- **Where:** *Employees*, Lucas Baker (E025): `Lucas.Bake@gmail.com`.
- **Fix:** Changed to `Lucas.Baker@gmail.com`. Verified by TC-13.

### BUG-06: Default shifts don't match the schedule
- **Actual:** Shifts posted in the app didn't exist in the schedule (Maria posted Friday 6PM–11PM but works 8AM–4PM; "Luis" is not an employee; times written in different formats).
- **Fix:** Replaced the 4 default shifts with real shifts from the schedule, using full names and the Excel time format. Verified by TC-15.

### BUG-07: Anyone can unclaim anyone's shift
- **Actual:** The **Unclaim** button shows for every claimed shift, and `unclaimShift()` doesn't check who claimed it.
- **Planned fix:** Show **Unclaim** and allow it only when `claimedBy` is the logged-in user.

### BUG-08: Manager view has no access control
- **Actual:** The *Manager view* checkbox is available to everyone. Fine for a prototype, not for real use.
- **Planned fix:** Role-based logins; manager actions only for users with the *Manager* role.

### BUG-09: Shift data is inserted as raw HTML
- **Actual:** Cards are built with `innerHTML`, so HTML inside a shift's data runs (TC-18). Becomes a real risk once data is imported from Excel or typed in by users.
- **Planned fix:** Escape the data before showing it, or build cards with `textContent`.

### BUG-10: Unapprove also removes the claim
- **Actual:** **Unapprove** sends the shift back to *Open* and clears who claimed it.
- **Question:** Should it go back to *Claimed* instead, so the manager can reconsider without the employee claiming again? To decide before logins are added.
