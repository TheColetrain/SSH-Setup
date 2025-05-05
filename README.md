# SSH Key Setup and Workstation Configuration

This repository contains notes, configuration files, and setup instructions for managing SSH keys and standardizing workstation environments for user TheColetrain.

## Purpose
- Document the SSH key generation and configuration process for GitHub access.
- Provide a reference for setting up new workstations consistently.
- Track overall goals and status for workstation configuration.

## Key Files
- `.github/copilot-instructions.md`: Instructions for GitHub Copilot interaction within this repository.
- `.copilot/goals.yaml`: Current goals, status, and next steps for workstation setup.
- `docs/`: Contains detailed setup logs for individual machines (e.g., `Gigabyte-C_Initial_Setup.md`).
- `.editorconfig`: Ensures consistent text file formatting.
- `.gitignore`: Specifies intentionally untracked files.
- `SSH-Setup.code-workspace`: VS Code workspace configuration for this project.

## User-Wide Configuration (Not in this repo)
- Global Git settings (`git config --global user.name`, `git config --global user.email`).
- User-level Copilot configuration: `~/.copilotconfig` (or equivalent path on Windows).
- SSH configuration: `~/.ssh/config` (or equivalent path on Windows).
- SSH keys: Stored securely, typically in `~/.ssh/`. Backups managed via Bitwarden.

## Next Steps
Refer to `.copilot/goals.yaml` for the current action plan.
