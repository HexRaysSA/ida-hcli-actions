# Install IDA GitHub Action

A GitHub Action to download and install IDA Pro using the Hex-Rays Command Line Interface (hcli). **Handles all dependencies automatically** - no need to set up Python or uv separately!

This can be used to include a CI workflow for your plugins. The action works with linux, windows and macos.

## 📋 Prerequisites

Before using this action, you need:

1. **Valid IDA Pro License**: You must have a valid IDA Pro license from Hex-Rays
2. **Hex-Rays CLI API Key**: Create an API key using the Hex-Rays CLI:
   ```bash
   hcli login 
   hcli auth key create
   ```

For detailed instructions, see the [Hex-Rays CLI documentation](https://hcli.docs.hex-rays.com/)

You can check your licenses and API keys on [https://my.hex-rays.com](https://my.hex-rays.com)

### Required Repository Secrets

Add these secrets to your GitHub repository:

1. **`IDA_LICENSE_ID`**: Your IDA Pro license identifier (eg: 96-ABCD-EFGH-00)
2. **`HCLI_API_KEY`**: Your Hex-Rays Command Line Interface API key (eg: hrp-1-f2f4e8.....)

WARNING: Make sure these values are configured as secrets ! Do not hardcode them in your workflow definition 

**To add secrets to your repository:**
1. Go to your repository → Settings → Secrets and variables → Actions
2. Click "New repository secret"
3. Add each required secret with the exact names above

## ✨ Features

- 📁 **IDA Pro installation** - Downloads and installs IDA Pro to temp directory  
- 🔗 **Environment setup** - Sets `IDADIR` and `IDABIN` for subsequent steps
- ✅ **Installation verification** - Verifies successful installation
- 🔧 **Automatic hcli installation** - Installs Hex-Rays CLI tool
- 🐍 **Automatic Python setup** - Sets up the specified Python version
- 📦 **Automatic uv installation** - Installs the latest uv package manager
- 🧹 **Flexible cleanup** - Option to clean up tools or keep them available

## Usage

```yaml
- name: Install IDA Pro
  uses: hexrays/ida-hcli-actions/install-ida@v1
  with:
    installer-id: 'release/9.1/ida-pro/ida-pro_91_x64linux.run'
    license-id: ${{ secrets.IDA_LICENSE_ID }}
    api-key: ${{ secrets.HCLI_API_KEY }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `api-key` | HCLI Api Key | ✅ | - |
| `installer-id` | IDA Pro installer ID (e.g., "release/9.1/ida-pro/ida-pro_91_x64linux.run") | ✅ | - |
| `license-id` | IDA Pro license ID | ✅ | - |
| `hcli-version-specifier` | Version specifier for ida-hcli to install (e.g., ">=0.6.0", "==0.6.0", "~=0.6.0") | ❌ | `>=0.6.0` |
| `clean` | Clean up installation tools after IDA installation (true=clean, false=keep tools) | ❌ | `true` |
| `python-version` | Python version to install | ❌ | `3.10` |

## Outputs

| Output | Description |
|--------|-------------|
| `ida-dir` | Directory where IDA Pro was installed |
| `ida-bin` | Path to IDA Pro binary |


## 🚀 Complete Example Workflow

```yaml
name: Test with IDA Pro

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v4
      
    # Option 1: Clean installation (default) - only IDA Pro remains
    - name: Install IDA Pro (clean)
      uses: hexrays/ida-hcli-actions/install-ida@v1
      with:
        installer-id: 'release/9.1/ida-pro/ida-pro_91_x64linux.run'
        license-id: ${{ secrets.IDA_LICENSE_ID }}
        api-key: ${{ secrets.HCLI_API_KEY }}
    
    # Option 2: Keep tools available for subsequent steps
    - name: Install IDA Pro
      id: ida-install
      uses: hexrays/ida-hcli-actions/install-ida@v1
      with:
        installer-id: 'release/9.1/ida-pro/ida-pro_91_x64linux.run'
        license-id: ${{ secrets.IDA_LICENSE_ID }}
        api-key: ${{ secrets.HCLI_API_KEY }}
        clean: false  # Keep tools for subsequent steps
    
    - name: Use IDA Pro
      run: |
        echo "IDA installed at: ${{ steps.ida-install.outputs.ida-dir }}"
        echo "IDA binary at: ${{ steps.ida-install.outputs.ida-bin }}"
        
        # Run IDA headless
        "${{ steps.ida-install.outputs.ida-bin }}" --help || echo "IDA help not available"
```

## Cross-Platform Matrix Example

```yaml
name: Test IDA Pro Installation

on: [push, pull_request]

jobs:
  test-ida:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        include:
          - { os: ubuntu-latest,  platform: linux,   installer_id: "release/9.1/ida-pro/ida-pro_91_x64linux.run" }
          - { os: windows-latest, platform: windows, installer_id: "release/9.1/ida-pro/ida-pro_91_x64win.exe" }
          - { os: macos-latest,   platform: macos,   installer_id: "release/9.1/ida-pro/ida-pro_91_armmac.app.zip" }
      fail-fast: false

    name: Test IDA 9.1 on ${{ matrix.platform }}

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Install IDA Pro
      id: ida-install
      uses: hexrays/ida-hcli-actions/install-ida@v1
      with:
        installer-id: ${{ matrix.installer_id }}
        license-id: ${{ secrets.IDA_LICENSE_ID }}
        api-key: ${{ secrets.HCLI_API_KEY }}

    - name: Verify Installation
      shell: bash
      run: |
        echo "IDA installed at: ${{ steps.ida-install.outputs.ida-dir }}"
        echo "IDA binary at: ${{ steps.ida-install.outputs.ida-bin }}"
        
        # Check if IDA directory exists
        if [ -d "${{ steps.ida-install.outputs.ida-dir }}" ]; then
          echo "✅ IDA directory exists"
          ls -la "${{ steps.ida-install.outputs.ida-dir }}"
        else
          echo "❌ IDA directory not found"
          exit 1
        fi

    - name: Test IDA Binary
      shell: bash
      run: |
        # Test IDA binary (platform-specific)
        if [ -f "${{ steps.ida-install.outputs.ida-bin }}" ]; then
          echo "✅ IDA binary found at: ${{ steps.ida-install.outputs.ida-bin }}"
          "${{ steps.ida-install.outputs.ida-bin }}" --help || echo "IDA help not available"
        elif [ -f "${{ steps.ida-install.outputs.ida-bin }}.exe" ]; then
          echo "✅ IDA binary found at: ${{ steps.ida-install.outputs.ida-bin }}.exe"
          "${{ steps.ida-install.outputs.ida-bin }}.exe" --help || echo "IDA help not available"
        else
          echo "❌ IDA binary not found"
          exit 1
        fi
```

## 🔧 Advanced Configuration

### Clean Installation (Default)
Only IDA Pro remains after installation - all tools are cleaned up:

```yaml
- name: Install IDA Pro (clean)
  uses: hexrays/ida-hcli-actions/install-ida@v1
  with:
    installer-id: 'release/9.1/ida-pro/ida-pro_91_x64linux.run'
    license-id: ${{ secrets.IDA_LICENSE_ID }}
    api-key: ${{ secrets.HCLI_API_KEY }}
    # clean: true is the default
```

### Keep Tools Available
Keep Python, uv, and hcli for subsequent workflow steps:

```yaml
- name: Install IDA Pro (keep tools)
  uses: hexrays/ida-hcli-actions/install-ida@v1
  with:
    installer-id: 'release/9.1/ida-pro/ida-pro_91_x64linux.run'
    license-id: ${{ secrets.IDA_LICENSE_ID }}
    api-key: ${{ secrets.HCLI_API_KEY }}
    clean: false  # Keep tools available
```

### Custom Python Version

```yaml
- name: Install IDA Pro with Python 3.12
  uses: hexrays/ida-hcli-actions/install-ida@v1
  with:
    installer-id: 'release/9.1/ida-pro/ida-pro_91_x64linux.run'
    license-id: ${{ secrets.IDA_LICENSE_ID }}
    api-key: ${{ secrets.HCLI_API_KEY }}
    python-version: '3.12'
    clean: false
```

### Custom hcli Version

```yaml
- name: Install IDA Pro with specific hcli version
  id: ida-install
  uses: hexrays/ida-hcli-actions/install-ida@v1
  with:
    installer-id: 'release/9.1/ida-pro/ida-pro_91_x64linux.run'
    license-id: ${{ secrets.IDA_LICENSE_ID }}
    api-key: ${{ secrets.HCLI_API_KEY }}
    hcli-version-specifier: '==0.6.0'

- name: Use IDA Pro
  run: |
    echo "IDA binary: ${{ steps.ida-install.outputs.ida-bin }}"
    "${{ steps.ida-install.outputs.ida-bin }}" --version
```

### Cross-job Usage

```yaml
jobs:
  install:
    runs-on: ubuntu-latest
    outputs:
      ida-dir: ${{ steps.ida-install.outputs.ida-dir }}
      ida-bin: ${{ steps.ida-install.outputs.ida-bin }}
    steps:
      - name: Install IDA Pro
        id: ida-install
        uses: hexrays/ida-hcli-actions/install-ida@v1
        with:
          installer-id: 'release/9.1/ida-pro/ida-pro_91_x64linux.run'
          license-id: ${{ secrets.IDA_LICENSE_ID }}
          api-key: ${{ secrets.HCLI_API_KEY }}

  test:
    needs: install
    runs-on: ubuntu-latest
    steps:
      - name: Use IDA from previous job
        run: |
          echo "IDA directory: ${{ needs.install.outputs.ida-dir }}"
          echo "IDA binary: ${{ needs.install.outputs.ida-bin }}"
```

## Notes

- IDA Pro is installed to a temporary directory (`$RUNNER_TEMP/opt/ida`)
- The action provides outputs `ida-dir` and `ida-bin` for subsequent workflow steps
- This action uses the official Hex-Rays CLI tool for installation
- The action works across Linux, Windows, and macOS platforms
- No `sudo` permissions required - uses temporary directories

## License

This action is provided as-is. IDA Pro itself requires a valid license from Hex-Rays.

## Contributing

Issues and pull requests are welcome!