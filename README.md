# Repository Setup and Cloning Guide

> **Detailed step-by-step guide to set up the Repo tool and clone SDV automotive workspaces (`cluster_ws` and `infotainment_ws`) using WSL via VS Code (recommended) or Git Bash on Windows.**

***

## Prerequisites

### Option A: WSL via VS Code (Recommended) 

**WSL provides full Linux environment ideal for Yocto builds and automotive source management.**

#### 1. Install WSL2
```powershell
# Open PowerShell as Administrator
wsl --install
```

**Complete Flow:**
```
wsl --install (PowerShell Admin)
    ↓ Restart PC
Ubuntu appears in Start Menu
    ↓ CLICK Ubuntu
"Installing..." (1-5 min)
    ↓
"Enter new UNIX username: " → your_username
"Enter new password: " → ********
Success! → username@computer:~$
```

**Verify:** `wsl -l -v` (should show **WSL 2**)

#### 2. Install VS Code + WSL Extension
1. [Download VS Code](https://code.visualstudio.com/)
2. Install **WSL extension**: `Ctrl+Shift+X` → search **"WSL"**
3. **Connect**: `Ctrl+Shift+P` → **"WSL: Connect to WSL"**
   - New VS Code window opens (**WSL: Ubuntu** bottom-left)

#### 3. Install Dependencies (WSL Terminal)
```bash
sudo apt update
sudo apt install git python3 curl -y
```

#### 4. Setup Repo Tool
```bash
mkdir -p ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
echo 'export PATH=~/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
repo --version  # ✅ Verify installation
```

### Option B: Git Bash (Windows Native)
⚠️ **Use Git Bash as Administrator** for cloning

1. [Download Repo Tool](https://storage.googleapis.com/git-repo-downloads/repo)
2. Create folder: `C:\Users\YourName\bin` → paste `repo` file
3. **Add to PATH**:
   - Search **"Environment Variables"**
   - **Edit System PATH** → **New** → Add `C:\Users\YourName\bin`
4. [Install Python](https://python.org/) → (python pakage manager)  **check "Add to PATH"**

***

## SSH Key Authentication (REQUIRED)

### 1. Choose Your Environment
| Environment | Launch Method | Terminal |
|-------------|---------------|----------|
| **WSL (Recommended)** | VS Code → `Ctrl+Shift+P` → **WSL: Connect to WSL** | WSL Ubuntu |
| **Git Bash** | Right-click → **"Git Bash as Administrator"** | Windows Git Bash |

### 2. Generate Private + Public Keys
***Note*** you can use  only public key or private key based on your choice (when set up the ssh key)
```bash
cd ~  # Go to home directory
ssh-keygen -t rsa -b 4096 -C "your_gitlab_email@rampgroup.com"
```

**EXACT Prompts:**
```
Enter file: [PRESS ENTER] (default)
Enter passphrase: [strong password OR ENTER]
Enter same passphrase: [confirm]
```

**⚠️ IMPORTANT:** If you press **Enter continuously** (no passphrase), you may face authentication issues during `repo init`. Fix with:
```bash
git config --global user.email "your_gitlab_email@rampgroup.com"
git config --global user.name "Your Name"
```

**✅ Files Created:**
```bash
cd .ssh && ls
```
```
├── id_rsa      ← PRIVATE KEY (3KB) - NEVER SHARE
└── id_rsa.pub  ← PUBLIC KEY  (1KB) - UPLOAD TO GITLAB
```

### 3. Secure Private Key (MANDATORY)

**WSL:**
```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_rsa
chmod 644 ~/.ssh/id_rsa.pub
```

**Git Bash (Windows):**
***Note*** you can use  private key by using gitbash follwing this steps, otherwise skip.
```powershell
# PowerShell as Administrator
icacls $env:USERPROFILE\.ssh\id_rsa /inheritance:r
icacls $env:USERPROFILE\.ssh\id_rsa /grant:r "$($env:USERNAME):(R,W)"
```

**Verify:**
```bash
ls -la ~/.ssh/
-rw-------  id_rsa      ← ✅ Correct (600)
-rw-r--r--  id_rsa.pub  ← ✅ Correct (644)
```

### 4. Copy Public Key
**WSL:** 
```bash
cat ~/.ssh/id_rsa.pub
# COPY entire single line (ssh-rsa ... email)
```

**Git Bash:**
```bash
cat ~/.ssh/id_rsa.pub | clip  # Copies to clipboard
```
**Note:** SSH key may not display directly - use clipboard copy(automatically copy)

### 5. Add Public Key to GitLab
1. [https://gitlab.rampgroup.com](https://gitlab.rampgroup.com) → **LOGIN**
2. **Avatar** → **"Preferences"** (wrench icon)
3. **Left sidebar** → **"SSH Keys"**
4. **Click "Add key" button**
5. **Key**: [PASTE public key - entire line]
6. **Title**: `"WSL-Repo-2026"`
7. **Passphrase**: [LEAVE BLANK]
8. **"Add SSH Key"** → ✅ **GREEN CHECKMARK**

### 6. Load Private Key (SSH Agent)
**WSL:**
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa
```

**Git Bash:**
```bash
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_rsa
```

**Verify:** `ssh-add -l`

### 7. Verify SSH Connection
```bash
ssh -T git@gitlab.rampgroup.com
```
**✅ SUCCESS:** `Welcome to GitLab, @username!`

***

## Cloning Workspaces

### WSL (VS Code Terminal) 
**VS Code → `Ctrl+Shift+\`` OR `Ctrl+Shift+P` → Connect to WSL**

#### Cluster Workspace
```bash
cd ~
mkdir cluster_ws && cd cluster_ws
repo init -u git@gitlab.rampgroup.com:sdv/v2/manifest.git -m cluster.xml
repo sync -j$(nproc)     # FAST 
# OR
repo sync                # SLOW (then compare to  -j$(nproc))
```

#### Infotainment Workspace
```bash
cd ~
mkdir infotainment_ws && cd infotainment_ws
repo init -u git@gitlab.rampgroup.com:sdv/v2/manifest.git -m infotainment.xml
repo sync -j$(nproc)     # FAST 
# OR
repo sync                #SLOW (then compare to  -j$(nproc))
```

### Git Bash (Administrator)
```bash
cd ~
mkdir cluster_ws && cd cluster_ws
repo init -u git@gitlab.rampgroup.com:sdv/v2/manifest.git -m cluster.xml
repo sync -j8            # FAST
# OR
repo sync                # SLOW
```

#### Infotainment Workspace
```bash
cd ~
mkdir infotainment_ws && cd infotainment_ws
repo init -u git@gitlab.rampgroup.com:sdv/v2/manifest.git -m infotainment.xml
repo sync -j8    # FAST 
# OR
repo sync                # SLOW (based on size and net speed)
```
***

## File Locations

| Environment | Private Key | Public Key | Workspaces | Windows Access |
|-------------|-------------|------------|------------|----------------|
| **WSL** | `~/.ssh/id_rsa` | `~/.ssh/id_rsa.pub` | `~/cluster_ws` | `\\wsl$\Ubuntu\home\username\` |
| **Git Bash** | `C:\Users\YourName\.ssh\id_rsa` | `C:\Users\YourName\.ssh\id_rsa.pub` | `C:\Users\YourName\cluster_ws` | Direct paths |

***

## Troubleshooting

| Issue | Solution |
|-------|----------|
| **`repo: command not found`** | `source ~/.bashrc` |
| **`Permission denied (publickey)`** | Verify SSH: `ssh -T git@gitlab.rampgroup.com` |
| **Git Bash permissions** | **Run as Administrator** + Windows `icacls` |
| **Slow sync** | `repo sync -j$(nproc)` |
| **No passphrase issues** | `git config --global user.email` + `user.name` |

***
# SDV Automotive Ubuntu/Linux Stable Setup

**One-command automated setup for SDV Cluster & Infotainment workspaces with GitLab SSH authentication**

## 📋 Complete Bash Script (Copy & Paste)

```bash
#!/bin/bash
set -e

echo "=================================================="
echo "              SDV COMPLETE SETUP - AUTO REPO"
echo "=================================================="
echo

# --------------------------------------------------
# 1. INSTALL DEPENDENCIES
# --------------------------------------------------
echo "[INFO] Updating system and installing dependencies..."
sudo apt update -qq
sudo apt install -y git curl python3 openssh-client xdg-utils

echo "[OK] Dependencies ready."
echo

# --------------------------------------------------
# 2. AUTO INSTALL REPO TOOL (No Manual Steps)
# --------------------------------------------------
echo "[INFO] Installing repo tool automatically..."

# Create bin directory and add to PATH
mkdir -p ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo -o ~/bin/repo
chmod a+x ~/bin/repo
export PATH=~/bin:$PATH
echo 'export PATH=~/bin:$PATH' >> ~/.bashrc

# Verify installation
if ! repo --version >/dev/null 2>&1; then
    echo "[ERROR] Repo installation failed."
    exit 1
fi

echo "[OK] Repo tool ready: $(repo --version)"
echo

# --------------------------------------------------
# 3. SSH KEY SETUP (Smart Detection)
# --------------------------------------------------
echo "[INFO] SSH Key Setup..."

mkdir -p ~/.ssh
chmod 700 ~/.ssh

if [ ! -f ~/.ssh/id_ed25519 ]; then
    echo "[INFO] Generating new SSH key..."
    ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_ed25519 -C "sdv-workspace"
    chmod 600 ~/.ssh/id_ed25519
    chmod 644 ~/.ssh/id_ed25519.pub
fi

echo
echo "=================================================="
echo "     COPY THIS SSH PUBLIC KEY TO GITLAB"
echo "=================================================="
echo
cat ~/.ssh/id_ed25519.pub
echo
echo "1. GitLab → Settings → SSH Keys"
echo "2. Paste entire key above → Add Key"
read -p "Press Enter AFTER adding the SSH key to GitLab..."

# SSH Agent & Verification
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# SSH Verification Loop
while true; do
    echo "[INFO] Verifying SSH authentication..."
    if ssh -o StrictHostKeyChecking=no -o ConnectTimeout=10 -T git@gitlab.rampgroup.com >/dev/null 2>&1; then
        echo "[OK] SSH verified successfully!"
        break
    else
        echo "[WARN] SSH authentication failed. Retrying..."
        read -p "Press Enter to retry or Ctrl+C to exit..."
    fi
done
echo

# --------------------------------------------------
# 4. WORKSPACE SELECTION
# --------------------------------------------------
echo "==============================================="
echo "Select Workspace to Clone"
echo "==============================================="
echo "1. Cluster (~10-20GB)"
echo "2. Infotainment (~20-40GB)" 
echo "3. Both (~40-60GB)"
echo

read -p "Enter choice (1-3): " choice

case "$choice" in
    1) MODE="cluster" ;;
    2) MODE="infotainment" ;;
    3) MODE="both" ;;
    *) echo "Invalid selection."; exit 1 ;;
esac

echo
echo "=================================================="
echo "              CLONING STARTED ($MODE)"
echo "=================================================="
echo "💾 Needs 50-100GB free space | ⏱️ 45-120 minutes"
echo

# --------------------------------------------------
# 5. CLEAN + CLONE WORKSPACES
# --------------------------------------------------

if [ "$MODE" = "cluster" ] || [ "$MODE" = "both" ]; then
    echo "[INFO] Cloning Cluster Workspace..."
    rm -rf ~/cluster_ws
    mkdir -p ~/cluster_ws
    cd ~/cluster_ws

    repo init -u git@gitlab.rampgroup.com:sdv/v2/manifest.git -m cluster.xml -v || { echo "[ERROR] Cluster init failed"; exit 1; }
    repo sync -j$(nproc) --force-sync -c --no-clone-bundle -v || { echo "[ERROR] Cluster sync failed"; exit 1; }

    echo "[OK] Cluster workspace ready: ~/cluster_ws ($(du -sh . | cut -f1))"
    xdg-open ~/cluster_ws 2>/dev/null || echo "Open: ~/cluster_ws"
fi

if [ "$MODE" = "infotainment" ] || [ "$MODE" = "both" ]; then
    echo "[INFO] Cloning Infotainment Workspace..."
    rm -rf ~/infotainment_ws
    mkdir -p ~/infotainment_ws
    cd ~/infotainment_ws

    repo init -u git@gitlab.rampgroup.com:sdv/v2/manifest.git -m infotainment.xml -v || { echo "[ERROR] Infotainment init failed"; exit 1; }
    repo sync -j$(nproc) --force-sync -c --no-clone-bundle -v || { echo "[ERROR] Infotainment sync failed"; exit 1; }

    echo "[OK] Infotainment workspace ready: ~/infotainment_ws ($(du -sh . | cut -f1))"
    xdg-open ~/infotainment_ws 2>/dev/null || echo "Open: ~/infotainment_ws"
fi

echo
echo "=================================================="
echo "              ✅ SETUP COMPLETED SUCCESSFULLY"
echo "=================================================="
echo

echo "Workspace locations:"
[ "$MODE" = "cluster" ] && echo "📁 ~/cluster_ws"
[ "$MODE" = "infotainment" ] && echo "📁 ~/infotainment_ws" 
[ "$MODE" = "both" ] && echo "📁 ~/cluster_ws and ~/infotainment_ws"

echo
echo "🚀 Next steps for QA testing:"
echo "1. cd ~/cluster_ws  (or infotainment_ws)"
echo "2. repo status"
echo "3. Open in VS Code: code ."

echo
read -p "Press Enter to exit..."
```

##  Setup Guide

### **Step 1: Create Script File**
```bash
# Open Ubuntu Terminal (Ctrl+Alt+T)
nano sdv-ubuntu-setup.sh
```

### **Step 2: Copy-Paste Script**
```
1. COPY entire script above (Ctrl+A → Ctrl+C)
2. In nano: PASTE (Right-click or Ctrl+Shift+V)
3. SAVE: Ctrl+O → Enter
4. EXIT: Ctrl+X
```

### **Step 3: Make Executable & Run**
```bash
chmod +x sdv-ubuntu-setup.sh
./sdv-ubuntu-setup.sh
```

### **Step 4: Add SSH Key to GitLab (ONE TIME)**
```
Script displays: "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI..."

1. https://gitlab.rampgroup.com → LOGIN
2. Avatar → Preferences (🔧 wrench)
3. Left sidebar → SSH Keys
4. "Add key" button
5. Key: [PASTE ENTIRE LINE from script]
6. Title: "Ubuntu-Repo-2026"
7. Passphrase: [LEAVE BLANK]
8. "Add SSH Key" → ✅ Green checkmark
9. Press Enter in terminal to continue
```

### **Step 5: Select Workspace & Wait**
```
1 = Cluster (~10-20GB, 45min)
2 = Infotainment (~20-40GB, 60min) 
3 = Both (~40-60GB, 120min)
```

## 📁 Final File Structure

```
$HOME/ (your home directory)
├── bin/
│   └── repo                 ← Auto-installed repo tool
├── .ssh/
│   ├── id_ed25519          ← Private key (KEEP SECRET)
│   └── id_ed25519.pub      ← Public key (added to GitLab)
├── cluster_ws/             ← Cluster repos (choice 1 or 3)
└── infotainment_ws/        ← Infotainment repos (choice 2 or 3)
```

## ✅ Prerequisites Check

| Requirement | Auto-Installed | Notes |
|-------------|----------------|-------|
| Ubuntu 20.04+ | ✅ | Tested on 22.04/24.04 |
| Git | ✅ | `sudo apt install git` |
| Python3 | ✅ | `sudo apt install python3` |
| OpenSSH | ✅ | `sudo apt install openssh-client` |
| 100GB+ Free Space | ⚠️ | Check: `df -h ~` |
| Sudo Access | ✅ | Password may be prompted |

##  Storage Details

| Component | Location | Size |
|-----------|----------|------|
| Repo Tool | `~/bin/repo` | 2MB |
| SSH Keys | `~/.ssh/` | 1KB |
| Cluster WS | `~/cluster_ws` | 10-20GB |
| Infotainment WS | `~/infotainment_ws` | 20-40GB |

## 🔧 Customization Options

Edit **VARIABLES** at top of script:
```bash
# Change workspace location
WORKSPACE_ROOT="/mnt/d/SDV"  # External drive

# Different GitLab
GITLAB_HOST="your.gitlab.com"
```

## Troubleshooting

| Error | Fix |
|--------|-----|
| `Permission denied` | `chmod +x sdv-ubuntu-setup.sh` |
| `sudo: command not found` | Use normal Ubuntu terminal |
| `SSH authentication failed` | Add public key to GitLab |
| `Disk full` | `df -h` → Free 100GB space |
| `repo: command not found` | `source ~/.bashrc` |
| `Network timeout` | Check VPN/firewall |



# SDV Automotive Windows Stable Setup



**One-click automated setup for SDV Cluster & Infotainment workspaces with GitLab SSH authentication**

## 📋 Complete Batch Script (Copy & Paste)

```batch
@echo off
title SDV Windows Stable Setup
color 0A
setlocal EnableDelayedExpansion

echo ====================================================
echo SDV Automotive Windows Stable Setup
echo ====================================================
echo.

:: ==============================
:: VARIABLES - EDIT THESE IF NEEDED
:: ==============================
set "HOME_DIR=%USERPROFILE%"
set "BIN_DIR=%HOME_DIR%\bin"
set "REPO_PY=%BIN_DIR%\repo.py"
set "WORKSPACE_ROOT=D:\SDV"
set "GITLAB_HOST=gitlab.rampgroup.com"
set "GITLAB_EMAIL=%USERNAME%@rampgroup.com"

:: ==============================
echo [1/8] Checking Python...
python --version >nul 2>&1 || (
    echo ERROR: Python not installed!
    pause
    exit /b 1
)

echo [2/8] Checking Git...
git --version >nul 2>&1 || (
    echo ERROR: Git not installed!
    pause
    exit /b 1
)

echo [3/8] Checking SSH...
ssh -V >nul 2>&1 || (
    echo ERROR: OpenSSH not available!
    pause
    exit /b 1
)

:: ==============================
echo [4/8] Preparing Repo Tool...
if not exist "%BIN_DIR%" mkdir "%BIN_DIR%"

if not exist "%REPO_PY%" (
    echo Downloading repo tool...
    powershell -Command "Invoke-WebRequest -Uri 'https://storage.googleapis.com/git-repo-downloads/repo' -OutFile '%REPO_PY%'"
    if not exist "%REPO_PY%" (
        echo ERROR: Repo download failed!
        pause
        exit /b 1
    )
) else (
    echo Repo already available.
)

:: ==============================
echo [5/8] Git Configuration...
git config --global user.email "%GITLAB_EMAIL%"
git config --global user.name "%USERNAME%"

:: ==============================
echo [6/8] SSH Setup...
if not exist "%HOME_DIR%\.ssh" mkdir "%HOME_DIR%\.ssh"

if not exist "%HOME_DIR%\.ssh\id_rsa" (
    echo Generating SSH key...
    ssh-keygen -t rsa -b 4096 -C "%GITLAB_EMAIL%" -f "%HOME_DIR%\.ssh\id_rsa" -N ""
)

echo Adding key to agent...
sc config ssh-agent start=auto >nul
net start ssh-agent >nul 2>&1
ssh-add "%HOME_DIR%\.ssh\id_rsa"

echo.
echo ====================================================
echo ADD BELOW KEY TO GITLAB (Step-by-Step below)
echo ====================================================
type "%HOME_DIR%\.ssh\id_rsa.pub"
echo.
pause

echo Testing SSH...
ssh -T git@%GITLAB_HOST%
if errorlevel 1 (
    echo SSH failed. Check GitLab key.
    pause
    exit /b 1
)

echo SSH connection successful!
echo.

:: ==============================
echo [7/8] Workspace Selection
echo C = Cluster
echo I = Infotainment  
echo B = Both
echo.
set /p wschoice="Select C, I, or B: "

if not exist "%WORKSPACE_ROOT%" mkdir "%WORKSPACE_ROOT%"

:: ==============================
echo [8/8] Cloning Workspace...

if /i "%wschoice%"=="C" goto CLUSTER
if /i "%wschoice%"=="I" goto INFOTAINMENT
if /i "%wschoice%"=="B" goto BOTH

echo Invalid selection!
pause
exit /b 1

:CLUSTER
set "TARGET=%WORKSPACE_ROOT%\cluster_ws"
if not exist "%TARGET%" mkdir "%TARGET%"
cd /d "%TARGET%"

python "%REPO_PY%" init ^
--repo-url=https://github.com/GerritCodeReview/git-repo ^
-u git@%GITLAB_HOST%:sdv/v2/manifest.git ^
-m cluster.xml

if errorlevel 1 goto ERROR

python "%REPO_PY%" sync -j8
if errorlevel 1 goto ERROR
goto SUCCESS

:INFOTAINMENT
set "TARGET=%WORKSPACE_ROOT%\infotainment_ws"
if not exist "%TARGET%" mkdir "%TARGET%"
cd /d "%TARGET%"

python "%REPO_PY%" init ^
--repo-url=https://github.com/GerritCodeReview/git-repo ^
-u git@%GITLAB_HOST%:sdv/v2/manifest.git ^
-m infotainment.xml

if errorlevel 1 goto ERROR

python "%REPO_PY%" sync -j8
if errorlevel 1 goto ERROR
goto SUCCESS

:BOTH
call :CLUSTER
call :INFOTAINMENT
goto SUCCESS

:ERROR
echo.
echo ERROR: Repo operation failed!
echo Check:
echo - SSH key added to GitLab
echo - Network access
echo - Manifest permission
pause
exit /b 1

:SUCCESS
echo.
echo ====================================================
echo SDV SETUP COMPLETED SUCCESSFULLY!
echo ====================================================
echo Workspace Location: %WORKSPACE_ROOT%
echo.
pause
exit /b 0
```

##  Setup Guide

### **Step 1: Create Batch File**
```
1. COPY entire script above (Ctrl+A → Ctrl+C)
2. Open Notepad (Win+R → notepad)
3. PASTE script (Ctrl+V)  
4. Save As → sdv-setup.bat
5. Save as type: "All Files (*.*)"
6. ✅ Verify: Gear icon ⚙️ (NOT text icon)
```

### **Step 2: Run as Administrator (MANDATORY)**
```
Right-click sdv-setup.bat → "Run as administrator"
```

### **Step 3: Add SSH Key to GitLab (ONE TIME)**
```
Script shows: "ssh-rsa AAAAB3NzaC1yc2E... your.email@company.com"

1. Go: https://gitlab.rampgroup.com → LOGIN
2. Avatar → Preferences (🔧 wrench)
3. Left sidebar → SSH Keys
4. "Add key" button
5. Key: [PASTE ENTIRE LINE from script]
6. Title: "Windows-SDV-2026"
7. Passphrase: [LEAVE BLANK]
8. "Add SSH Key" → ✅ Green checkmark
9. Press any key in script to continue
```

### **Step 4: Select Workspace**
```
C = Cluster only (~4GB)
I = Infotainment only (~6GB) 
B = Both (~10GB)
```

### **Step 5: Wait for Completion**
```
✅ "SDV SETUP COMPLETED SUCCESSFULLY!"
✅ Press any key to close
```

## 📁 Final File Structure

```
C:\Users\[YourName]\
├── bin\repo.py                 ← Repo tool
└── .ssh\
    ├── id_rsa                  ← Private key (KEEP SECRET)
    └── id_rsa.pub              ← Public key (added to GitLab)

D:\SDV\
├── cluster_ws\                 ← Cluster repos (C or B choice)
└── infotainment_ws\            ← Infotainment repos (I or B choice)
```

## ✅ Prerequisites Check

| Tool | Required | Install From |
|------|----------|--------------|
| Python 3.x | ✅ | python.org |
| Git | ✅ | git-scm.com |
| OpenSSH | ✅ | Windows 10/11 (built-in) |
| D:\ Drive | ✅ | 10GB+ free space |
| Admin Rights | ✅ | Right-click → Run as admin |

##  Customization Options

Edit **VARIABLES section** at top of script:

```batch
set "WORKSPACE_ROOT=E:\SDV"                    # Change drive
set "GITLAB_HOST=your.gitlab.com"             # Different GitLab
set "GITLAB_EMAIL=your.email@company.com"     # Your email
```

## Troubleshooting

| Error | Fix |
|--------|-----|
| `Python not installed` | Install Python + check PATH |
| `Git not installed` | Install Git for Windows |
| `OpenSSH not available` | Enable in Windows Features |
| `SSH failed` | Add public key to GitLab SSH Keys |
| `Permission denied` | Run as Administrator |
| `Repo download failed` | Check internet/firewall |



***




