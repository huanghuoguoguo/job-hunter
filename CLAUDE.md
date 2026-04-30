# Claude Code Instructions

This repository is a Claude Code skill package for BOSS job hunting with OpenCLI.

When the user asks to set up or use this repository:

1. Read `README.md` and `SKILL.md`.
2. Check whether `opencli` is available.
3. Run `opencli doctor` before using browser-backed commands.
4. Ask the user to open Chrome and log in to `https://www.zhipin.com` if needed.
5. If `用户偏好.md` does not exist, copy `用户偏好.template.md` to `用户偏好.md` and help the user fill it in.
6. Use the workflow in `SKILL.md` for searching, filtering, duplicate checks, and assisted delivery.
7. Treat `用户偏好.md` as the single place for durable user preferences: location, salary, job direction, industry, company preferences, education, experience, work mode, and delivery rules. Do not edit `SKILL.md` for user-specific preferences.
8. Never commit private files such as `用户偏好.md`, resumes, delivery records, cookies, screenshots, or browser session data.
