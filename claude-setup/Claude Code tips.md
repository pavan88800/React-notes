## Claude Code Tips

---

### 1. Run from the project root

When you first launch Claude Code, run it from your project's root directory. Claude works best when it can see your whole project structure: configs, `package.json`, and your `CLAUDE.md`.

```bash
cd path/to/your-project
claude
```

---

### 2. Run `/init` immediately

Running `/init` creates a **`CLAUDE.md`**

What does it do? Claude goes through your codebase, analyzes it, and then creates the initial **`CLAUDE.md`** file.

Right now it's just looking at the codebase, looking for the architecture, and basically creating the **`CLAUDE.md`** file.

If a CLAUDE.md already exists, `/init` improves on it instead of starting from scratch.

---

### What is the use of the `CLAUDE.md` file?

**`CLAUDE.md`** tells Claude your project's context and rules at the start of every session.

---

### 3. Check memory with `/memory`

Running `/memory` shows where Claude's memory is stored for your project.

- **Auto-memory:** When `true`, Claude automatically saves useful things it learns while working.
- **Project instructions** (`./CLAUDE.md`): Rules for **this project only**. Usually committed to git and shared with the team.
- **User instructions** (`~/.claude/CLAUDE.md`): Your personal preferences. Applies to **all your projects**.
- **Open auto-memory folder:** Opens the notes Claude saved automatically, so you can read, edit, or delete them.

|                         | Who writes it              | Applies to        |
| ----------------------- | -------------------------- | ----------------- |
| Project **`CLAUDE.md`** | You (team rules)           | This project      |
| User **`CLAUDE.md`**    | You (personal preferences) | All your projects |
| Auto-memory             | Claude                     | This project      |

---

### 4. keep **`CLAUDE.md`** concise

- It gets prepended to every prompt. A bloated files eats your context window and degrades performance. Focus on what matters.

---

### 5. Structure: What, Domain, Validation

- what = tech stack. **`Domain context`** = what each part does. **`Validation steps`** = tests, linters, type checkers.

---

## Keyboard Shortcuts

### 6. `Shift+Tab` toggles modes

Press `Shift+Tab` to cycle through modes:

- **Auto** (default): Claude works without asking. A safety check blocks risky actions.
- **Manual**: Claude asks before every edit.
- **Accept edits**: Claude edits files without asking.
- **Plan**: Claude only plans and doesn't change any code. Use this before big changes.

### 7. `Esc` interrupts

Press `Esc` to stop Claude mid-response. It keeps the work done so far, so you can redirect it.

### 8. Double `Esc` clears input

When your prompt has text, press `Esc` twice to clear it. The draft is saved, so press `↑` to get it back.

### 9. Double `Esc` on empty input = rewind

When your prompt is empty, press `Esc` twice to open the **rewind** menu. It lets you go back to an earlier point and restore your code, the conversation, or both.

### 10. Screenshot and drag

Take a screenshot (for example, a UI bug or an error) and drag the image file into the Claude Code terminal. Claude can see and understand the image.

You can also paste an image from your clipboard with `Ctrl+V` (`Cmd+V` in iTerm2).

### 11. Add context to screenshots

Don't send a screenshot alone. Tell Claude what to look at and what you want.

- ❌ `[Image #1]`
- ✅ `[Image #1] The submit button overlaps the footer on mobile. Fix the spacing.`
