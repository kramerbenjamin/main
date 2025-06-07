
# 🧠 Auto-Sync Git Workspace on macOS with VS Code

This guide shows you how to automatically commit and push file changes from a local folder (`~/homelab`) to GitHub using a LaunchAgent, and how to integrate this with VS Code.

---

## 📁 1. Create Your Workspace and Git Repository

```bash
mkdir ~/homelab
cd ~/homelab
git init
echo "# Homelab" > README.md
git add .
git commit -m "Initial commit"
```

Then push it to GitHub:

```bash
git remote add origin git@github.com:YOUR_USERNAME/homelab.git
git push -u origin main
```

---

## 🔧 2. Create `.gitignore` File

Prevent unnecessary files from being tracked:

```gitignore
# macOS
.DS_Store

# Log files
*.log

# VS Code settings
.vscode/

# Temp files
*.tmp
*.swp
```

---

## 🛠 3. Create the Auto-Sync Script

Save this as `~/homelab/git-auto-sync.sh`:

```bash
#!/bin/bash
WATCH_DIR="$HOME/homelab"
cd "$WATCH_DIR"
exec >> "$WATCH_DIR/git-auto-sync.log" 2>&1
/opt/homebrew/bin/fswatch -o "$WATCH_DIR" | while read; do
  git add .
  git commit -m "Auto-sync: $(date '+%Y-%m-%d %H:%M:%S')"
  git push
done
```

Make it executable:

```bash
chmod +x ~/homelab/git-auto-sync.sh
```

> Make sure to replace `/opt/homebrew/bin/fswatch` with the result of `which fswatch`

---

## 🚀 4. Create a LaunchAgent

Create `~/Library/LaunchAgents/com.user.homelab.sync.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" 
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>com.user.homelab.sync</string>
    <key>ProgramArguments</key>
    <array>
      <string>/bin/bash</string>
      <string>/Users/YOUR_USERNAME/homelab/git-auto-sync.sh</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
    <key>StandardOutPath</key>
    <string>/Users/YOUR_USERNAME/homelab/git-auto-sync.out.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/YOUR_USERNAME/homelab/git-auto-sync.err.log</string>
  </dict>
</plist>
```

Replace `YOUR_USERNAME` with your macOS username.

Then load it:

```bash
launchctl load ~/Library/LaunchAgents/com.user.homelab.sync.plist
```

---

## 💻 5. Use With VS Code

Open the workspace:

```bash
code ~/homelab
```

Use the Git sidebar to:
- Stage changes
- View diffs
- Manually push if needed

VS Code will automatically respect `.gitignore`.

---

## 🧹 6. Clean Existing Tracked Files

If `.log` or `.DS_Store` files were already added, remove them:

```bash
git rm --cached *.log
git rm --cached .DS_Store
git commit -m "Remove log and DS_Store files"
git push
```

---

## ✅ Done!

Your workspace is now:
- Synced automatically with GitHub on any change
- Cleaned of unnecessary files
- Integrated with VS Code

