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
