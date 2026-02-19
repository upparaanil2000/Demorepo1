                                    # Repository Setup and Cloning Guide
                                    #Method-1(Manual Cloning Process)

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
                            # Automate the Cloning Process using Scripts

**One-command automated setup for SDV Cluster & Infotainment workspaces with GitLab SSH authentication**



