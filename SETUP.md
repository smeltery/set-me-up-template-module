# Module Setup Guide

This guide will help you create a new set-me-up module using this template.

## Quick Start

1. **Copy this template to create your new module:**

   ```bash
   cp -r modules/template-module modules/your-module-name
   cd modules/your-module-name
   ```

2. **Rename the main script:**

   ```bash
   mv module-name.sh your-module-name.sh
   chmod +x your-module-name.sh
   ```

3. **Update the module script** (`your-module-name.sh`):
   - Replace `MODULE_NAME` with your actual module name
   - Implement the `install_module()` function
   - Implement the `configure_module()` function
   - Add any OS-specific checks needed

4. **Update the README** (`.github/README.md`):
   - Replace `module-name` with your actual module name
   - Add description of what your module does
   - List features and requirements
   - Update platform support checkboxes

5. **Initialize git repository:**

   ```bash
   git init
   git add .
   git commit -m "Initial commit: Create MODULE_NAME module"
   ```

6. **Create a GitHub repository** and push:

   ```bash
   git remote add origin https://github.com/smeltery/set-me-up-YOUR-MODULE.git
   git branch -M main
   git push -u origin main
   ```

7. **Add as submodule to set-me-up:**

   ```bash
   cd /path/to/set-me-up
   git submodule add https://github.com/smeltery/set-me-up-YOUR-MODULE.git modules/your-module-name
   git submodule update --init --recursive
   ```

## Integrating into Your Blueprint Repository

Once your module is created and published, you need to integrate it into
your personal set-me-up blueprint repository.

### Step 1: Navigate to Your Blueprint Repository

```bash
# Go to your personal blueprint repo
cd ~/path/to/your-blueprint-repo
```

### Step 2: Add Module as Submodule

```bash
# Navigate to the dotfiles/modules directory
cd dotfiles/modules

# Add your module as a git submodule
git submodule add https://github.com/YOUR-USERNAME/set-me-up-YOUR-MODULE.git your-module-name

# Initialize and update the submodule
git submodule update --init --recursive
```

### Example: Complete Integration Workflow

```bash
# 1. Navigate to blueprint
cd ~/my-set-me-up-blueprint

# 2. Add submodule
cd dotfiles/modules
git submodule add https://github.com/myusername/set-me-up-docker.git docker

# 3. Commit and push
git add .gitmodules dotfiles/modules/docker
git commit -m "Add Docker module integration"
git push origin main
```

### Updating Module in Blueprint

When you update your module repository, update it in your blueprint:

```bash
cd ~/your-blueprint-repo/dotfiles/modules/your-module-name
git pull origin main
cd ../../..
git add dotfiles/modules/your-module-name
git commit -m "Update your-module-name to latest version"
git push origin main
```

## Module Structure

```text
your-module-name/
├── .github/
│   ├── workflows/
│   │   ├── lint.yml      # Automated linting workflow
│   │   ├── tests.yml     # Automated testing workflow
│   │   └── release.yml   # Automated release workflow
│   ├── LICENSE           # MIT license
│   └── README.md         # Module documentation
├── your-module-name.sh   # Main installation script
├── CHANGELOG.md          # Version history (optional but recommended)
├── SETUP.md             # This setup guide (can be deleted)
└── [config files]        # Optional: any config files needed
```

## Available Utility Functions

The `utilities.sh` file provides many helpful functions:

### OS Detection

- `is_macos` - Check if running on macOS
- `is_debian` - Check if running on Debian-based Linux
- `is_arch_linux` - Check if running on Arch-based Linux
- `is_ubuntu` - Check if running on Ubuntu

### Package Management

- `brew_install "package"` - Install via Homebrew
- `brew_update` - Update Homebrew
- `brew_upgrade` - Upgrade Homebrew packages
- `brew_cleanup` - Clean up Homebrew
- `apt_install "package"` - Install via APT
- `apt_install_from_file "filename"` - Install packages listed in file
- `install_ports_from_file "filename"` - Install MacPorts from file
- `install_pkg_from_URL "url"` - Install .pkg from URL

### Utilities

- `ask_for_sudo` - Request sudo privileges
- `cmd_exists "command"` - Check if command exists
- `action "message"` - Print action message
- `success "message"` - Print success message
- `error "message"` - Print error message
- `warning "message"` - Print warning message
- `initialize_brew` - Initialize Homebrew environment

## Examples

### Example 1: Simple Package Installation

```bash
main() {
    ask_for_sudo

    if is_macos; then
        brew_install "package-name"
    elif is_debian; then
        apt_install "package-name"
    fi
}
```

### Example 2: Installing from a Package List

```bash
# Create a file called 'packages' with one package per line
main() {
    ask_for_sudo

    if is_debian; then
        apt_install_from_file "packages"
    fi
}
```

### Example 3: Multi-step Installation with Configuration

```bash
main() {
    ask_for_sudo

    # Install
    action "Installing MyApp"
    brew_install "myapp"
    success "MyApp installed"

    # Configure
    action "Configuring MyApp"
    cp -f config.yaml "$HOME/.config/myapp/config.yaml"
    success "MyApp configured"
}
```

## Best Practices

1. **Always source utilities.sh** at the beginning of your script
2. **Use OS detection functions** to ensure cross-platform compatibility
3. **Request sudo only when needed** using `ask_for_sudo`
4. **Provide clear feedback** using `action`, `success`, `error` functions
5. **Check if tools exist** before using them with `cmd_exists`
6. **Make scripts idempotent** - safe to run multiple times
7. **Include shellcheck directive** for proper linting
8. **Make your script executable** with `chmod +x`

## Testing Your Module

Before committing, test your module:

```bash
# Run your module script
./your-module-name.sh

# Check for errors
echo $?  # Should be 0 for success

# Verify installation
which your-installed-tool
```

## GitHub Actions Workflows

The template includes three automated workflows:

### Lint Workflow (`.github/workflows/lint.yml`)

Runs automatically on:

- Pull requests that modify `.md`, `.sh`, or `.bash` files
- Pushes to `main` or `master` branch

Performs:

- **Markdown Lint**: Checks markdown formatting
- **Link Checker**: Validates all links in markdown files
- **ShellCheck**: Static analysis for shell scripts
- **Bash Syntax Check**: Validates bash syntax

### Tests Workflow (`.github/workflows/tests.yml`)

Runs automatically on all pushes and pull requests.

Performs:

- **Cross-platform Testing**: Tests on both Ubuntu and macOS
- **Mock Environment**: Creates a mock set-me-up environment with utilities.sh
- **Syntax Validation**: Verifies bash syntax for all scripts
- **Execution Testing**: Runs scripts with mocked dependencies
- **Structure Validation**: Checks for required files and proper structure
- **OS Detection Testing**: Validates OS-specific functionality
- **Test Reports**: Generates detailed test summaries

#### Test Environment

The tests workflow creates a mock environment:

- Mock `utilities.sh` with all common functions
- Package management functions that log instead of installing
- OS detection functions that work in CI
- Safe execution without making system changes

#### Customizing Tests

You can customize tests for your module:

```yaml
# Add module-specific tests in .github/workflows/tests.yml
- name: Test specific functionality
  run: |
    # Your custom test commands
    source "$HOME/set-me-up/dotfiles/utilities/utilities.sh"
    # Test your module's functions
```

### Release Workflow (`.github/workflows/release.yml`)

Runs automatically when you push a version tag (e.g., `v1.0.0`).

Features:

- Extracts version from git tag
- Reads changelog from `CHANGELOG.md` (if present)
- Generates comprehensive release notes
- Creates GitHub release automatically
- Provides installation instructions in release

### Creating a Release

1. **Update CHANGELOG.md** with your changes:

   ```markdown
   ## [1.0.0] - 2026-01-17
   
   ### Added
   - Initial release
   - Feature X
   ```

2. **Commit your changes**:

   ```bash
   git add .
   git commit -m "Prepare v1.0.0 release"
   ```

3. **Create and push a version tag**:

   ```bash
   git tag v1.0.0
   git push origin main
   git push origin v1.0.0
   ```

4. **GitHub Actions will automatically**:
   - Run the release workflow
   - Create a GitHub release
   - Include changelog in release notes
   - Provide installation instructions

### Customizing Workflows

You can customize the workflows for your specific needs:

**Add additional linting:**

```yaml
# In .github/workflows/lint.yml
- name: Custom lint step
  run: |
    # Your custom linting commands
```

**Modify release notes:**
Edit the release notes template in `.github/workflows/release.yml` around line 85-124.

**Change trigger conditions:**
Modify the `on:` section of either workflow file.

## Cleanup

Once you're done setting up:

```bash
# Remove this setup guide
rm SETUP.md

# Commit your changes
git add .
git commit -m "Remove setup guide"
git push
```

## Questions?

See existing modules for reference:

- `modules/xcode/` - Simple macOS-only module
- `modules/macports/` - Module with package file
- `modules/homebrew/` - Complex cross-platform module
