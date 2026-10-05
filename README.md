# Code Buddy

Your friendly helper for Frappe/ERPNext projects. Ask in plain language and Code Buddy reviews your code, finds security and performance problems, checks upgrade readiness, tidies messy apps, and reviews or designs your servers.

## What it can do

| Skill | Try asking |
|---|---|
| **Code review** | "Review this pull request" · "Is this doctype controller okay?" |
| **Security audit** | "Audit the APIs in my app for security issues" |
| **Performance review** | "Submitting a Sales Invoice takes 20 seconds, why?" |
| **Upgrade check** | "Is my custom app ready for v16?" |
| **App restructure** | "This controller is 1,200 lines, clean it up" |
| **Customization cleanup** | "Move all UI-made custom fields and scripts into my app" |
| **Pattern conversion** | "Convert the raw SQL in this report to the query builder" |
| **Server review** | "Review my production server setup" |
| **Server architecture** | "Client has 150 users with manufacturing and payroll, suggest a server setup" |
| **Team standards** | "What's the rule for whitelisted methods?" |

Covers Frappe/ERPNext v14, v15 and v16, plus Python, JavaScript, SQL, Git workflow, and dev/staging/production servers (bench, Docker, nginx, MariaDB, Redis).

## Install

**From the Claude directory:** search for "Code Buddy" and select Add.

**From GitHub (Claude Code):**
```
/plugin marketplace add Satya8848/code-buddy
/plugin install code-buddy@code-buddy
```

## Use your team's own standards

Code Buddy ships with sensible best-practice standards. To use your team's rules instead:

1. Create a repo with a `standards/` folder. Copy the files from `skills/team-standards/references/` as a starting point and edit them.
2. Add a `CODE_BUDDY.md` file to each project:
   ```
   standards_repo: your-org/your-standards
   branch: main
   path: standards
   ```
3. Connect GitHub in Claude so Code Buddy can read the repo. Any file you don't provide falls back to the defaults.

## Privacy & data

Code Buddy has no servers of its own and sends nothing anywhere. It only reads the code, files and configs you share in your conversation, and (if you configure one) your standards repo through your own GitHub connection. Secrets found in configs are masked in its output.

## Notes

- Server sizes and cost figures are starting estimates; validate with load testing and your provider's pricing.
- The upgrade checklist is based on Frappe's official migration guides; always cross-check the latest release notes.

## License

MIT © Satyabrata Panda
