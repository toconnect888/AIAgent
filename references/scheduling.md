# Scheduling the Lead Scanner — Mac & Windows

Run the Gmail lead scan automatically every 3 days at 8:00am on your personal computer.

The trigger mechanism is a small script that opens Claude.ai and sends a message to
kick off the scan. The simplest approach is a shell/batch script that uses your
browser or curl to trigger a Claude conversation.

---

## Option A: Mac (using launchd)

### Step 1: Create the trigger script

Save this as `~/lead-scan.sh`:

```bash
#!/bin/bash
# Opens Claude.ai with the lead scan prompt
open "https://claude.ai/new?q=Run+the+Gmail+lead+scan+for+the+past+3+days+and+upload+to+HubSpot"
```

Make it executable:
```bash
chmod +x ~/lead-scan.sh
```

### Step 2: Create the launchd plist

Save this file as `~/Library/LaunchAgents/com.leadscan.plist`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.leadscan</string>

  <key>ProgramArguments</key>
  <array>
    <string>/bin/bash</string>
    <string>/Users/YOUR_USERNAME/lead-scan.sh</string>
  </array>

  <key>StartCalendarInterval</key>
  <array>
    <!-- Day 1 -->
    <dict>
      <key>Hour</key><integer>8</integer>
      <key>Minute</key><integer>0</integer>
      <key>Weekday</key><integer>1</integer>
    </dict>
    <!-- Day 4 (every 3 days approximation: Mon + Thu) -->
    <dict>
      <key>Hour</key><integer>8</integer>
      <key>Minute</key><integer>0</integer>
      <key>Weekday</key><integer>4</integer>
    </dict>
  </array>

  <key>RunAtLoad</key>
  <false/>
</dict>
</plist>
```

> Note: Replace `YOUR_USERNAME` with your Mac username (run `whoami` in Terminal).
> launchd doesn't support "every N days" natively, so Monday + Thursday (every ~3 days)
> is the closest approximation. Adjust weekdays as needed.

### Step 3: Load the schedule
```bash
launchctl load ~/Library/LaunchAgents/com.leadscan.plist
```

To unload/stop:
```bash
launchctl unload ~/Library/LaunchAgents/com.leadscan.plist
```

---

## Option B: Windows (Task Scheduler)

### Step 1: Create the trigger script

Save this as `C:\Users\YOUR_USERNAME\lead-scan.bat`:

```batch
@echo off
start "" "https://claude.ai/new?q=Run+the+Gmail+lead+scan+for+the+past+3+days+and+upload+to+HubSpot"
```

### Step 2: Set up Task Scheduler

1. Open **Task Scheduler** (search in Start menu)
2. Click **Create Basic Task**
3. Name it: `Gmail Lead Scanner`
4. Trigger: **Weekly** → Select Monday and Thursday at 8:00 AM
5. Action: **Start a program**
   - Program: `C:\Users\YOUR_USERNAME\lead-scan.bat`
6. Click **Finish**

To run every 3 days more precisely:
- In Task Scheduler, after creating the task, open its properties
- Under **Triggers**, edit the trigger
- Check "Repeat task every" and set a custom interval if desired

---

## Important Notes

- Your computer must be **on and not sleeping** at 8am for the task to run
- If asleep, Mac launchd will run the task when the computer wakes up (same day)
- Windows Task Scheduler can be configured to wake the computer from sleep:
  check "Wake the computer to run this task" in the Conditions tab
- You still need to manually confirm the scan in Claude.ai when the browser opens
  (Claude requires your active session — fully automated headless runs require
  a server setup instead)