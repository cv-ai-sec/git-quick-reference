# Pre-push secret checklist

Run this before every push to every repo, not only the first push of a new project. A secret that reaches a remote,
public or private, has to be treated as compromised and rotated. Catching it before it leaves the machine costs
nothing.

## The checklist

1. **Review `git status` after any broad `git add`.** Never assume `git add -A` or `git add .` staged only what you
   expected. Read the output, and use `git diff --cached --stat` for a summary.

2. **Confirm a real `.env` was never tracked:**
   ```bash
   git ls-files | grep -E "(^|/)\.env"
   ```
   This should print nothing, or only `.env.example`. If a real `.env` appears, the ignore file was bypassed at some
   point. Unstage it with `git restore --staged .env`. If it was already committed, remove it from tracking with
   `git rm --cached .env` before continuing.

3. **Confirm the ignore file is respected**, rather than assuming it is:
   ```bash
   git check-ignore -v .env
   ```
   This should print the rule that matched. No output means the ignore file does not match that path. Investigate
   before committing.

4. **Search the actual diff, not only the files you remember changing.** Check two scopes: the staged diff (the next
   commit) and the diff against the remote (everything a push would send):
   ```bash
   git diff --cached | grep -iE "api[_-]?key|apikey|secret|password|token|sk-[a-zA-Z0-9]|AIza|BEGIN (RSA|OPENSSH|EC) PRIVATE KEY"
   git diff origin/main..HEAD | grep -iE "api[_-]?key|apikey|secret|password|token|sk-[a-zA-Z0-9]|AIza|BEGIN (RSA|OPENSSH|EC) PRIVATE KEY"
   ```
   PowerShell equivalent:
   ```powershell
   git diff --cached | Select-String -Pattern "api[_-]?key|apikey|secret|password|token|sk-[a-zA-Z0-9]|AIza|BEGIN (RSA|OPENSSH|EC) PRIVATE KEY"
   git diff origin/main..HEAD | Select-String -Pattern "api[_-]?key|apikey|secret|password|token|sk-[a-zA-Z0-9]|AIza|BEGIN (RSA|OPENSSH|EC) PRIVATE KEY"
   ```

5. **Every hit needs a human judgment, not just a pattern match.** Expected, safe hits look like:
   - placeholder values such as `your_token_here` or `change_me`
   - variable or field names in code or docs, such as `API_KEY=`
   - documented fixture values that a local service ignores

   A hit that looks like a real key, password, or live token means stop. Don't push, and don't quietly delete the
   line and continue.

6. **No hard-coded local paths, usernames, or internal addresses.** Search for your operating system username,
   absolute local paths, and real internal network addresses in config, comments, and example logs.

7. **Review the full diff and commit list once more before the push:**
   ```bash
   git diff origin/main..HEAD --stat
   git log --oneline origin/main..HEAD
   ```

## If step 4 finds something real

- **Only local, never pushed:** amend or drop the commit, remove the secret from the file, and recommit.
- **Already on a remote, even a private one:** treat it as compromised. Rotate the credential first, by revoking and
  regenerating it at its source. Then clean up the history. Deleting the line in a new commit is not enough, because
  the old commit still holds it until the history is rewritten or the repository is deleted.

## Why repeat it every time

A clean build says nothing about whether a secret slipped into an example log, a debug print, or a config file edited
in a hurry. The check is cheap, and skipping it once is the expensive version.
