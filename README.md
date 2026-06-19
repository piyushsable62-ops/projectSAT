You are completing and fixing "Pathway", a student academic tracker SPA built 
in a single HTML file using React 18 (UMD/Babel), Tailwind CSS CDN, and 
inline SVG icons. No build step. All state lives in React; persistence uses 
localStorage. The full current source code is provided above. Implement every 
item below exactly as specified.

=======================================================================
SECTION 1 — AUTHENTICATION & ONBOARDING
=======================================================================

1.1 AUTH GATE
- App component must check a persisted auth state on mount.
  On load: read localStorage key "pathway-user". If it exists and is valid 
  JSON with a `name` field, parse it and set as user state, skip Login.
  If absent or invalid, render <Login> full-screen instead of the main app.
- After successful login (onLogin callback), JSON.stringify the user object 
  and write it to localStorage["pathway-user"], then show the main app.
- The sidebar logout button must clear localStorage["pathway-user"] and 
  reset auth state back to the Login screen (no page reload needed).

1.2 GOOGLE OAUTH — GRACEFUL FALLBACK
- Keep the existing Google Identity Services button flow as-is.
- Add a clearly separated "or" divider below the Google button container.
- Below it, add a simple email + password form (two inputs + "Sign in" 
  button) that works as a LOCAL fallback — no real backend needed. 
  On submit, if both fields are non-empty, treat it as a successful login 
  and proceed to the profile setup step with email pre-filled.
- Add a "Demo / Guest" button (no credentials needed) that pre-fills 
  the profile step with name="Demo Student", email="demo@pathway.app" 
  and skips directly to the main app with default profile values.
- The profile setup step (step === "profile") must auto-generate the 
  avatar initials from the name field in real time and display a live 
  preview of the avatar circle next to the name input.

1.3 PERSISTENT PROFILE
- On "Get Started" in the profile step, save the full user object to 
  localStorage["pathway-user"].
- In SettingsModule, the "Save Changes" button must actually update the 
  user state (call setUser) AND re-save to localStorage. Show a green 
  success toast for 2 seconds after saving.

=======================================================================
SECTION 2 — DATA PERSISTENCE (localStorage)
=======================================================================

All mutable data must survive page refresh. Use these localStorage keys:

  "pathway-assignments"  → array of assignment objects (init from ASSIGNMENTS)
  "pathway-cas"          → array of CA objects (init from INITIAL_CAS)
  "pathway-notifications"→ array of notification objects
  "pathway-community-messages" → array of chat message objects

Helper pattern to use for each:
  const [items, setItems] = usePersistedState("pathway-assignments", ASSIGNMENTS);

Implement usePersistedState(key, defaultValue) as a custom hook:
  function usePersistedState(key, defaultValue) {
    const [state, setState] = useState(() => {
      try {
        const raw = localStorage.getItem(key);
        return raw ? JSON.parse(raw) : defaultValue;
      } catch { return defaultValue; }
    });
    useEffect(() => {
      try { localStorage.setItem(key, JSON.stringify(state)); } catch {}
    }, [state, key]);
    return [state, setState];
  }

Replace all useState calls for assignments, cas, notifications, and 
community messages with usePersistedState.

=======================================================================
SECTION 3 — ASSIGNMENTS MODULE
=======================================================================

3.1 ADD ASSIGNMENT MODAL
The "Add Assignment" button must open a modal (similar in style to AddCAModal).
The modal form must have:
  - Title (text input, required)
  - Subject (select from SUBJECTS, required)
  - Description (textarea)
  - Due Date (date input, required)
  - Priority (select: High / Medium / Low, default Medium)
  - Status (select: Pending / In Progress / Completed, default Pending)
  - "Save" and "Cancel" buttons
On save, push the new assignment into the assignments array with a unique 
id (Date.now()). Close the modal and show a brief success toast.

3.2 EDIT & DELETE IN DETAIL MODAL
In the existing assignment detail modal:
  - "Edit" button must switch the modal into an edit mode where all fields 
    become inputs pre-filled with current values. A "Save Changes" button 
    replaces "Edit" and commits the updates.
  - "Delete" button must show a confirm dialog (browser confirm() is fine), 
    then remove the item from state and close the modal.

3.3 SEARCH / FILTER
Wire the global search input (in Topbar) so that when on the assignments 
page it filters the assignments list by title or subject name (case-insensitive).

=======================================================================
SECTION 4 — CA MODULE
=======================================================================

4.1 SUBJECT PROGRESS BAR
The progress bar in each subject card in the left panel must calculate 
completion correctly: a CA is "completed" if its date is strictly before 
today's date (use new Date() for today, not the hardcoded "2026-06-11").

4.2 CA STATUS — USE REAL TODAY
Replace all occurrences of `daysUntil` that use the hardcoded base date 
"2026-06-11" with a function that uses the real current date:
  function daysUntil(dateStr) {
    const today = new Date();
    today.setHours(0,0,0,0);
    const target = new Date(dateStr);
    target.setHours(0,0,0,0);
    return Math.ceil((target - today) / (1000*60*60*24));
  }

4.3 EDIT CA
In ExamDetailModal, add an "Edit" button next to "Delete CA". 
Clicking it closes the detail modal and opens AddCAModal pre-filled 
with the existing CA's values (pass initialValues prop). 
On save of an edit, update the matching CA in the cas array by id.

=======================================================================
SECTION 5 — CALENDAR MODULE
=======================================================================

5.1 DYNAMIC MONTH NAVIGATION
Replace the hardcoded "June 2026" calendar with a fully dynamic calendar:
  - State: [viewYear, viewMonth] initialized to current real date.
  - "Prev" / "Next" chevron buttons must decrement/increment the month 
    (handling year rollover correctly).
  - The displayed month label must reflect viewYear/viewMonth.
  - The grid cells must be generated from viewYear/viewMonth (correct 
    number of days, correct start-day offset using new Date(y,m,1).getDay()).
  - Events must be matched against the currently displayed month/year.
  - "Today" highlight must use the real current date.

5.2 ADD EVENT FROM CALENDAR
Add a "+" floating button (bottom-right of the calendar grid). 
Clicking it opens a simple modal to add a custom event:
  - Title (text, required)
  - Date (date input, defaulting to today, required)
  - Type (select: Assignment / CA / Study / Personal)
  - Color auto-assigned by type.
On save, push the new event into a "pathway-events" persisted state array 
and it should appear on the calendar.

5.3 DAY VIEW
When a day cell is clicked (even in month view), show a small popover or 
side panel listing all events for that day with their time and type badges.

=======================================================================
SECTION 6 — PYQ MODULE
=======================================================================

6.1 DOWNLOAD FUNCTIONALITY
Replace the fake toast-only download with an actual file generation:
  - On "Download" click, generate a simple text file (.txt) from the 
    MOCK_QUESTIONS data for that semester with the paper title as heading.
  - Use the Blob + URL.createObjectURL + a hidden <a> tag pattern to 
    trigger a real browser download. The filename format:
    "Pathway_PYQ_Sem{N}_{Term}_{Year}_{Variant}.txt"
  - The toast can remain as an additional UX indicator.

6.2 UPLOAD FLOW
The "Drag & drop" upload zone at the bottom must be functional:
  - Accept .pdf files only (input accept=".pdf").
  - On file select, add a new entry to a persisted "pathway-pyq-uploads" 
    array with { id, name, size, semester: activeFirstSemester, 
    year: activeYear, term: activeTerm, uploader: "You", downloads: 0 }.
  - Show uploaded files in a new "Community Uploads" section below the 
    existing semester blocks, with a delete button for files uploaded by "You".

=======================================================================
SECTION 7 — COMMUNITY MODULE
=======================================================================

7.1 SUBJECT ROOM MESSAGES
Each subject room (math, phy, chem, c, eee) should have its own message 
thread separate from "general". Currently all messages are tagged 
room:"general". 
  - CommunityChat must filter messages by `m.room === room`.
  - Seed 1-2 starter messages per subject room in COMMUNITY_MESSAGES 
    (e.g. a welcome message from a bot user { name: "Pathway Bot", avatar: "PB", color: "#9B6BFF" }).
  - When a user sends a message in a room, tag it with that room's id.

7.2 FILE ATTACHMENT IN CHAT
The paperclip button must open a hidden file input (accept any file). 
On file select, add the message with an attachment preview card 
(filename + size) — no actual upload, just show it in the chat.

7.3 DOUBT POSTING
When "Post Doubt" is clicked in the new doubt modal, actually push a 
new doubt object into a persisted doubts array (not just close the modal). 
The new doubt should appear at the top of the doubts list with status 
"Unsolved" and 0 replies.

=======================================================================
SECTION 8 — NOTIFICATIONS
=======================================================================

8.1 AUTO-GENERATE DEADLINE NOTIFICATIONS
On app load (useEffect with empty deps), scan assignments and CAs for 
items due within 2 days. For each one not already in the notifications 
array (check by a generated id like "deadline-{itemId}"), add a new 
notification object with type "deadline", priority "high", read: false.

8.2 CLEAR NOTIFICATIONS
Add a "Clear all read" button that removes all notifications where 
read === true.

=======================================================================
SECTION 9 — DARK MODE FIXES
=======================================================================

Fix all dark mode visual breaks. The modal backgrounds use hardcoded 
"bg-white" — replace with a dark-compatible pattern:

Pattern: instead of className="bg-white", use a CSS variable-based class.
Add this to the <style> block:
  .surface { background: var(--card); }

Apply className="surface" to every modal panel, every card, every 
dropdown that currently uses bg-white.

Additional dark mode rules to add in <style>:
  html.dark .surface { background: var(--card); color: var(--ink); }
  html.dark input, html.dark select, html.dark textarea { 
    background: rgba(255,255,255,0.06); 
    border-color: var(--line); 
    color: var(--ink); 
  }
  html.dark .glass { 
    background: rgba(33,29,79,0.7); 
    border: 1px solid rgba(255,255,255,0.08); 
  }

Audit every modal (assignment detail, CA detail, AddCAModal, calendar 
event modal, doubt modal, PDF preview modal) and apply "surface" class.

=======================================================================
SECTION 10 — MOBILE RESPONSIVENESS
=======================================================================

10.1 MOBILE SIDEBAR
On viewport width < 768px (md breakpoint):
  - The sidebar must NOT be rendered in the normal flow (remove ml-64/ml-20 
    from <main> on mobile).
  - Instead, render a bottom navigation bar with icons only for the 5 most 
    important pages: dashboard, assignments, cas, community, notifications.
  - The full sidebar slides in as an overlay when a hamburger button 
    (top-left of the mobile topbar) is tapped.
  - An overlay div (bg-black/50) behind the open sidebar closes it on tap.

Implement with a useMobile hook:
  function useMobile() {
    const [isMobile, setIsMobile] = useState(window.innerWidth < 768);
    useEffect(() => {
      const handler = () => setIsMobile(window.innerWidth < 768);
      window.addEventListener('resize', handler);
      return () => window.removeEventListener('resize', handler);
    }, []);
    return isMobile;
  }

10.2 TOPBAR ON MOBILE
On mobile, the search bar should be hidden by default but a search 
icon button should expand it full-width as an overlay input (with an X 
to dismiss).

10.3 MODAL SIZING
All modals must use `max-h-[90vh] overflow-y-auto` and on mobile must 
be bottom-sheet style (align-bottom, rounded-t-3xl, no bottom radius, 
full-width). Add class logic:
  isMobile 
    ? "fixed inset-x-0 bottom-0 rounded-t-3xl max-h-[90vh]" 
    : "relative rounded-3xl max-w-lg w-full"

=======================================================================
SECTION 11 — DASHBOARD ENHANCEMENTS
=======================================================================

11.1 REAL STATS
All stat cards must reflect real data from the persisted state arrays:
  - "Total Assignments" → assignments.length
  - "Completed" → filtered count
  - "Pending" → filtered count  
  - "Upcoming CAs" → CAs with daysUntil(date) >= 0
  - "Study Progress" → (completed assignments / total) * 100, rounded
  - "Day Streak" → calculate from localStorage["pathway-streak"]:
      On every app load, check if today's date string matches 
      localStorage["pathway-last-visit"]. If it's a new day, 
      increment streak. If more than 1 day has passed, reset to 1. 
      Store in localStorage["pathway-streak"] and ["pathway-last-visit"].

11.2 SUBJECT PROGRESS WIDGET
In the "Subject-wise Progress" card, the percentages must derive from 
the real assignments data (completed / total per subject), not from 
the hardcoded fallback array.

=======================================================================
SECTION 12 — STUDY MATERIAL MODULE
=======================================================================

12.1 ADD MATERIAL
The "Add material" card (dashed border) must open a modal with:
  - Title (text, required)
  - Type (select: Book / Notes / Video / Link)
  - URL or description (text)
  - Subject (already set to activeSubject)
On save, push to a persisted "pathway-materials" array and render 
the new item in the appropriate section.

12.2 VIDEO PLACEHOLDER → LINK
Replace the two static black video cards with real YouTube embed 
links (just 2 hardcoded playlist URLs per subject are fine). 
Use an <iframe> with loading="lazy" inside the aspect-ratio div, 
showing a poster until clicked.

=======================================================================
SECTION 13 — GLOBAL UX POLISH
=======================================================================

13.1 TOAST SYSTEM
Implement a global toast system (small notification pop-up, bottom-right):
  - ToastContext / useToast hook with showToast(message, type) method.
    type: "success" | "error" | "info" | "warning"
  - Colors: success=#7C9885, error=#FF6B5B, info=#5B8DEF, warning=#F4A259
  - Toast auto-dismisses after 3 seconds with a fade-out animation.
  - A ToastContainer component renders all active toasts stacked.
  - Replace all existing alert() and setToast calls with showToast().

13.2 LOADING STATES
Add a skeleton loader pattern (animated shimmer divs) for the 
initial render of each page's main list/grid. Show skeletons for 
~400ms on first visit to a page (use a per-page "visited" ref).

13.3 EMPTY STATES
Every list/grid that could be empty must show a contextual empty state:
  - Assignments: "No assignments yet. Add your first one!"
  - CAs: already handled — keep existing, ensure it also shows for 
    when all CAs are filtered out
  - Notifications: "You're all caught up! ✨"
  - Doubts: "No doubts in this filter. Be the first to ask!"
  - Resources: "No resources shared yet. Upload the first one!"

13.4 CONFIRMATION DIALOGS
Replace all browser confirm() calls with a custom ConfirmDialog 
component that matches the app's visual style:
  - Appears as a small modal with title, message, "Cancel" and 
    "Confirm" (red) buttons.
  - Implement useConfirm() hook that returns a confirm(message) 
    function returning a Promise<boolean>.

13.5 PAGE TRANSITIONS
Wrap each page render in a div with key={page} so React remounts 
it. Apply the existing .float-in animation class to that wrapper 
div so every page switch animates in smoothly.

=======================================================================
SECTION 14 — SETTINGS MODULE COMPLETION
=======================================================================

14.1 NOTIFICATION PREFERENCES
Add a "Notification Preferences" section with toggles (styled 
checkbox-as-toggle using CSS) for:
  - Deadline reminders (default: on)
  - CA reminders (default: on)  
  - Community activity (default: on)
  - System announcements (default: on)
Persist toggle states in localStorage["pathway-notif-prefs"].

14.2 DATA MANAGEMENT
Add a "Data & Privacy" section with:
  - "Export my data" button: creates and downloads a JSON file 
    containing all persisted data (assignments, CAs, messages, etc.)
    using the same Blob download pattern as PYQs.
  - "Reset all data" button: clears all "pathway-*" localStorage 
    keys after a ConfirmDialog, then reloads the page.

14.3 ACCOUNT SECTION
Add "Sign out" button (styled in red/coral) that calls the logout 
handler from Section 1.1.

=======================================================================
SECTION 15 — CODE QUALITY & STRUCTURE
=======================================================================

15.1 Remove the hardcoded date "2026-06-11" from EVERY occurrence 
and use the real-date daysUntil() from Section 4.2 everywhere.

15.2 The GOOGLE_CLIENT_ID constant — add a comment above it:
  // SETUP: Replace with your Google OAuth Client ID from 
  // https://console.cloud.google.com/apis/credentials
  // Add your hosting domain to "Authorized JavaScript origins".
  // For local testing, add http://localhost to origins.

15.3 Ensure no console errors on initial render. Fix any key prop 
warnings (e.g., map without unique keys in icon renderer).

15.4 The SVG icon system uses React.createElement — ensure the 
key prop is passed as a number string to avoid warnings. Current 
code already does this; verify no regressions.

=======================================================================
DELIVERABLE
=======================================================================

Output the COMPLETE updated single HTML file. Do not omit any section.
Do not truncate with "// rest of code remains the same" — every line 
must be present. The file must:
  ✓ Open in Chrome/Firefox with no console errors
  ✓ Work fully offline (no network calls except Google OAuth + CDN)
  ✓ Persist all user data across page refreshes via localStorage
  ✓ Be fully functional on both desktop (1280px+) and mobile (375px)
  ✓ Have working dark mode for every component
  ✓ Have no placeholder/demo buttons that do nothing
  ✓ Use the real current date (not hardcoded 2026-06-11) everywhere