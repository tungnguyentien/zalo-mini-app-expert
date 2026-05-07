# Zalo Mini App Skills

The `skills/` directory is standardized as **a single skill bundle** for Zalo Mini App.

## Standard structure

```
skills/
├── README.md
└── zalo-mini-app/
    ├── SKILL.md
    └── references/
        ├── agent-instructions.md
        ├── general.md
        ├── zmp-sdk-api.md
        ├── open-apis.md
        ├── zaui-react.md
        ├── app-config.md
        ├── authentication.md
        ├── payment.md
        ├── permissions.md
        ├── devtools.md
        ├── deployment.md
        ├── compatibility.md
        ├── troubleshooting.md
        ├── best-practices.md
        └── use-cases.md
```

## How to use

### With Cursor
Cursor can load these files as project context to provide higher-accuracy assistance for Zalo Mini App development.

**Recommendation**: add `skills/` to your project AI context (e.g. via `.cursorrules`) so the agent can reference it when needed.

### With Claude Code

Claude Code auto-discovers skills via the `SKILL.md` file. Install the bundle in one of two locations:

**1. Per-project** (recommended when actively working on a Zalo Mini App):

```bash
# From your project root
mkdir -p .claude/skills
cp -r /path/to/zalo-mini-app-expert/zalo-mini-app .claude/skills/
```

**2. Global** (available across all projects):

```bash
mkdir -p ~/.claude/skills
cp -r /path/to/zalo-mini-app-expert/zalo-mini-app ~/.claude/skills/
```

Once installed, just ask Claude Code anything related to Zalo Mini App — it will load `SKILL.md` and pull in the relevant files from `references/` on demand. Examples:

- *"Integrate Zalo login into my Mini App"* → loads `authentication.md` + `use-cases.md`
- *"Call an Open API to fetch user info"* → loads `open-apis.md`
- *"Build and deploy my Mini App to production"* → loads `deployment.md`

You can also point at a specific file: *"Read `zalo-mini-app/references/payment.md` and help me wire up payments."*

### With Claude.ai (web / desktop)

Claude.ai doesn't load the skill format automatically. Two practical options:

- **Project knowledge**: create a new Project and upload `SKILL.md` along with the `references/` files you need into the Knowledge section.
- **Paste on demand**: copy the reference file matching your question (see the Quick reference table below) and paste it at the start of the conversation.

### With other AI tools (Cursor, Copilot, …)

Follow the *With Cursor* guidance above — the general rule is to add the skill directory to the tool's AI context configuration.

### Quick reference

| Goal | Reference |
|-------------|----------|
| Get started with Zalo Mini App | `zalo-mini-app/references/general.md` |
| Implement authentication | `zalo-mini-app/references/authentication.md` + `.../use-cases.md` |
| Implement payments | `zalo-mini-app/references/payment.md` + `.../use-cases.md` |
| Call server-side APIs | `zalo-mini-app/references/open-apis.md` |
| Use ZaUI components | `zalo-mini-app/references/zaui-react.md` |
| Troubleshoot issues | `zalo-mini-app/references/troubleshooting.md` |
| Check version support | `zalo-mini-app/references/compatibility.md` |
| End-to-end code examples | `zalo-mini-app/references/use-cases.md` |

## Official references

- [Zalo Mini App Documentation](https://mini.zalo.me/documents/)
- [Zalo Mini App API Reference](https://mini.zalo.me/documents/api/)
- [Zalo Mini App Community](https://mini.zalo.me/community/)
- [Zalo for Developers](https://developers.zalo.me/)

## Notes

- This bundle is derived from Zalo Mini App’s official documentation and common engineering practices.
- Always verify against the latest official docs, as APIs and policies may change.
- If any content becomes outdated, update the corresponding reference file in `zalo-mini-app/references/`.
