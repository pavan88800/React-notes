## Claude Code Tips

---

### 1. Run from the project root

When you first launch Claude Code, run it from your project's root directory. Claude works best when it can see your whole project structure: configs, `package.json`, and your `CLAUDE.md`.

```bash
cd path/to/your-project
claude
```

Running from the root lets Claude Code see your whole project (configs, dependencies, **`CLAUDE.md`**, and how files connect), so it understands the codebase instead of guessing from a single folder.

---

### 2. Run `/init` immediately

Running `/init` creates a **`CLAUDE.md`** file in your project root. Think of it as Claude's long-term memory for your project. It's a starting point, so edit it to add your own rules and conventions.
