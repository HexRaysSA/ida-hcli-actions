# Install IDA Pro Action

A GitHub Action to download and install IDA Pro using the Hex-Rays Command Line Interface (hcli). **Handles all dependencies automatically** - no need to set up Python or uv separately!

## ✨ Features

- 🐍 **Automatic Python setup** - Sets up the specified Python version
- 📦 **Automatic uv installation** - Installs the latest uv package manager
- 🔧 **Automatic hcli installation** - Installs Hex-Rays CLI tool
- 📁 **IDA Pro installation** - Downloads and installs IDA Pro to `/opt/ida`
- 🔗 **Environment setup** - Sets `IDADIR` for subsequent steps
- ✅ **Installation verification** - Verifies successful installation

## Usage

```yaml
- name: Install IDA Pro
  uses: your-username/install-ida-action@v1
  with:
    installer-id: 'release/9.1/ida-pro/ida-pro_91_x64linux.run'
    license-id: ${{ secrets.IDA_LICENSE_ID }}
    api-key: ${{ secrets.HCLI_API_KEY }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `installer-id` | IDA Pro installer ID path | ✅ | - |
| `license-id` | Your IDA Pro license ID | ✅ | - |
| `api-key` | Hex-Rays Command Line Interface API key | ✅ | - |
| `python-version` | Python version to set up | ❌ | `3.13` |
| `hcli-version` | ida-hcli version to install | ❌ | `0.0.27` |

## Outputs

| Output | Description |
|--------|-------------|
| `ida-dir` | Directory where IDA Pro was installed (`/opt/ida`) |
| `python-version` | Python version that was set up |

## Environment Variables Set

After installation, the action sets the `IDADIR` environment variable to `/opt/ida` for use in subsequent workflow steps.

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
      
    # That's it! Just install IDA Pro - everything else is handled automatically
    - name: Install IDA Pro
      uses: your-username/install-ida-action@v1
      with:
        installer-id: 'release/9.1/ida-pro/ida-pro_91_x64linux.run'
        license-id: ${{ secrets.IDA_LICENSE_ID }}
        api-key: ${{ secrets.HCLI_API_KEY }}
    
    # Python, uv, and IDADIR are all ready to use!
    - name: Install Python dependencies and run tests
      run: |
        # uv is already available
        uv sync --extra dev
        uv pip install idapro
        
        # IDADIR is automatically set
        echo "IDA installed at: $IDADIR"
        
        # Run your tests
        uv run pytest -v
```

## Matrix Strategy Example

```yaml
name: Test Multiple IDA Versions

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        ida:
          - version: "9.1"
            installer_id: "release/9.1/ida-pro/ida-pro_91_x64linux.run"
          - version: "9.2"
            installer_id: "beta/9.2-beta-3/ida-pro/ida-pro_92_x64linux.run"
      fail-fast: false
      
    name: Test IDA ${{ matrix.ida.version }}

    steps:
    - uses: actions/checkout@v4
    
    - name: Install IDA Pro ${{ matrix.ida.version }}
      uses: your-username/install-ida-action@v1
      with:
        installer-id: ${{ matrix.ida.installer_id }}
        license-id: ${{ secrets.IDA_LICENSE_ID }}
        api-key: ${{ secrets.HCLI_API_KEY }}
        python-version: '3.11'  # Optional: specify Python version
    
    - name: Run tests
      run: |
        uv sync
        uv run pytest --ida-version=${{ matrix.ida.version }}
```

## 🔧 Advanced Configuration

### Custom Python Version

```yaml
- name: Install IDA Pro with Python 3.11
  uses: your-username/install-ida-action@v1
  with:
    installer-id: 'release/9.1/ida-pro/ida-pro_91_x64linux.run'
    license-id: ${{ secrets.IDA_LICENSE_ID }}
    api-key: ${{ secrets.HCLI_API_KEY }}
    python-version: '3.11'
```

### Custom hcli Version

```yaml
- name: Install IDA Pro with specific hcli version
  uses: your-username/install-ida-action@v1
  with:
    installer-id: 'release/9.1/ida-pro/ida-pro_91_x64linux.run'
    license-id: ${{ secrets.IDA_LICENSE_ID }}
    api-key: ${{ secrets.HCLI_API_KEY }}
    hcli-version: '0.0.26'
```

## Secrets Setup

You'll need to add these secrets to your repository:

1. **`IDA_LICENSE_ID`**: Your IDA Pro license identifier
2. **`HCLI_API_KEY`**: Your Hex-Rays Command Line Interface API key

To add secrets:
1. Go to your repository settings
2. Navigate to "Secrets and variables" → "Actions"
3. Click "New repository secret"
4. Add the required secrets

## Notes

- IDA Pro will be installed to `/opt/ida`
- The `IDADIR` environment variable is automatically set for subsequent workflow steps
- This action uses the official Hex-Rays CLI tool for installation
- The action requires `sudo` permissions to create the installation directory

## License

This action is provided as-is. IDA Pro itself requires a valid license from Hex-Rays.

## Contributing

Issues and pull requests are welcome!