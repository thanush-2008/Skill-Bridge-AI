# SkillBridge AI

**Show what you can build, not just what you list.**

SkillBridge AI helps students see how ready they are for internships at companies in India and around the world. Paste a resume, and the app finds your skills, scores you against company roles, shows what is missing (with a project idea for each gap), and lets you ask an AI career assistant questions like *"Can I get an internship at Infosys?"*.

> Prototype built for a hackathon. The company requirements are **typical examples, not live job postings**. Always check a company's careers page before relying on them.

## Features

- **Resume reading:** paste text or upload a PDF/TXT; skills are extracted automatically.
- **Company explorer:** 134 companies, 20 countries, 15 sectors and 344 role profiles (IT services, big tech, startups, banks and fintech, consulting, semiconductors, automotive, telecom, government and more).
- **Smart search and filters:** search by company, country, sector, role or skill (for example `india fintech`), and filter by sector and country.
- **Match score and skill gap:** a weighted match percentage for every role, with core skills marked and a suggested project for each missing skill.
- **Skill verification:** submit a public GitHub repo and the app checks it (relevance, README, tests, size) to mark a skill as verified.
- **AI career chatbot:** ask about a company or role using your resume. Works offline with no key, and gives natural-language answers when an API key is added.
- **Accounts and profile:** register, log in, and save your college, degree, links and bio.
- **Glassmorphism UI:** responsive, animated glass interface.

## Tech stack

| Part | Technology |
| --- | --- |
| Backend | Python, Flask |
| Database | SQLite |
| Frontend | HTML, CSS and plain JavaScript (single page, no framework) |
| Data | `roles.json` (skills, role types, companies) |
| AI (optional) | Claude API via the Anthropic Messages API |
| PDF reading | pypdf |

## Project structure

```
skillbridge/
├── app.py              # Backend: auth, skills, scoring, search, GitHub check, chatbot
├── roles.json          # Skills, role types and the company list
├── requirements.txt    # Python dependencies
├── static/
│   └── index.html      # The whole frontend, including the chat widget
└── skillbridge.db      # SQLite database (created automatically, not committed)
```

## Getting started

**Requirements:** Python 3.9 or newer.

```bash
# 1. Get the code
git clone https://github.com/YOUR-USERNAME/skillbridge.git
cd skillbridge

# 2. (Optional) create a virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run
python app.py
```

Open http://127.0.0.1:5000, create an account, paste a resume and click **Find my matches**.

## Configuration

All settings are optional and are read from environment variables, or from a `.env` file placed next to `app.py`.

| Variable | Purpose |
| --- | --- |
| `SECRET_KEY` | Signs the login session cookie. **Set a long random value in production.** |
| `ANTHROPIC_API_KEY` | Turns on AI mode for the chatbot. Without it the chatbot runs in offline mode. |
| `CHAT_MODEL` | Claude model used for chat. Defaults to `claude-haiku-5-5`. |
| `GITHUB_TOKEN` | Raises the GitHub API rate limit for skill verification. |

Example `.env`:

```
SECRET_KEY=change-this-to-a-long-random-string
ANTHROPIC_API_KEY=sk-ant-your-key-here
```

Never commit your `.env` file. It is already listed in `.gitignore`.

## How it works

### Skill matching (rule-based)

1. Each skill in `roles.json` has aliases (for example `react`, `reactjs`). The resume is scanned for whole-word matches, and found skills are saved as **claimed**.
2. Every role lists the skills it needs with a weight (3 = core, 2, 1).
3. The match score is:

   ```
   Match % = 100 × Σ(weight × credit) ÷ Σ(weight)
   ```

   Credit is `1.0` for a verified skill, `0.5` for a claimed skill and `0` for a missing skill.
4. Missing skills are listed with core skills first, each with a suggested project.

### Skill verification

Submitting a public GitHub repo for a skill runs four checks. A score of 60 or more marks the skill as **verified**.

| Check | Points |
| --- | --- |
| Repo content relates to the skill | 40 |
| Has tests | 25 |
| Has at least 5 files | 20 |
| Has a README | 15 |

### AI chatbot

1. The backend finds the company or role in the question, reads your saved skills plus any pasted resume, and computes scores and gaps.
2. **AI mode** (API key set): those facts are sent to Claude with strict instructions, and Claude writes the reply.
3. **Offline mode** (no key): the same facts are turned into a built-in answer.
4. The chatbot never promises selection. It gives a readiness level and explains what else matters (openings, tests, cut-offs, college, referrals).

Safety measures: resume text is passed as data inside tags and instructions in it are ignored, each user is limited to 30 AI chats per hour, the API key stays on the server, and any AI failure falls back to offline mode.

## Data and security

- **Tables:** `users` (account and profile), `skills` (one row per user and skill, with status and evidence), `subs` (GitHub submissions).
- Passwords are stored as salted hashes (Werkzeug), never as plain text.
- Login uses a signed, HttpOnly, SameSite=Lax session cookie.
- All database queries are parameterized, and all text shown in the browser is escaped.
- Uploads are limited to 5 MB.

## API overview

| Endpoint | Method | Purpose |
| --- | --- | --- |
| `/api/register`, `/api/login`, `/api/logout` | POST | Account actions |
| `/api/me` | GET | Current user |
| `/api/resume` | POST | Read a resume and save found skills |
| `/api/skills` | GET, POST | List or add skills |
| `/api/roles` | GET | Search roles (`q`, `sector`, `country`, `limit`) |
| `/api/gap/{id}` | GET | Skill gap for one role |
| `/api/meta` | GET | Sector and country lists with counts |
| `/api/profile` | GET, POST | Read or save the profile |
| `/api/submit` | POST | Verify a skill with a GitHub repo |
| `/api/chat` | POST | Ask the chatbot |
| `/api/chat/status` | GET | Whether AI mode is on |

## Customising the data

Open `roles.json` to add a company, a skill or a role type. A company entry looks like this:

```json
{"n": "Infosys", "c": "India", "s": "IT Services", "r": ["trainee", "fullstack", "data"]}
```

`n` is the name, `c` the country, `s` the sector, `r` the role types it hires (defined under `archetypes`), `i: 1` marks a global company with major India offices, and `al` adds extra names the chatbot recognises.

## Deployment

The app runs on any host that supports Flask. A free option is **PythonAnywhere**:

1. Upload the project and run `pip install --user -r requirements.txt` in a Bash console.
2. Create a web app with **Manual configuration** and point its WSGI file at the project:

   ```python
   import sys
   path = '/home/YOURUSERNAME/skillbridge'
   if path not in sys.path:
       sys.path.insert(0, path)
   from app import app as application
   ```

3. Put `SECRET_KEY` (and optionally `ANTHROPIC_API_KEY`) in a `.env` file inside the project folder.
4. Click **Reload** on the Web tab.

Note: SQLite needs a host with a persistent disk. Serverless platforms such as Netlify or Vercel will not keep the database.

## Limitations

- Skill extraction is keyword-based, so it can miss synonyms or count a mention without real ability.
- Only 27 skills are recognised, and PDFs must contain selectable text.
- Company data is a curated sample, not live job postings.
- SQLite suits a prototype, not heavy traffic.
- No email verification or password reset yet.

## Roadmap

- Semantic skill matching with embeddings
- Live job data from company career pages or APIs
- Mock interview practice and learning roadmaps
- Google or LinkedIn sign-in
- PostgreSQL for scale
- A dashboard for college placement cells

## Team

Add your team members here.

| Name | Role |
| --- | --- |
| Your name | Developer |

## License

Add a license of your choice (for example MIT) before publishing.
