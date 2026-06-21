# Information Disclosure in Version Control History

**Lab:** Information disclosure in version control history
**Series:** Information Disclosure — Lab 5
**Level:** Apprentice
**Goal:** Find the exposed `.git` folder, recover the admin password from old commit history, log in as administrator, and delete the user `carlos`.

---

## What this bug is

The developers left the website's `.git` folder open to the public.

`.git` is the hidden folder Git uses to store the **full history** of a project — every change, every old version, every deleted secret. If it's reachable over the web, anyone can download the entire source history.

A password was committed once, then removed in a later commit. But Git never forgets. The old version still lives in the history.

---

## Step 1 — Map the site for hidden folders

I ran a content discovery scan to look for files and folders not linked anywhere on the page.

Run gobuster against the target.

gobuster dir -u https://TARGET-ID.web-security-academy.net -w /usr/share/wordlists/dirb/common.txt


<img width="1424" height="552" alt="image" src="https://github.com/user-attachments/assets/6be8dd94-7ed0-44e4-a914-8b3d883b837e" />

- `gobuster dir` → directory/file brute-force mode.
- `-u` → the target URL (swap in your lab ID).
- `-w` → the wordlist of names to try.

In the results, one line stood out:
.git                 (Status: 200) [Size: 1201]
.git/HEAD            (Status: 200) [Size: 23]
.git/config          (Status: 200) [Size: 157]
.git/index           (Status: 200) [Size: 225]
.git/logs/           (Status: 200) [Size: 548]
ADMIN                (Status: 401) [Size: 2721]
Admin                (Status: 401) [Size: 2721]
Login                (Status: 200) [Size: 3296]
admin                (Status: 401) [Size: 2721]
analytics            (Status: 200) [Size: 0]


`Status: 200` means the page exists and is readable. The `.git` folder is exposed.



---

## Step 2 — Confirm the .git folder is really open

Before dumping anything, I checked one known Git file by hand.

Open the config file in the browser or with curl.

<img width="1100" height="769" alt="image" src="https://github.com/user-attachments/assets/35a3ceef-3a93-451c-ad21-04c99022607b" />

curl https://0a970069031e92168127269600750008.web-security-academy.net/.git

<img width="902" height="343" alt="image" src="https://github.com/user-attachments/assets/a2d51f6d-0f14-465e-9fa2-69304bea4e46" />


- `curl` → fetches a URL and prints the response.
- the path → a standard Git file that always exists inside `.git`.

It returned Git config text instead of an error. Confirmed — the whole repo is downloadable.

[INSERT SCREENSHOT: browser showing /.git/config contents]

---

## Step 3 — Download the whole repository

I pulled the entire `.git` folder down to my machine.

Recursively download everything under the target URL.

wget -r https://0a970069031e92168127269600750008.web-security-academy.net/.git

<img width="1445" height="308" alt="image" src="https://github.com/user-attachments/assets/79301556-944c-41bf-a438-7d911cbe13c5" />

- `wget` → command-line downloader.
- `-r` → recursive: follow links and grab every file in the folder tree.
- the path → start the download at the exposed `.git` folder.

This rebuilds a local copy of the project's Git repository.



---

## Step 4 — Move into the downloaded repo

Go into the folder wget created.

- `cd` → change directory (move into the folder named after the target).

<img width="842" height="275" alt="image" src="https://github.com/user-attachments/assets/b0d7ab0d-cbbf-4f55-a77a-e53c3c9200af" />


---

## Step 5 — Read the commit history

I listed every commit to see what changed over time.

Show the history, one line per commit.

  git log --oneline    

<img width="896" height="204" alt="image" src="https://github.com/user-attachments/assets/0d2063c6-d43b-4d81-b1c9-7e9a3feab0f5" />

- `git log` → shows the commit history.
- `--oneline` → short format: one commit per line, with its short hash and message.

One commit message gave it away:
9b052f4 (HEAD -> master) Remove admin password from config
683326d Add skeleton admin panel


A skeleton admin panel is exactly where a hardcoded password would sneak in.



---

## Step 6 — Open the suspicious commit

I read the full contents of that commit.

Show everything that commit changed.

<img width="888" height="351" alt="image" src="https://github.com/user-attachments/assets/c6fe58dd-bdaa-4e3d-ae3c-1df0dd104b2d" />



Green `+` = added here. A later commit removed it (red `-` line), but this old version still holds it. Deleted in the present, alive in the history.

[INSERT SCREENSHOT: git show output with the ADMIN_PASSWORD line highlighted]

---

## Step 7 — Log in as administrator

I went to the login page and signed in.

- Username: `administrator`
- Password: the value recovered from the commit

The admin panel loaded.

<img width="1064" height="741" alt="image" src="https://github.com/user-attachments/assets/20190d51-0d29-4e3f-a31b-4c8528492a18" />


---

## Step 8 — Delete carlos

In the admin panel, I deleted the user `carlos` to solve the lab.

<img width="1066" height="647" alt="image" src="https://github.com/user-attachments/assets/830f10de-8ad4-4b19-b3de-6d3de072b656" />


---

## Why this works

`.git` is meant to stay on the developer's machine and the server's deploy pipeline — never in the public web root. When it's exposed, the attacker gets the **entire source history**, not just the live code. Removing a secret in a new commit does not erase it; Git keeps every old version forever unless the history is rewritten and force-pushed. Deleting the line is not deleting the secret.

---

## How I'd spot this in a real app

1. Always run content discovery for `.git`, `.svn`, `.hg`, and `.bzr` folders.
2. Test `/.git/HEAD` and `/.git/config` directly — a `200` means the repo is exposed.
3. If open, dump it (`wget -r`, or tools like `git-dumper`).
4. Read the **full history**, not just current files — `git log`, then `git show` on anything named like config, admin, secret, deploy.
5. Grep the history for `password`, `secret`, `key`, `token`, `.env`.

---

**One-line summary:** An open `.git` folder hands over the project's whole history — and a password "deleted" in a later commit was still sitting in an older one.
