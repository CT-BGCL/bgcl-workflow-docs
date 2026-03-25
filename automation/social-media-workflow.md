# Boys & Girls Clubs of Lanier
## Social Media Automation Workflow
### Handoff & Reference Document

**Last Updated:** March 24, 2026  
**Maintained by:** Chris Tyson

> PURPOSE OF THIS DOCUMENT: This is the master reference for BGCL's social media automation workflow. It exists so that nothing slips through the cracks, any team member can pick up where another left off, and a new Claude session can get up to speed instantly without reconstructing context from scratch. Update this document whenever the workflow changes.

---

## 1. Overview

This document describes the end-to-end workflow for capturing, organizing, captioning, and publishing social media content for Boys & Girls Clubs of Lanier. The workflow is designed to be as automated as possible while maintaining quality control and brand consistency.

The single manual step in the workflow is downloading photos from Slack. Everything else --- sorting, file conversion, caption writing, photo filtering, cluster detection, and posting --- runs automatically once the system is fully built.

### Current Build Status

| Component | Status | Notes |
|---|---|---|
| Sorting Script (Sort-BGCLPhotos.ps1) | Complete | v2.2 as of March 24, 2026 |
| HEIC to JPG Conversion | Complete | Requires ImageMagick installed |
| File System Watcher / Task Scheduler | Complete | Watch-BGCLPhotos.ps1 + Register-BGCLWatcher.ps1 running |
| Claude API Caption Generation | Complete | Runs automatically after sort; backfill script also available |
| Photo Quality Filtering | Not yet built | Next step |
| Cluster Detection | Not yet built | Flags same-day batches for review |
| Meta Graph API Posting | Not yet built | Facebook + Instagram |
| Constant Contact API Posting | Not yet built | LinkedIn |
| Priority Queue | Not yet built | Time-sensitive and grant-required content |
| Workflow Document (this file) | Complete | Update as workflow evolves |

---

## 2. Tools & Credentials

The following tools are connected or in use. Credentials must be stored as Windows environment variables and never shared in email, chat, or screenshots.

| Platform | Credential | Where to Find It |
|---|---|---|
| Slack | Workspace: onelanierbgcl.slack.com | Chris Tyson is authorized user (U0A92NVKZMX) |
| Claude (claude.ai) | Project: Marketing & Comms Hub | Start every session from this project |
| Meta Business Suite | Facebook Page + Instagram | Chris has Full Control admin access |
| Meta Graph API | Access Token (not yet generated) | Requires one-time setup --- see Section 5 |
| Constant Contact | Account Manager plan | API Key: see developer.constantcontact.com |
| Constant Contact API | Device Authorization Flow / Long Lived Tokens | Client ID on Details tab of app '3-9' |
| Anthropic API | ANTHROPIC_API_KEY | Stored as Windows Machine environment variable |
| OneDrive | Boys and Girls Clubs of Lanier tenant | All sorted content stored here |
| ImageMagick | Installed on Chris's machine | Required for HEIC to JPG conversion and image resizing |

### Key Slack IDs

| Name | Slack ID | Role |
|---|---|---|
| Chris Tyson | U0A92NVKZMX | Communications --- primary user |
| Kendall | U07GMQ0KFTP | Executive Director / Workspace Admin |
| #general | C07GQLM63DY | Main channel --- staff post photos here |
| #social-media-queue | C0AKENC746N | Automation notifications and cluster flags |

---

## 3. File Paths

| Folder | Path |
|---|---|
| SlackExport | C:\Users\Chris Tyson\OneDrive - Boys and Girls Clubs of Lanier\Pictures\SlackExport |
| BGCL_Social_Posts | C:\Users\Chris Tyson\OneDrive - Boys and Girls Clubs of Lanier\Documents\BGCL_Social_Posts |
| Sort Script | C:\Users\Chris Tyson\OneDrive - Boys and Girls Clubs of Lanier\Documents\Sort-BGCLPhotos.ps1 |
| Caption Backfill Script | C:\Users\Chris Tyson\OneDrive - Boys and Girls Clubs of Lanier\Documents\Generate-BGCLCaptions.ps1 |
| Watcher Script | C:\Users\Chris Tyson\OneDrive - Boys and Girls Clubs of Lanier\Documents\Watch-BGCLPhotos.ps1 |
| Watcher Installer | C:\Users\Chris Tyson\OneDrive - Boys and Girls Clubs of Lanier\Documents\Register-BGCLWatcher.ps1 |
| Priority Queue | BGCL_Social_Posts\\_PRIORITY (subfolders named PRIORITY_YYYY-MM-DD_HHMM_Description) |

**Subfolder naming convention:** YYYY-MM-DD_Topic (sorts chronologically; contributor name is not included).

---

## 4. The Workflow --- Step by Step

### Step 1 --- Download Photos from Slack (Manual)

Open Slack and go to #general. Click the Files tab in the top right of the channel. Download new photos to the SlackExport folder. Groups of photos attached to the same message can be downloaded together. This is the only manual step in the workflow.

> Known limitation: Slack's free plan does not allow bulk downloads or third-party file access. Upgrading to Slack Pro ($7.25/user/month) would remove this limitation entirely and allow full automation of this step. Worth raising with Kendall as a productivity investment.

### Step 2 --- File System Watcher Detects New Files (Automated)

Windows Task Scheduler runs Watch-BGCLPhotos.ps1, which monitors the SlackExport folder using a File System Watcher. When new files are detected, a 60-second countdown begins. This delay allows multiple files from the same download session to arrive before processing begins. Once the 60-second window closes with no new files, the sorting script fires automatically.

**Status: Complete.** Installed via Register-BGCLWatcher.ps1. Configured to start at logon and restart on failure up to 5 times.

### Step 3 --- Sorting Script Runs (Automated)

Sort-BGCLPhotos.ps1 (v2.2) runs automatically. It:

- Reads EXIF metadata to determine the date each photo was taken
- Groups photos by date and 2-hour time window
- Calls the Claude API (claude-haiku-4-5-20251001) to analyze each group and generate a descriptive topic slug
- Creates a folder named YYYY-MM-DD_Topic in BGCL_Social_Posts
- Moves each photo into its folder, converting HEIC files to JPG via ImageMagick
- Skips files already recorded in sorted-manifest.txt
- Never modifies existing folders in BGCL_Social_Posts

To run manually:

```
powershell -ExecutionPolicy Bypass -File "C:\Users\Chris Tyson\OneDrive - Boys and Girls Clubs of Lanier\Documents\Sort-BGCLPhotos.ps1"
```

> Note: Photos that arrive without EXIF data and without a timestamp in the filename fall back to file creation date. These may cluster incorrectly if multiple unrelated batches were downloaded on the same day. Review any folder named with today's date if the topic slug looks wrong.

### Step 4 --- Caption Generation (Automated)

Caption generation runs automatically at the end of each sort run. For each new folder, Sort-BGCLPhotos.ps1 sends up to 3 sample images to the Claude API along with the folder name and writes platform-specific captions to CAPTIONS.txt inside the folder.

Captions are generated for Instagram, Facebook, and LinkedIn following BGCA brand voice rules. Folders that already have a CAPTIONS.txt are always skipped.

**Backfill script:** Generate-BGCLCaptions.ps1 walks all existing folders in BGCL_Social_Posts and generates captions for any folder missing a CAPTIONS.txt. Safe to re-run at any time.

To run the backfill manually:

```
powershell -ExecutionPolicy Bypass -File "C:\Users\Chris Tyson\OneDrive - Boys and Girls Clubs of Lanier\Documents\Generate-BGCLCaptions.ps1"
```

Caption standards: assertive but not aggressive, uplifting but not unrealistic, expert but not technical, empathetic but not passive, inclusive but not unfocused. Standard hashtags on every post: #GreatFuturesStartHere and #BGCLanier. Instagram uses 5 to 7 hashtags. Facebook uses 2 to 3. LinkedIn uses professional topical hashtags only.

### Step 5 --- Photo Filtering (Automated with Flags)

Not yet built. Claude will evaluate each photo against quality criteria before it is queued for posting. Photos that do not pass will be flagged in #social-media-queue with the reason and held for review. They are never deleted.

Criteria will include:
- Is the image in focus and well lit?
- Are identifiable minors present? If yes, assume media release is on file and proceed.
- Is anything visible in the background that would be inappropriate for public posting?
- Is the image a duplicate or near-duplicate of another in the same batch?
- Does it add anything beyond what other photos in the batch already show?

### Step 6 --- Cluster Detection (Automated with Flags)

Not yet built. The script will check whether new folders from the same date appear thematically related. If a potential cluster is detected, it sends a notification to #social-media-queue with three options: merge into one post, keep separate on consecutive days, or hold for manual review.

### Step 7 --- Posting to Meta and Constant Contact (Automated)

Not yet built. Approved content will be posted automatically via the Meta Graph API (Facebook and Instagram) and the Constant Contact API (LinkedIn) on the following schedule:

- Tuesday, Thursday, Saturday --- Facebook and Instagram at 11:00 AM or 4:30 PM
- Tuesday and Thursday --- LinkedIn at 11:00 AM

### Priority Queue

Time-sensitive content goes into the _PRIORITY subfolder inside BGCL_Social_Posts. Name the subfolder using the format PRIORITY_YYYY-MM-DD_HHMM_Description. The script detects it, generates captions, and posts at the specified time rather than waiting for the regular schedule.

### Grants Queue

Grant-required social media posts have hard deadlines tied to reporting periods and are treated as priority content. Grant posts are mapped to required dates and built into the priority queue with deadline alerts sent to #social-media-queue in advance. The 2026 Grant Management Calendar is the authoritative source for social media deadlines.

---

## 5. API Setup (One-Time Steps)

### Anthropic API

The ANTHROPIC_API_KEY is stored as a Windows Machine-level environment variable on Chris's machine. It was set on March 24, 2026. The key is named BGCL-Automation in the Anthropic Console at console.anthropic.com under the BGCL organization.

To verify the key is set, run in PowerShell:

```powershell
if ([System.Environment]::GetEnvironmentVariable('ANTHROPIC_API_KEY', 'Machine')) {
    Write-Host "ANTHROPIC_API_KEY is set." -ForegroundColor Green
} else {
    Write-Host "ANTHROPIC_API_KEY is NOT set." -ForegroundColor Red
}
```

To set or replace the key, run in an elevated (Run as Administrator) PowerShell window:

```powershell
[System.Environment]::SetEnvironmentVariable('ANTHROPIC_API_KEY', 'your-key-here', 'Machine')
```

Never paste the key value into chat, email, or any document. Store it only as an environment variable.

### Meta Graph API

Requires one-time token generation through the Facebook Developer account. Chris has Full Control admin access on the BGCL Meta Business Suite account.

- Go to developers.facebook.com and log in with BGCL Facebook credentials
- Create a new app or use an existing one associated with the BGCL Page
- Generate a Page Access Token with permissions: pages_manage_posts, pages_read_engagement, instagram_basic, instagram_content_publish
- Store the token as a Windows environment variable --- never hardcoded

> Meta access tokens expire. The posting script will verify token validity at the start of every run and send a notification to #social-media-queue if re-authentication is needed.

### Constant Contact API

The Constant Contact application '3-9' has been created on the developer portal with Device Authorization Flow and Long Lived Refresh Tokens.

- Application name: 3-9
- OAuth Type: DEVICE
- Refresh token type: Long Lived
- Client ID: saved separately

Device Authorization Flow does not use a Client Secret. The first time the posting script runs, it will display a device code and a URL for browser authorization. After that, the Long Lived token handles all future runs without re-authentication.

---

## 6. Known Limitations & Mitigations

### Slack Free Plan --- Manual Download Required

Slack's free plan does not allow third-party tools to access or download files programmatically. The one manual step in the workflow cannot be automated without upgrading to Slack Pro ($7.25/user/month).

### Photos Without EXIF Data

Slack strips EXIF metadata from photos before delivery. Photos that also lack a timestamp in their filename fall back to file creation date. If multiple unrelated batches were downloaded on the same day, they may be grouped into a single folder incorrectly. Review any new folder whose topic slug looks wrong and rename or reorganize as needed.

### Claude Has No Persistent Memory Between Sessions

Each Claude session starts fresh. This document is the mitigation. Start every session from the Marketing & Communications Hub project in Claude and share this document so Claude can pick up where things left off.

### HEIC Files Require ImageMagick

iPhone photos posted to Slack arrive as HEIC files. The sorting script converts them to JPG automatically, but this requires ImageMagick installed on Chris's machine. Download at imagemagick.org/script/download.php#windows.

### Instagram Stories --- Interactive Elements

Instagram does not allow third-party tools to auto-apply music, stickers, polls, links, or other interactive elements to Stories. Clean photo and video Stories can be auto-published. Stories requiring interactive elements need to be finished manually in the Instagram app.

### Caption Quality Depends on Photo Clarity

Caption generation combines image analysis with the folder name for context. Posts sorted into clearly named folders produce more specific captions. Low-quality or ambiguous photos may produce more general captions. Review CAPTIONS.txt for any folder where the topic slug is vague before posting.

---

## 7. Starting a New Claude Session --- Handoff Instructions

### Step 1 --- Open the Right Project

Start every session from the Marketing & Communications Hub project in Claude.

### Step 2 --- Share This Document

Share this document at the start of the session. At minimum, share the Build Status table from Section 1. This tells Claude exactly where things stand.

### Step 3 --- State What You Need

Be specific about what the session is for. Examples: 'I have new photos from #general to process', 'I need to build the next piece of the posting script', or 'I need to review captions for a specific folder.'

### Step 4 --- Update This Document

At the end of any session where something changes, update this document and push it to GitHub before closing out.

> REMINDER TO CLAUDE: If you are reading this in a new session, start by confirming the Build Status table in Section 1 with the user before taking any action. Ask what has changed since the last update date on the cover page. Do not assume the document reflects the current state --- verify first.

---

## 8. Script Reference

### Sort-BGCLPhotos.ps1 (v2.2)

Primary automation script. Runs automatically via Task Scheduler when new files arrive in SlackExport. Also safe to run manually at any time.

What it does: reads EXIF dates, clusters by date and 2-hour window, calls Claude API for topic slug, creates YYYY-MM-DD_Topic folders, moves and converts files, generates CAPTIONS.txt for each new folder.

### Generate-BGCLCaptions.ps1

One-time backfill script. Walks all folders in BGCL_Social_Posts and generates CAPTIONS.txt for any folder that does not already have one. Safe to re-run --- existing CAPTIONS.txt files are never overwritten.

### Watch-BGCLPhotos.ps1

File System Watcher. Monitors SlackExport for new files with a 60-second debounce, then triggers Sort-BGCLPhotos.ps1.

### Register-BGCLWatcher.ps1

Installs Watch-BGCLPhotos.ps1 as a Windows Task Scheduler task. Configured to start at logon and restart on failure up to 5 times. Run once to install --- no ongoing maintenance needed.
