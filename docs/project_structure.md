# SSH-Setup Project Structure

## Directory Layout

```
SSH-Setup/
├── .copilot/
│   └── goals.yaml               # Structured project goals and steps
├── .github/
│   └── copilot-instructions.md  # Repository-specific instructions for Copilot
├── docs/
│   ├── Gigabyte-C_Initial_Setup.md # Setup instructions for the Gigabyte-C machine
│   ├── Workstation_Setup.md     # Unified setup guide for all workstations
│   └── project_structure.md     # This document explaining repository organization
├── .editorconfig                # Editor configuration for consistent coding style
├── .gitignore                   # Specifies files/directories Git should ignore
├── README.md                    # Main project overview and description
└── SSH-Setup.code-workspace     # VS Code workspace configuration file
```

## Component Descriptions

### Configuration and Development Files

- **`.copilot/`**: Contains files specifically for GitHub Copilot's context
  - `goals.yaml`: Structured project goals and implementation steps

- **`.github/`**: Standard location for GitHub-related files
  - `copilot-instructions.md`: Repository-specific instructions for Copilot

- **`.editorconfig`**: Ensures consistent coding style across different editors
  - Standardizes line endings, indentation, charset, etc.

- **`.gitignore`**: Prevents committing unwanted files
  - Excludes logs, secrets, environment files, and other non-versioned content

- **`SSH-Setup.code-workspace`**: VS Code workspace configuration
  - Defines editor settings specific to this project

### Documentation

- **`docs/`**: Directory containing all project documentation
  - `Gigabyte-C_Initial_Setup.md`: Configuration instructions for Gigabyte-C machine
  - `Workstation_Setup.md`: Unified setup guide for all workstations
  - `project_structure.md`: This document explaining the repository organization

- **`README.md`**: Primary entry point for understanding the project
  - Contains project overview, purpose, sync instructions, and usage information

### Version Control

- **`.git/`**: Git repository metadata (automatically created)
  - Contains version history and branch information
  - Not committed to remote repositories

## GitHub Synchronization

This project is synchronized with GitHub to facilitate collaboration and version control.
To maintain synchronization:

1. Clone the repository: `git clone https://github.com/[username]/SSH-Setup.git`
2. Make changes locally
3. Commit changes: `git commit -am "Descriptive message"`
4. Push changes to remote: `git push origin main`
5. Pull remote changes: `git pull origin main`

### Using VS Code's GitHub Integration

VS Code provides built-in GitHub integration through its Source Control panel:

1. Access the GitHub extension in the Activity Bar (same area as Copilot Chat)
2. Use the Source Control panel (Ctrl+Shift+G) to:
   - Stage changes
   - Create commits
   - Push/pull from remote repositories
   - Create and manage branches
3. The GitHub extension allows repository management without leaving VS Code

### Using Copilot CLI for Git Operations

GitHub Copilot Chat provides a terminal interface for executing Git commands directly:

1. Type `/terminal` or `/shell` in the Copilot Chat input
2. Execute Git commands in the terminal interface:
   ```
   git status
   git add .
   git commit -m "Your commit message"
   git push origin main
   ```
3. The terminal interface allows you to perform Git operations without leaving the editor