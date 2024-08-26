# Love2D starter

This is a starter template mostly made for myself. It includes:

- Libraries I use often
- A simple script to build and push to itch.io

This has small enough code that it can be used for gamejams.

## Requirements

Before running any scripts, ensure that the following tools and dependencies are installed:

- **Node.js** (v16 or higher) - Required for running the `love-packager` command.
- **Butler** - A command-line tool used for uploading games to [itch.io](https://itch.io/). You can download stable and bleeding-edge builds of butler from its itch.io page https://itchio.itch.io/butler, but I recommend getting the [itch app](https://itch.io/app) and install it there as it's easy.

The `.env` file needs to be populated as well. For github actions, the secrets need to be setup on Github.

You can find your Butler API key locally:

Linux: `~/.config/itch/butler_creds`
Mac: `~/Library/Application Support/itch/butler_creds`
Windows: `%USERPROFILE%\\.config\\itch\\butler_creds`

Or on your API keys user settings page - the key you're looking for will have its source set to wharf.

## Scripts

### `package.sh`

This script packages the Love2D game into different formats (e.g., Windows, Mac, Web) using `love-packager`.

**Usage:**
```bash
./package.sh
```

**Optional Flags:**
- `--increment-version` - Increments the version number in the `packager.yml` file.

**Requirements:**
- Ensure `butler` is installed and the `BUTLER_CREDENTIALS` environment variable is set.

### `package-and-upload.sh`

This script first runs `package.sh` to package the game and then uploads the resulting packages to itch.io using `butler`.

**Usage:**
```bash
./package-and-upload.sh
```

**Optional Flags:**
- `--increment-version` - Increments the version number in the `packager.yml` file before packaging and uploading.

**Requirements:**
- Same as `package.sh`.
- Have a project created on itch with the same project name as teh one in `packager.yml`.
- Web builds will manually have to be selected to run in the page afterwards.

### `test-web-build.sh`

This script uncompresses the Web build (`Web.zip`) created by `package.sh` and starts a simple Python server to test the WebGL compatibility of the shader files.

**Usage:**
```bash
./test-web-build.sh
```

**Requirements:**
- Ensure Python 3 is installed to run the local HTTP server.

## GitHub Actions Workflow

The repository is configured to automatically run `package-and-upload.sh` whenever changes are pushed to the `main` branch.

### GitHub Actions Workflow Configuration

The workflow is defined as follows:

```yaml
name: Package and Upload

on:
  push:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "16"

      - name: Set up Butler
        run: |
          curl -L -o /usr/local/bin/butler https://broth.itch.ovh/butler/linux-amd64/head/butler
          chmod +x /usr/local/bin/butler

      - name: Install dependencies
        run: npm install

      - name: Run package and upload script
        env:
          ITCH_IO_API_KEY: ${{ secrets.ITCH_IO_API_KEY }}
          ITCH_IO_USERNAME: ${{ secrets.ITCH_IO_USERNAME }}
        run: |
          chmod +x package-and-upload.sh
          ./package-and-upload.sh --increment-version
```

### Explanation:

- **`runs-on: ubuntu-latest`:** The workflow runs on the latest Ubuntu environment.
- **Node.js Setup:** Installs Node.js v16, which is required for running `love-packager`.
- **Butler Setup:** Installs `butler` to upload the game to itch.io.
- **Environment Variables:** The `ITCH_IO_API_KEY` and `ITCH_IO_USERNAME` secrets are required for uploading to itch.io. Make sure these secrets are set in your GitHub repository settings.
- **Script Execution:** The `package-and-upload.sh` script is executed with the `--increment-version` flag to package and upload the game.

## Setting Up GitHub Secrets

To use the GitHub Actions workflow, you need to add the following secrets in your GitHub repository settings:

1. **ITCH_IO_API_KEY**: Your itch.io API key.
2. **ITCH_IO_USERNAME**: Your itch.io username.

These secrets will be used by the workflow to upload the packages to your itch.io account.

## Conclusion

This README provides the necessary instructions to package, upload, and test your Love2D game using the provided scripts. Follow the requirements and usage instructions carefully to ensure everything runs smoothly.
```

### GitHub Actions Workflow Review

The provided GitHub Actions workflow is correct, except for a minor adjustment in the script execution step. The correct script name should be used in the `chmod +x` and script execution command:

- Replace `your-script.sh` with `package-and-upload.sh` in the workflow:

```yaml
- name: Run package and upload script
  env:
    ITCH_IO_API_KEY: ${{ secrets.ITCH_IO_API_KEY }}
    ITCH_IO_USERNAME: ${{ secrets.ITCH_IO_USERNAME }}
  run: |
    chmod +x package-and-upload.sh
    ./package-and-upload.sh --increment-version
```

This workflow should be sufficient to run the `package-and-upload.sh` script on GitHub Actions whenever changes are pushed to the `main` branch.
