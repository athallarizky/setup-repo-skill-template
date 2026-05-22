# Demo Dashboard — Local Setup Guide

**Repo:** `git@github.com:example-org/demo-dashboard.git`  
**Default clone name:** `demo-dashboard`  
**Default domain:** `demo-dashboard.local`  
**Default port:** `3000`  
**Supported environments:** macOS, Linux

> For known setup failures such as SSH errors, private package install errors, missing env files, blocked native builds, and nginx proxy failures, see [`docs/troubleshooting.md`](docs/troubleshooting.md).

---

## Step 1 — Ask Clone Folder

Use a user-select question:

- `demo-dashboard` (default)
- custom folder name

Store the chosen value as `<clone-folder>`.

If user-select is unavailable, ask the same choice in chat and continue after the user answers.

---

## Step 2 — Verify SSH Access

Before cloning, verify the user has SSH access to the repository:

```bash
git ls-remote git@github.com:example-org/demo-dashboard.git
```

- If the command succeeds, continue to Step 3.
- If the command fails with permission denied or timeout, stop and tell the user:
  > SSH access to `git@github.com:example-org/demo-dashboard.git` failed. Please ensure your SSH key is added to GitHub and your account has access to `example-org`.
- Do not attempt the clone until SSH access is confirmed.

---

## Step 3 — Clone or Use Existing Folder

Check whether `<clone-folder>` already exists in the current directory.

If the folder does not exist:

```bash
git clone git@github.com:example-org/demo-dashboard.git <clone-folder>
```

If the folder already exists, use a user-select question:

- use the existing `<clone-folder>`
- choose a different folder name

If the user chooses a different folder name, return to Step 1.

Enter the folder before proceeding:

```bash
cd <clone-folder>
```

---

## Step 4 — Select Package Manager

Inspect the repository to infer the package manager:

| Lockfile | Package manager |
|---|---|
| `pnpm-lock.yaml` | `pnpm` |
| `yarn.lock` | `yarn` |
| `package-lock.json` | `npm` |
| `bun.lockb` or `bun.lock` | `bun` |

If multiple lockfiles exist, use a user-select question with the detected options and recommend the newest modified lockfile.

Store the chosen package manager as `<pm>`.

---

## Step 5 — Node Version Setup

Prefer `nvm` when a Node version file is present.

Check in this order:

1. `.nvmrc`
2. `.node-version`
3. `package.json` `engines.node`

If a version is found and `nvm` is available:

```bash
nvm use
```

If `nvm` is not available, tell the user which Node version is required and ask them to activate it with their preferred version manager.

If no Node version is declared, continue with the active Node version and report it:

```bash
node --version
```

---

## Step 6 — Ensure Package Manager Is Available

Check the selected package manager:

```bash
<pm> --version
```

If `<pm>` is missing and the project uses `pnpm` or `yarn`, ask before enabling it via `corepack`:

```bash
corepack enable
corepack prepare <pm>@latest --activate
```

If `corepack` is unavailable, ask before using another installer such as `npm install -g <pm>` or a system package manager.

---

## Step 7 — Install Dependencies

Run the install command for the selected package manager:

| Package manager | Install command |
|---|---|
| `pnpm` | `pnpm install` |
| `yarn` | `yarn install` |
| `npm` | `npm install` |
| `bun` | `bun install` |

If install fails with a `401` or `404` for `@example-org` packages, read sections 2 and 3 in [`docs/troubleshooting.md`](docs/troubleshooting.md).

Important:

- Do not print token values.
- Do not ask the user to paste tokens into chat.
- Prefer environment variable references such as `${NPM_TOKEN}` in config examples.

---

## Step 8 — Verify Env Files

Inspect the repository for env examples:

```bash
find . -maxdepth 3 -name ".env.example" -o -name "*.env.example"
```

Common defaults:

| If this example exists | Create this local file when missing |
|---|---|
| `.env.example` | `.env` |
| `.env.local.example` | `.env.local` |
| `config/.env.example` | `config/.env` |

Ask before overwriting existing env files. Never print secret values from env files.

If no env example exists, ask the user where the env template is located or whether the app can run without one.

---

## Step 9 — Select Run Script

Read `package.json` and list available scripts.

Recommend a local development script using this priority:

1. scripts whose name contains `local`
2. scripts whose name contains `dev`
3. scripts whose name contains `start`

Use a user-select question to ask which script to run. Store the choice as `<run-script>`.

---

## Step 10 — Ask Install Mode

Use a user-select question:

- `localhost` - run directly on a local port
- `.local domain proxy` - run behind an nginx proxy at a local domain

Store the choice as `<install-mode>`.

---

## Step 11 — Ask Port

Use a user-select question:

- `3000` (default)
- custom port

Store the chosen value as `<port>`.

---

## Step 12 — Run the App

### Localhost Mode

Start the app on the chosen port:

```bash
PORT=<port> <pm> run <run-script>
```

If the framework ignores `PORT`, adapt to the framework's convention and verify the app is listening on `<port>`.

Verify with:

```bash
curl -I http://localhost:<port>
```

Report the local URL and response status.

---

### `.local` Domain Proxy Mode

#### 12a — Ask Domain

Use a user-select question:

- `demo-dashboard.local` (default)
- custom domain

Store the chosen value as `<domain>`.

#### 12b — Ask HTTPS Preference

Use a user-select question:

- HTTP only
- HTTPS with `mkcert` certificate

Store the choice as `<use-https>`.

#### 12c — Check Required Tools

Check for `nginx`:

```bash
nginx -v
```

If `nginx` is missing, ask before installing it.

If `<use-https>` is HTTPS, check for `mkcert`:

```bash
mkcert --version
```

If `mkcert` is missing, ask before installing it.

#### 12d — Add `/etc/hosts` Entry

Check whether the domain is already in `/etc/hosts`:

```bash
grep "<domain>" /etc/hosts
```

If not present, ask before adding:

```bash
echo "127.0.0.1  <domain>" | sudo tee -a /etc/hosts
```

#### 12e — Determine nginx Config Path

On macOS, prefer Homebrew nginx config roots in this order:

1. `/opt/homebrew/etc/nginx`
2. `/usr/local/etc/nginx`

Place the site file at:

```text
<nginx-root>/servers/demo-dashboard.conf
```

On Linux, place the site file at:

```text
/etc/nginx/conf.d/demo-dashboard.conf
```

Ask before editing system-level nginx files.

#### 12f — Generate nginx Config

HTTP only:

```nginx
server {
    listen 80;
    server_name <domain>;

    location / {
        proxy_pass http://127.0.0.1:<port>;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

HTTPS:

```bash
mkdir -p <nginx-root>/certs
mkcert -install
mkcert -cert-file <nginx-root>/certs/<domain>.pem -key-file <nginx-root>/certs/<domain>-key.pem <domain>
```

```nginx
server {
    listen 80;
    server_name <domain>;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name <domain>;

    ssl_certificate     <nginx-root>/certs/<domain>.pem;
    ssl_certificate_key <nginx-root>/certs/<domain>-key.pem;

    location / {
        proxy_pass http://127.0.0.1:<port>;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

#### 12g — Validate and Reload nginx

Validate before reload:

```bash
nginx -t
```

If validation passes, reload nginx:

- macOS: `brew services restart nginx` or `sudo nginx -s reload`
- Linux: `sudo systemctl reload nginx` or `sudo nginx -s reload`

If validation fails, show the error and do not reload.

#### 12h — Start and Verify

Start the app:

```bash
PORT=<port> <pm> run <run-script>
```

Verify the domain:

```bash
curl -I http://<domain>
```

For HTTPS:

```bash
curl -I https://<domain>
```

Report the result and the final URL.

---

## Notes for the Agent

- Always ask before modifying system files such as `/etc/hosts`, nginx config, and `nginx.conf`.
- Always ask before installing missing tools.
- Always ask before overwriting existing env files.
- Never print package registry tokens, API keys, or env secret values.
- Never ask the user to paste secret values into chat.
- If sudo cannot prompt for a password in the agent shell, give the exact command for the user to run manually.
- If any step fails unexpectedly, show the relevant error output before suggesting a fix.
- Prefer `rg`, `find`, or structured parsers over noisy shell output when inspecting a project.

