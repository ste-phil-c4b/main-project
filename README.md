# Main Project - Git Submodule Workflow Demo

This project demonstrates how to manage feature development across a main repository and git submodules using **GitHub Flow**.

## Quick Start

Clone this repository with all submodules:

```bash
git clone --recurse-submodules https://github.com/ste-phil-c4b/main-project.git
```

Or if you already cloned without submodules:

```bash
git submodule update --init --recursive
```

## Project Structure

```
main-project/
├── README.md           # This file
├── WORKFLOW.md         # Detailed GitHub Flow guide for submodules
├── libs/
│   ├── sub-lib-a/      # Submodule: https://github.com/ste-phil-c4b/sub-lib-a
│   └── sub-lib-b/      # Submodule: https://github.com/ste-phil-c4b/sub-lib-b
└── .gitmodules         # Submodule configuration
```

## Submodules

| Submodule | Repository | Description |
|-----------|------------|-------------|
| sub-lib-a | [ste-phil-c4b/sub-lib-a](https://github.com/ste-phil-c4b/sub-lib-a) | Demo library A |
| sub-lib-b | [ste-phil-c4b/sub-lib-b](https://github.com/ste-phil-c4b/sub-lib-b) | Demo library B |

## Workflow Documentation

For detailed instructions on how to develop features with submodules using GitHub Flow, see **[WORKFLOW.md](WORKFLOW.md)**.

Key scenarios covered:
- Feature development in a submodule only
- Feature spanning main project and submodules
- Cloning and syncing submodules
- Common pitfalls and best practices

## Common Commands

```bash
# Update all submodules to latest commits on their tracked branches
git submodule update --remote

# Check submodule status
git submodule status

# Enter a submodule to work on it
cd libs/sub-lib-a
git checkout -b feature/my-feature
# ... make changes ...
```
