# Kandies Project: Startup Commands

Reference for starting each system in the Kandies project. Last updated 2026-10-22.

Each command says where it runs. Run one box at a time. No passwords, keys or emails are stored here.

---

## 1. Windows PC: run Kandies locally (test before going live)

**Run in: PowerShell (window 1, the "server" window)**

```powershell
cd "$HOME\kandies-app\www"
```

```powershell
python -m http.server 8080
```

Then open this in Chrome (not Firefox, which forces https and breaks it):

```text
http://localhost:8080
```

- Press **Ctrl+Shift+R** in Chrome after any file change.
- Don't click inside this window while it runs (it pauses the server). If the title says "Select", press **Esc**.
- To stop the server, press **Ctrl+C** in this window.

---

## 2. Windows PC: open the code editor

**Run in: PowerShell (window 2, the "commands" window)**

```powershell
codium "$HOME\kandies-app"
```

If that isn't recognized, open **VSCodium** from the Start menu, then **File > Open Folder**, then choose `C:\Users\<you>\kandies-app`.

---

## 3. Windows PC: get the latest code before working

**Run in: PowerShell (window 2)**

```powershell
cd "$HOME\kandies-app"
```

```powershell
git pull
```

---

## 4. Windows PC: save and send changes to GitHub

**Run in: PowerShell (window 2)**

```powershell
cd "$HOME\kandies-app"
```

```powershell
git status
```

```powershell
git add -A
```

```powershell
git commit -m "Describe what you changed"
```

```powershell
git push
```

A push also starts a new Android test build automatically.

---

## 5. Windows PC: bring in a new index.html from Claude

**Run in: PowerShell (window 2)**

Clear the old download first:

```powershell
Remove-Item "$HOME\Downloads\index.html"
```

Download the new file (click the file card, then the arrow next to Copy, then Download). Then check its size matches what Claude says:

```powershell
Get-ChildItem "$HOME\Downloads" -Filter "index*"
```

Copy it into the project:

```powershell
Copy-Item "$HOME\Downloads\index.html" "$HOME\kandies-app\www\index.html"
```

If a file ever gets messed up, restore the last saved version from GitHub:

```powershell
git -C "$HOME\kandies-app" restore www/index.html
```

---

## 6. Windows PC: GitHub sign-in check

**Run in: PowerShell**

```powershell
gh auth status
```

If it says you're not logged in:

```powershell
gh auth login
```

Choose: GitHub.com, then HTTPS, then Yes, then Login with a web browser.

---

## 7. Android test app (APK)

**Run in: PowerShell.** See recent builds:

```powershell
gh run list --repo duppy-c/kandies-app
```

Download the newest test APK to your current folder:

```powershell
gh run download --repo duppy-c/kandies-app -n kandies-test-apk
```

Or on GitHub: open the repo, then **Actions**, then the newest green run, then **kandies-test-apk**.

---

## 8. Put the website live (Netlify)

1. Go to **app.netlify.com** and open the **kandies** project.
2. In the **"Already built it?"** box, click **folder**.
3. In the **Folder:** box, paste this, then click **Upload**:

```text
C:\Users\<you>\kandies-app\www
```

Always upload the whole **www** folder, never a single file.

---

## 9. Supabase (database)

Web only: **supabase.com**, then your Kandies project, then **SQL Editor**.

Check the scheduled jobs are all running:

**Run in: Supabase SQL Editor**

```sql
select jobname, schedule, active from cron.job order by jobid;
```

You should see 5 jobs, all active.

---

## 10. Raspberry Pi (Raspberry Pi OS, the Kandies builder)

**Run in: Pi Terminal**

```bash
cd ~/kandies-app
```

```bash
git pull
```

```bash
npx cap sync android
```

Shut down safely before unplugging (wait for only the solid red light):

```bash
sudo shutdown -h now
```

---

## 11. Raspberry Pi (Parrot OS, the cybersecurity lab)

**Run in: Parrot Terminal.** Update everything:

```bash
sudo parrot-upgrade
```

Check the firewall is on:

```bash
sudo ufw status verbose
```

Fix the screen if the top bar gets cut off:

```bash
xrandr --output HDMI-2 --mode 1360x768
```

Restart or shut down:

```bash
sudo reboot
```

```bash
sudo shutdown -h now
```

---

## 12. Parrot OS virtual machine on Windows (shelved, slow on this PC)

**Run in: PowerShell**

```powershell
& "C:\Program Files\Oracle\VirtualBox\VBoxManage.exe" startvm "Parrot"
```

To free memory first:

```powershell
wsl --shutdown
```

---

## Quick start: a normal work session

1. PowerShell window 2: `cd "$HOME\kandies-app"`, then `git pull`
2. PowerShell window 1: `cd "$HOME\kandies-app\www"`, then `python -m http.server 8080`
3. Chrome: `http://localhost:8080`
4. Edit, test with Ctrl+Shift+R
5. Window 2: `git add -A`, then `git commit -m "..."`, then `git push`
6. Netlify: upload the `www` folder
