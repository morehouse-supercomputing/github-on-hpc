---
layout: default
title: GitHub Authentication on HPC
---

# Authenticating to GitHub on HPC

When you try to `git push` from a TACC compute node, you will hit an error like this:

```
remote: Invalid username or token. Password authentication is not supported for Git operations.
fatal: Authentication failed for 'https://github.com/.../your-repo.git'
```

This is normal. Two things are going on:

1. **GitHub no longer accepts passwords** for Git. You have to use a **Personal Access Token (PAT)** or an **SSH key** instead.
2. **Compute nodes have no graphical screen**, so the system's pop-up password box fails with a `cannot open display` / `gnome-ssh-askpass` warning. You need git to ask for credentials in the terminal instead.

This guide fixes both. **Method 1 (token)** is the recommended path on TACC and works everywhere. **Method 2 (SSH key)** never expires but can be blocked by the TACC firewall.

> Run every command below **on the HPC system** (your terminal prompt will look like `c611-152[gh]$` on a compute node or `login1.vista$` on a login node). Your `$HOME` is shared across nodes, so this setup follows you everywhere on the system.

---

## One-time setup (do this first)

### Set your git identity

This clears the "Committer ... configured automatically" warning and puts your real name on commits:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@morehouse.edu"
```

### Stop the pop-up password box

So git prompts you in the terminal instead of trying to open a window that does not exist:

```bash
unset SSH_ASKPASS
```

---

# Method 1: Personal Access Token (Recommended)

A Personal Access Token (PAT) is a long password-like string from GitHub that you use in place of your password.

## Step 1: Create the token

Do this part in a **web browser on your laptop**:

1. Go to GitHub and sign in.
2. Click your profile picture (top right) → **Settings**.
3. In the left menu, scroll down to **Developer settings**.
4. Click **Personal access tokens → Tokens (classic)**.
5. Click **Generate new token → Generate new token (classic)**.
6. Give it a name (e.g. `tacc-vista`), set **Expiration** to **No expiration** (or a long window so you are not redoing this constantly).
7. Check the **`repo`** box (this grants push/pull access).
8. Click **Generate token**.
9. **Copy the token now.** It starts with `ghp_...`. GitHub shows it only once. If you lose it, you make a new one.

## Step 2: Push using the token

Back on the HPC, push as usual:

```bash
git push origin main
```

When prompted:

- **Username:** your GitHub username
- **Password:** paste the **token** (not your GitHub password)

> Pasting into a terminal usually does nothing visible. That is expected. Paste and press Enter.

That is it. Your commit goes up.

## Step 3: Save it so you never retype it

By default git asks every push. To store the token once and reuse it forever:

```bash
git config --global credential.helper store
git push origin main
```

Enter your username and token **one more time**. After that, git saves it to `~/.git-credentials` and never asks again, from any node.

Lock the file down so others on the system cannot read it:

```bash
chmod 600 ~/.git-credentials
```

> **Heads up:** the token is saved in plain text in that file. This is fine on your own personal account. **Never** do this on a shared, demo, or training login that other people use.

---

# Method 2: SSH Key (No Token Needed)

An SSH key is a pair of files: a private key that stays on the HPC and a public key you give to GitHub. Once set up, pushing never asks for anything and never expires.

> **Important caveat:** TACC sometimes blocks outbound SSH on port 22, which is what GitHub uses by default. If Method 2 fails with a timeout or "connection refused," use the port-443 workaround at the bottom, or just use Method 1.

## Step 1: Create the key

On the HPC:

```bash
ssh-keygen -t ed25519 -C "you@morehouse.edu"
```

Press **Enter** through all the prompts (accept the default location, leave the passphrase blank for unattended pushing).

## Step 2: Copy your public key

```bash
cat ~/.ssh/id_ed25519.pub
```

Copy the entire line it prints (starts with `ssh-ed25519 ...`).

## Step 3: Add it to GitHub

In a browser on your laptop:

1. GitHub → **Settings → SSH and GPG keys**.
2. Click **New SSH key**.
3. Title it (e.g. `tacc-vista`), paste the key, click **Add SSH key**.

## Step 4: Point your repo at the SSH address

```bash
git remote set-url origin git@github.com:ORG/REPO.git
git push origin main
```

Replace `ORG/REPO` with your actual org and repo name. The first push asks you to confirm GitHub's fingerprint; type `yes`.

### If port 22 is blocked (TACC firewall)

Tell SSH to reach GitHub over port 443 instead. Create or edit `~/.ssh/config`:

```bash
cat >> ~/.ssh/config <<'EOF'
Host github.com
  Hostname ssh.github.com
  Port 443
  User git
EOF
```

Then `git push origin main` again. If this still fails, fall back to Method 1, which uses HTTPS and is rarely blocked.

---

# Quick Reference

| Task | Command |
|------|---------|
| Set identity | `git config --global user.name "Your Name"` |
| Set email | `git config --global user.email "you@morehouse.edu"` |
| Stop pop-up prompt | `unset SSH_ASKPASS` |
| Save token permanently | `git config --global credential.helper store` |
| Secure the saved token | `chmod 600 ~/.git-credentials` |
| Make an SSH key | `ssh-keygen -t ed25519 -C "you@morehouse.edu"` |
| Show public key | `cat ~/.ssh/id_ed25519.pub` |
| Switch repo to SSH | `git remote set-url origin git@github.com:ORG/REPO.git` |

---

# Troubleshooting

**"Password authentication is not supported"**
You typed your GitHub password. Use a Personal Access Token instead (Method 1).

**"cannot open display" / "gnome-ssh-askpass" warning**
The node is trying to open a graphical password box. Run `unset SSH_ASKPASS` and push again so it prompts in the terminal.

**"Committer ... configured automatically based on your username and hostname"**
Harmless, but set your identity with the two `git config --global` commands above to make commits show your real name.

**It still asks for my token every time**
You used `credential.helper cache` (temporary) or skipped it. Run `git config --global credential.helper store`, push once more, and it will stop asking.

**SSH push hangs or says "connection refused" / "timed out"**
Port 22 is likely blocked. Use the port-443 workaround in Method 2, or switch to Method 1.

**"remote: Repository not found"**
Either the repo URL is wrong, or your token/key does not have access to that repo. Confirm the org/repo name and that your token has the `repo` scope.

---

# Getting Help

- **GitHub token docs:** [docs.github.com/.../managing-your-personal-access-tokens](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens)
- **MSCF Questions:** [ashley.scruse@morehouse.edu](mailto:ashley.scruse@morehouse.edu)
