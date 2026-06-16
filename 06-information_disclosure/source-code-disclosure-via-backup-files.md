# The File They Forgot to Delete

*Lab 3 of my Information Disclosure series. No payload. No trick. Just a backup file left lying around — with the database password inside it.*

[INSERT SCREENSHOT: lab home page]

Welcome back to the series.

Last lab, a forgotten note in the code pointed us to a debug page, and the server's secret key fell out. This lab is even lazier — for the attacker, anyway.

No comment to read this time. No page to find. Just a file the developer made, used, and forgot.

I asked the server for it. It said yes.

Let me show you.

## A Quick Recap of Where We Are

New here? Glad you found me. We're working through information disclosure — getting an app to leak things it never meant to share.

Lab 1, the leak was loud. We broke the input and a crash report fell out. It told us the software version.

Lab 2, the leak was quiet. A comment pointed to a debug page, and the page dumped the secret key.

This time the leak is just... sitting there. A whole file. Source code and password, ready to download.

The damage keeps climbing.

## What the Bug Actually Is

Developers make backup copies before they change risky code. They stick `.bak` or `.old` on the end of the filename (backup file = a saved copy of a file, kept "just in case").

Then they forget to delete it.

The problem: the live server still hands out that backup. Anyone can download it.

And backups of source code often hide a secret. A password typed straight into the code (hardcoded = the password is written into the code itself, instead of stored somewhere safe).

That forgotten file is the bug.

## What I Noticed

I opened the lab. A normal shop page. Nothing special.

So I went to my second-favorite first move: I checked `/robots.txt`.

`robots.txt` is a public file that tells search engines which folders to skip (robots.txt = a "please ignore these folders" list — which is really a map of the folders the owner wants hidden).

[INSERT SCREENSHOT: /robots.txt showing "Disallow: /backup"]

There it was.

```
Disallow: /backup
```

The site just told me where the loot is.

## What Fell Out

I walked into that folder.

```
https://YOUR-LAB-ID.web-security-academy.net/backup
```

The folder showed me everything inside it (directory listing = the folder displays its full file list instead of hiding it). One file stood out:

```
ProductTemplate.java.bak
```

`.java` is source code. `.bak` means it's a backup. Two words that mean payday.

So I opened it.

```
https://YOUR-LAB-ID.web-security-academy.net/backup/ProductTemplate.java.bak
```

The raw source loaded in my browser.

[INSERT SCREENSHOT: the .java.bak source code, password line visible]

And right there, in the database connection code:

```java
ConnectionBuilder connectionBuilder = ConnectionBuilder.from(
        "org.postgresql.Driver",
        "postgresql://localhost:5432/postgres",
        "postgres",
        "xXxXxXxXxXxXxXxX"   // the database password, in plain text
).withAutoCommit();
```

The fourth line is the password. Typed straight into the code.

I copied it. Hit **Submit solution**. Pasted it in.

Lab solved.

[INSERT SCREENSHOT: "Congratulations, you solved the lab"]

## Why This Works

Two mistakes stack up.

One: the server serves files it shouldn't. The `/backup` folder is reachable, directory listing is on, and the server treats `.bak` as plain text — so it just hands it over.

Two: the password lives in the code. Easy to do when you're rushing. Backups make it worse — even after you "fix" the live code, the old copy with the password still sits on disk.

Why do devs cause this? Because backups feel temporary. You make one, ship the change, move on. The cleanup never happens. The file outlives the memory of it.

## Real Bug Bounty Payouts

This bug pays. Source code disclosure shows up everywhere:

- **$10,000 — private program.** A researcher found an exposed `.git` folder on four servers, rebuilt the source, and chained it into RCE (RCE = running your own commands on the server — total control). The leaked code showed him the way in.
- **U.S. Department of Defense — HackerOne.** An exposed `.git` folder. The researcher used a tool called `gitdumper` to rebuild the whole repo. DoD pays in recognition, not cash — but even the Pentagon left this open.
- **Udemy — HackerOne.** An exposed dashboard gave a researcher full read access to Udemy's entire source code. Critical.
- **Mozilla — public report.** A subdomain served a live `.git` endpoint. Anyone could pull down the source.

Same bug, wildly different payouts. The secret you find decides the price. Versions = small. Source code with a DB password = big.

## Where to Hunt This in Real Apps

Quick checklist for live targets:

- Read `/robots.txt` and `/sitemap.xml` first. Free folder map.
- For every file you find, try backup endings: `.bak`, `.old`, `.txt`, `.zip`, `~`, `.swp`, `.save`, `.orig`.
- Check for version control leaks: `/.git/HEAD`, `/.git/config`, `/.svn/`.
- Check config leaks: `/.env`, `/config.php.bak`, `/web.config.bak`.
- Fuzz directories (ffuf, feroxbuster). Hunt for `/backup`, `/old`, `/dev`, `/tmp`.
- Found source? Search it for `password`, `secret`, `api_key`, `token`.
- Found a directory listing? Browse it. People dump everything in there.

## The Fix

If you're defending:

- Never serve backup files from production. Block `.bak`, `.old`, `.zip`, `~` at the server.
- Turn off directory listing. A folder should never show its contents.
- Never hardcode secrets. Use environment variables or a vault.
- Clean up backups. Keep them out of the public web folder.
- If a password ever touched a public file, rotate it. Assume it's burned.

---

**One-line summary:** A forgotten `.bak` of the source code was still served by the server, leaking the hardcoded database password in plain text.
