# SSH Key Setup Summary for Gigabyte-C (Windows 11 / WSL)

## Goal:
Securely connect to GitHub using a single SSH key managed on Windows, accessible from Windows applications (Git Bash, PowerShell, VS Code) and WSL, without entering the passphrase repeatedly after the initial setup.

## Key Pair:
- Name: Gigabyte-C
- Type: Ed25519 (Modern, Secure)
- Location (Windows):
    - Private Key: C:\Users\colee\.ssh\Gigabyte-C
    - Public Key: C:\Users\colee\.ssh\Gigabyte-C.pub
- Passphrase: Set during generation (required for security).

## Steps Taken:

1.  **Check Existing Keys:** Verified no suitable existing keys in C:\Users\colee\.ssh\.
2.  **Generate New Key Pair (PowerShell):**
    - Command: ssh-keygen -t ed25519 -C "Gigabyte-C"
    - Saved as C:\Users\colee\.ssh\Gigabyte-C (private) and .pub (public).
    - Added a strong passphrase.
3.  **Add Public Key to GitHub:**
    - Copied the content of C:\Users\colee\.ssh\Gigabyte-C.pub.
    - Added to GitHub > Settings > SSH and GPG keys > New SSH key, named "Gigabyte-C".
4.  **Configure Windows OpenSSH Agent Service:**
    - Opened Windows Services (services.msc).
    - Set "OpenSSH Authentication Agent" Startup type to "Automatic".
    - Started the service.
    - Confirmed it was running via Get-Service ssh-agent in PowerShell.
5.  **Add Key to Windows Agent (PowerShell):**
    - Command: ssh-add $env:USERPROFILE\.ssh\Gigabyte-C
    - Entered passphrase when prompted.
    - Verified key was added using ssh-add -l.
    - Result: Key is now persistently managed by the Windows agent across reboots. Passphrase only needed when manually adding the key or if the agent service is restarted/stopped.
6.  **Configure WSL to Use Windows Agent:**
    - Installed socat in WSL: sudo apt update && sudo apt install socat.
    - Downloaded npiperelay.exe to a location accessible from WSL (e.g., /mnt/c/path/to/npiperelay.exe).
    - Added script snippet to WSL's ~/.zshrc (or ~/.bashrc) to:
        - Check if socat relay process is running.
        - Start socat listening on /tmp/ssh-agent.sock and relaying to npiperelay.exe which connects to the Windows agent pipe (\\.\pipe\openssh-ssh-agent).
        - Export SSH_AUTH_SOCK=/tmp/ssh-agent.sock.
7.  **Fix WSL SSH Config Conflicts:**
    - Edited WSL's ~/.ssh/config.
    - Commented out conflicting IdentitiesOnly yes and PubkeyAuthentication no directives found under Host * sections.
8.  **Test Connections:**
    - Windows (PowerShell/Git Bash/VS Code Windows Terminal): ssh -T git@github.com -> Success.
    - WSL (WSL Terminal/VS Code WSL Terminal): ssh -T git@github.com -> Success.
    - Git Clone (WSL): git clone git@github.com:TheColetrain/Obsidian2025.git -> Success without passphrase prompt.

## Current Status:
- SSH key pair Gigabyte-C generated and managed by the Windows OpenSSH Agent service for persistence.
- Public key added to GitHub account TheColetrain.
- Both Windows applications and WSL are configured to use the Windows agent.
- SSH connections and Git operations work from both environments without needing the passphrase (after the initial ssh-add).

---
*Instructions for Opti7090_O and XPS15-Laptop to follow.*