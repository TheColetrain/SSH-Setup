# Unified Workstation Setup Guide

This document provides instructions for setting up SSH and related configurations on all workstations, with specific sections for different machine types when necessary.

## Common Setup Steps

### 1. System Update

```bash
# Update package lists
sudo apt update

# Check if nala is installed, if not install it
if ! command -v nala &> /dev/null; then
  echo "Installing nala package manager..."
  sudo apt install -y nala
else
  echo "nala already installed, continuing..."
fi

# Upgrade installed packages (using nala for better output)
sudo nala upgrade -y
```

### 2. Install Required Software

```bash
# Install SSH server and related tools
sudo nala install -y openssh-server openssh-client git

# Install Visual Studio Code (if not on WSL)
if [ -z "$WSL_DISTRO_NAME" ]; then
  sudo apt install -y software-properties-common
  wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
  sudo install -o root -g root -m 644 packages.microsoft.gpg /etc/apt/trusted.gpg.d/
  sudo sh -c 'echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/trusted.gpg.d/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" > /etc/apt/sources.list.d/vscode.list'
  rm -f packages.microsoft.gpg
  sudo nala update
  sudo nala install -y code
fi
```

### 3. Configure SSH Server

```bash
# Backup original SSH configuration
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# Edit SSH configuration
sudo nano /etc/ssh/sshd_config
```

Recommended security settings:
- `PermitRootLogin no`
sudo systemctl restart ssh
```

### 4. Generate and Configure SSH Keys

```bash
# Check if SSH key already exists
if [ -f ~/.ssh/id_ed25519 ]; then
  echo "SSH key already exists. Using existing key."
else
  # Generate SSH key pair (replace MACHINE-NAME with actual machine identifier)
  ssh-keygen -t ed25519 -C "MACHINE-NAME-Key"
fi

# Start the SSH agent
eval "$(ssh-agent -s)"

# Add key to SSH agent
ssh-add ~/.ssh/id_ed25519
```

### 5. SSH Key Distribution

```bash
# Display your public key to copy to other machines
cat ~/.ssh/id_ed25519.pub
```

### 6. Testing the Connection

```bash
# Test connection (replace with actual hostname/IP)
ssh username@hostname
```

## Windows-WSL SSH Configuration

For workstations using Windows with WSL, follow these steps to share SSH keys:

### 1. Copy SSH Keys from WSL to Windows

```bash
# First, ensure the .ssh directory exists in Windows
WINDOWS_HOME=$(wslpath "$(wslvar USERPROFILE)")
mkdir -p "$WINDOWS_HOME/.ssh"

# Copy the SSH key to Windows
cp ~/.ssh/id_ed25519* "$WINDOWS_HOME/.ssh/"
```

### 2. Set Correct Permissions in Windows

After copying the keys, you must set proper permissions in Windows:
1. Open Windows Explorer and navigate to your Windows user folder (typically C:\Users\yourusername)
2. Navigate to the .ssh folder
3. Right-click on each key file (id_ed25519 and id_ed25519.pub)
4. Select Properties → Security → Advanced
5. Click "Disable inheritance" and select "Remove all inherited permissions"
6. Add your user with Full Control permissions
7. Click Apply and OK

### 3. Install and Configure npiperelay for SSH Agent Forwarding

To use your Windows SSH keys from WSL:

```bash
# Check if socat is installed, if not install it
if ! command -v socat &> /dev/null; then
  echo "Installing socat..."
  sudo nala install -y socat
else
  echo "socat already installed, continuing..."
fi

# Create directory for npiperelay
mkdir -p ~/bin

# Check if npiperelay is already installed
if [ ! -f ~/bin/npiperelay.exe ]; then
  # Download and install npiperelay in WSL
  echo "Downloading npiperelay..."
  wget -O ~/bin/npiperelay.exe https://github.com/jstarks/npiperelay/releases/latest/download/npiperelay.exe
  chmod +x ~/bin/npiperelay.exe
else
  echo "npiperelay already installed, continuing..."
fi

# Check if SSH agent forwarding configuration already exists in bashrc
if ! grep -q "SSH_AUTH_SOCK" ~/.bashrc; then
  echo "Adding SSH agent forwarding configuration to bashrc..."
  cat << 'EOF' >> ~/.bashrc

# SSH agent forwarding from Windows
export SSH_AUTH_SOCK=$HOME/.ssh/agent.sock
ss -a | grep -q "$SSH_AUTH_SOCK"
if [ $? -ne 0 ]; then
  rm -f $SSH_AUTH_SOCK
  (setsid socat UNIX-LISTEN:$SSH_AUTH_SOCK,fork EXEC:"$HOME/bin/npiperelay.exe -ei -s //./pipe/openssh-ssh-agent",nofork &) >/dev/null 2>&1
fi
EOF
  # Source the updated bashrc
  source ~/.bashrc
else
  echo "SSH agent forwarding already configured, continuing..."
fi
```

### 4. WSL Integration Enhancements

Optimize the WSL-Windows experience:

```bash
# Add Windows executables to WSL PATH
echo 'export PATH=$PATH:/mnt/c/Windows:/mnt/c/Windows/System32:/mnt/c/Program Files/Microsoft VS Code/bin' >> ~/.bashrc

# Set up Windows home shortcut
echo 'export WINHOME=$(wslpath "$(wslvar USERPROFILE)")' >> ~/.bashrc
echo 'alias cdwin="cd $WINHOME"' >> ~/.bashrc

# Configure Windows Terminal settings (if installed)
TERMINAL_SETTINGS="$WINHOME/AppData/Local/Packages/Microsoft.WindowsTerminal_8wekyb3d8bbwe/LocalState/settings.json"
if [ -f "$TERMINAL_SETTINGS" ]; then
  echo "Windows Terminal found. Review settings at: $TERMINAL_SETTINGS"
fi

# Mount additional Windows drives consistently
cat << 'EOF' > ~/mount-windows-drives.sh
#!/bin/bash
# Mount all available Windows drives in WSL
for drive in {a..z}; do
  if [ -d "/mnt/$drive" ]; then
    echo "Drive $drive already mounted"
  elif [ -b "/dev/sd$drive" ] || [ -b "/dev/nvme${drive}n1" ]; then
    sudo mkdir -p /mnt/$drive
    sudo mount -t drvfs $drive: /mnt/$drive
    echo "Mounted drive $drive to /mnt/$drive"
  fi
done
EOF
chmod +x ~/mount-windows-drives.sh
```

### 5. Graphics and GUI Application Support

For WSL GUI app support:

```bash
# Enable GUI application support in WSL2
sudo apt install -y x11-apps mesa-utils libgl1-mesa-glx

# Test X11 forwarding
echo "export DISPLAY=:0" >> ~/.bashrc
source ~/.bashrc
xeyes  # Should open a GUI window with eyes that follow cursor
```

## Machine-Specific Configuration

### XPS Workstation

Additional steps specific to the XPS workstation:

#### Key Generation
When generating keys, use:
```bash
ssh-keygen -t ed25519 -C "XPS-Workstation-Key"
```

#### XPS-Specific Software
```bash
# Install developer tools commonly used on XPS
sudo apt install -y git-flow visual-studio-code
```

#### Hardware Considerations
- Enable power management settings appropriate for laptop use
- Configure external display settings as needed
- Set up trackpad gesture controls

### Opti Machine

Additional steps specific to the Opti machine:

#### Key Generation
When generating keys, use:
```bash
ssh-keygen -t ed25519 -C "Opti-Machine-Key"
```

#### Opti-Specific Software
```bash
# Install server-focused tools
sudo apt install -y htop iotop
```

#### Hardware Considerations
- Configure performance settings for desktop use
- Set up backup schedules if this is a data storage machine

### Gigabyte-C Machine

See separate document [Gigabyte-C_Initial_Setup.md](Gigabyte-C_Initial_Setup.md) for detailed instructions specific to this machine.

## Cross-Machine Configuration

After setting up multiple workstations:

1. Exchange SSH keys between machines for passwordless authentication:
   ```bash
   # On source machine, copy public key
   cat ~/.ssh/id_ed25519.pub
   
   # On destination machine, add to authorized_keys
   mkdir -p ~/.ssh
   chmod 700 ~/.ssh
   echo "PASTE-KEY-HERE" >> ~/.ssh/authorized_keys
   chmod 600 ~/.ssh/authorized_keys
   ```

2. Configure shared aliases and settings:
   ```bash
   # Create common alias file
   nano ~/.bash_aliases
   ```

3. Set up automated backups or synchronization between workstations:
   ```bash
   # Install rsync if not already available
   sudo apt install -y rsync
   ```

## Git Synchronization

To synchronize your work across workstations using Git:

### 1. Git Initial Setup

```bash
# Configure Git globally
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Set default branch name
git config --global init.defaultBranch main

# Configure useful Git aliases
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
```

### 2. Create or Clone Repositories

For existing projects:
```bash
# Clone repository
git clone git@github.com:username/repository.git

# Or with HTTPS
git clone https://github.com/username/repository.git
```

For new projects:
```bash
# Initialize Git repository
mkdir my-project
cd my-project
git init

# Create initial commit
echo "# My Project" > README.md
git add README.md
git commit -m "Initial commit"
```

### 3. Setup Remote Repository

```bash
# Add remote repository
git remote add origin git@github.com:username/repository.git

# Push to remote repository
git push -u origin main
```

### 4. Synchronize Between Workstations

When switching workstations:

```bash
# Always pull latest changes before working
git pull

# After making changes, commit and push
git add .
git commit -m "Describe your changes"
git push
```

### 5. Git Credential Management

For Windows & WSL:
```bash
# In Windows, ensure Git credential manager is enabled
git config --global credential.helper manager-core

# In WSL, use Windows Git credential manager
git config --global credential.helper "/mnt/c/Program\ Files/Git/mingw64/bin/git-credential-manager-core.exe"
```

## Uniform Workspace Setup

To maintain consistent development environments across all workstations:

### 1. Standard Directory Structure

Create a uniform directory structure on all machines:

```bash
# Create standard directory structure
mkdir -p ~/Projects
mkdir -p ~/Documents/Notes
mkdir -p ~/Scripts/utilities
mkdir -p ~/Downloads/temp
```

### 2. Cross-Platform Directory Structure

Create a workspace organization that functions seamlessly across Windows and WSL:

```bash
# Create standard directories accessible from both Windows and WSL
PROJECTS_DIR=$(wslpath "$(wslvar USERPROFILE)")/Projects
mkdir -p "$PROJECTS_DIR"

# Create symbolic links between Windows and WSL project spaces
ln -sf "$PROJECTS_DIR" ~/Projects

# Create standardized subdirectories
mkdir -p ~/Projects/{personal,work,experiments,archives}

# Create specialized development folders
mkdir -p ~/Projects/work/client-projects
mkdir -p ~/Projects/work/internal-tools

# Add workspace identifier files
touch ~/Projects/README.md
cat << 'EOF' > ~/Projects/README.md
# Development Projects

This directory contains all development projects, organized by category.

## Structure
- personal/: Personal projects and experiments
- work/: Work-related projects
- experiments/: Testing and learning projects
- archives/: Archived projects

## Workspace Setup
This directory is symlinked between Windows and WSL for seamless access.
EOF
```

### 3. Development Environment Configuration Files

Create standardized configuration files:

```bash
# Create EditorConfig file template
cat << 'EOF' > ~/Templates/common-configs/.editorconfig
root = true

[*]
end_of_line = lf
insert_final_newline = true
charset = utf-8
trim_trailing_whitespace = true
indent_style = space
indent_size = 2

[*.{js,ts,jsx,tsx}]
indent_size = 2

[*.{py}]
indent_size = 4

[Makefile]
indent_style = tab

[*.md]
trim_trailing_whitespace = false
EOF

# Create Git configuration template
cat << 'EOF' > ~/Templates/common-configs/.gitconfig
[user]
	name = Your Name
	email = your.email@example.com
[core]
	editor = code --wait
	autocrlf = input
[init]
	defaultBranch = main
[pull]
	rebase = false
[alias]
	st = status
	co = checkout
	br = branch
	ci = commit
	lg = log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative
[credential "https://github.com"]
	helper = 
	helper = !gh auth git-credential
EOF

# Create VS Code workspace settings template
mkdir -p ~/Templates/common-configs/vscode
cat << 'EOF' > ~/Templates/common-configs/vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.renderWhitespace": "boundary",
  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true,
  "terminal.integrated.defaultProfile.linux": "bash",
  "terminal.integrated.defaultProfile.windows": "PowerShell",
  "git.enableSmartCommit": true,
  "workbench.colorTheme": "Default Dark+",
  "workbench.startupEditor": "newUntitledFile"
}
EOF
```

### 4. Project Templates

Create standardized templates for new projects:

```bash
# Create templates directory
mkdir -p ~/Templates/Projects

# Example: Create a basic web project template
mkdir -p ~/Templates/Projects/basic-web
cd ~/Templates/Projects/basic-web

# Add template files
mkdir -p css js images
touch index.html
touch css/style.css
touch js/main.js
touch README.md

# Script to deploy a new project from template
cat > ~/Scripts/utilities/create-project.sh << 'EOF'
#!/bin/bash
if [ $# -lt 2 ]; then
  echo "Usage: $0 <template-name> <project-name>"
  exit 1
fi

TEMPLATE=$1
PROJECT=$2
TEMPLATE_DIR=~/Templates/Projects/$TEMPLATE
PROJECT_DIR=~/Projects/$PROJECT

if [ ! -d "$TEMPLATE_DIR" ]; then
  echo "Template '$TEMPLATE' not found"
  exit 1
fi

cp -r "$TEMPLATE_DIR" "$PROJECT_DIR"
echo "Project created at $PROJECT_DIR"
EOF

chmod +x ~/Scripts/utilities/create-project.sh
```

### 5. Consistent Terminal Experience

Create a unified terminal configuration for all workstations:

```bash
# Install common terminal utilities
sudo nala install -y tmux zsh

# Set up Oh My Zsh (if preferred)
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# Configure Tmux
cat > ~/.tmux.conf << 'EOF'
# Improve colors
set -g default-terminal "screen-256color"

# Set scrollback buffer to 10000
set -g history-limit 10000

# Custom status bar
set -g status-bg black
set -g status-fg white
set -g status-left-length 30
set -g status-left '#[fg=green](#S) #(whoami)'
set -g status-right '#[fg=yellow]#(cut -d " " -f 1-3 /proc/loadavg)#[default] #[fg=white]%H:%M#[default]'
EOF
```

### 6. Cross-Platform Script Management

Create scripts that work across both Windows and WSL:

```bash
# Create bin directory
mkdir -p ~/bin

# Create a wrapper script for cross-platform commands
cat << 'EOF' > ~/bin/run-script
#!/bin/bash
# Detect platform and run appropriate version of script
SCRIPT_NAME="$1"
shift

if [ -n "$WSL_DISTRO_NAME" ]; then
  # Running in WSL
  if [ -f "$HOME/Scripts/wsl/$SCRIPT_NAME" ]; then
    "$HOME/Scripts/wsl/$SCRIPT_NAME" "$@"
  else
    "$HOME/Scripts/$SCRIPT_NAME" "$@"
  fi
else
  # Running in native Linux
  "$HOME/Scripts/$SCRIPT_NAME" "$@"
fi
EOF
chmod +x ~/bin/run-script

# Add to PATH
echo 'export PATH="$HOME/bin:$PATH"' >> ~/.bashrc
```

## Troubleshooting

Common issues and their solutions:
- Permission denied: Check key permissions (`chmod 600 ~/.ssh/id_ed25519`)
- Connection refused: Ensure SSH service is running (`sudo systemctl status ssh`)
- Host identification changed: Update known_hosts file (`ssh-keygen -R hostname`)

### WSL-Specific Troubleshooting

Common issues with WSL and their solutions:

- **WSL networking issues**: Restart the WSL service with `wsl --shutdown` from PowerShell, then restart.
- **File permission problems**: Ensure you're editing WSL files from within WSL, not directly from Windows.
- **Slow performance**: Avoid working directly in /mnt/* paths; use Linux filesystem for better performance.
- **Path translation**: Use `wslpath` to convert between Windows and WSL paths:
  ```bash
  # Convert Windows to WSL path
  wslpath 'C:\Users\username\Documents'
  
  # Convert WSL to Windows path
  wslpath -w /home/username/Documents
  ```
