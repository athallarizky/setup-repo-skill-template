# Demo Dashboard — Troubleshooting Guide

This document shows how to keep detailed failure handling outside the main setup guide. The setup agent should read only the relevant section when the matching error appears.

---

## 1 — SSH Access Failure Before Clone

**Symptoms**

```text
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.
```

**Cause**

The user's SSH key is not added to GitHub, or the account does not have access to `example-org/demo-dashboard`.

**Fix**

1. Verify GitHub SSH authentication:

   ```bash
   ssh -T git@github.com
   ```

2. If authentication fails, add a public key at `https://github.com/settings/keys`.
3. If authentication succeeds but `git ls-remote` fails, ask an organization admin to grant repository access.
4. Retry the clone only after `git ls-remote` succeeds.

---

## 2 — Install Fails With `401 Unauthorized` for `@example-org` Packages

**Symptoms**

```text
ERR_PNPM_FETCH_401 GET https://npm.pkg.github.com/@example-org%2F...: Unauthorized - 401
No authorization header was set for the request.
```

**Cause**

The package registry token is missing, expired, or not visible to the package manager process.

**Diagnostic**

Check whether config files exist without printing token values:

```bash
ls -la ~/.npmrc .npmrc
npm config get @example-org:registry
```

**Fix**

1. Ensure the active `.npmrc` maps the scope:

   ```ini
   @example-org:registry=https://npm.pkg.github.com/
   ```

2. Ensure the active `.npmrc` references an environment variable:

   ```ini
   //npm.pkg.github.com/:_authToken=${NPM_TOKEN}
   ```

3. Ensure `NPM_TOKEN` is set in the user's shell with a token that can read private packages.

Do not print the token. Do not ask the user to paste it into chat.

---

## 3 — Install Fails With `404 Not Found` for `@example-org` Packages

**Symptoms**

```text
ERR_PNPM_FETCH_404 GET https://registry.npmjs.org/@example-org%2F...: Not Found - 404
```

**Cause**

The package manager is looking in the public npm registry instead of GitHub Packages.

**Fix**

Add or verify this scope mapping in the project `.npmrc` or user `~/.npmrc`:

```ini
@example-org:registry=https://npm.pkg.github.com/
```

Then retry the install command.

---

## 4 — Native Build Scripts Are Blocked

**Symptoms**

```text
ERR_PNPM_IGNORED_BUILDS Ignored build scripts: ...
Run "pnpm approve-builds" to pick which dependencies should be allowed to run scripts.
```

**Cause**

The package manager blocked native dependency build scripts until the user approves them.

**Fix**

Ask the user before running an interactive approval command:

```bash
pnpm approve-builds
```

If dependencies are already installed and only the dev script is blocked by a lifecycle hook, run the framework binary directly as a temporary local workaround. For Next.js:

```bash
PORT=<port> ./node_modules/.bin/next dev
```

---

## 5 — Missing Env File

**Symptoms**

```text
Error: Missing required environment variable
ENOENT: no such file or directory, open '.env.local'
```

**Cause**

A local env file was not created from the example file.

**Fix**

Copy the matching example file after asking before overwriting anything:

```bash
cp .env.example .env
cp .env.local.example .env.local
```

If the app has multiple env locations, create each required file from its matching example.

---

## 6 — nginx `502 Bad Gateway`

**Symptoms**

```text
502 Bad Gateway
nginx/x.x.x
```

**Cause**

nginx is running, but the app is not listening on the proxied port.

**Diagnostic**

```bash
lsof -i :<port>
curl -I http://127.0.0.1:<port>
```

**Fix**

Start the app on the configured port:

```bash
PORT=<port> <pm> run <run-script>
```

Wait for the ready message, then reload the domain.

---

## 7 — Manual `/etc/hosts` Entry When sudo Cannot Prompt

**Symptoms**

```text
sudo: a terminal is required to read the password
```

**Fix**

Ask the user to run this in their own terminal:

```bash
sudo sh -c 'echo "127.0.0.1  demo-dashboard.local" >> /etc/hosts'
```

No service restart is required for hosts file changes.

---

## 8 — `mkcert -install` Fails or Browser Shows a Certificate Warning

**Symptoms**

```text
ERROR: failed to execute "security add-trusted-cert": exit status 1
sudo: a terminal is required to read the password
```

**Cause**

`mkcert -install` needs elevated privileges to add a local CA to the system trust store.

**Fix**

Ask the user to run this in their own terminal:

```bash
mkcert -install
```

Then regenerate or reuse the local certificate and reload nginx.

