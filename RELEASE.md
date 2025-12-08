# Release Instructions for Developers

This document explains how to create and publish official releases of the Image Interlacement program to PyPI and GitHub.

## Overview

The release process (Option G) consists of two distribution channels:

1. **PyPI Package** - for Python developers and users with `pip`
   - Makes the program installable via `pip install image-interlacement`
   - Requires Python 3.8+ to be installed on the user's system
2. **PyInstaller Executables** - for non-technical end users
   - Standalone binaries for macOS (arm64 & x86_64), Linux, and Windows
   - No Python installation required; users download and run directly
   - Attached to GitHub Releases for easy download

Both are built and published automatically via GitHub Actions when you push a version tag.

## Pre-Release Checklist

Before creating a release, ensure:

- [ ] All feature branches are merged to `main` and tested
- [ ] All unit tests pass: `pytest -v`
- [ ] Code is linted and formatted: `black src/ tests/`
- [ ] README.md is up to date with new features/usage
- [ ] Dependencies in `requirements.txt` and `pyproject.toml` are in sync
- [ ] Update `CHANGELOG.md` with a summary of changes for this version
- [ ] Verify the program works locally: `./.venv/bin/python src/main.py --help`

## Versioning

This project uses [Semantic Versioning](https://semver.org/) (MAJOR.MINOR.PATCH).

- **MAJOR** (e.g., 1.0.0): Breaking changes to the CLI or behavior
- **MINOR** (e.g., 0.1.0): New features that are backward-compatible
- **PATCH** (e.g., 0.1.1): Bug fixes

Update the version in `pyproject.toml` under the `[project]` section:
```toml
[project]
version = "0.1.0"  # Update this before releasing
```

## Release Steps

### 1. Update Version Number

Edit `pyproject.toml` and bump the version:

```toml
[project]
version = "0.2.0"  # Example: from 0.1.0 to 0.2.0
```

### 2. Update CHANGELOG (if it exists)

Add an entry at the top of `CHANGELOG.md`:

```markdown
## [0.2.0] - 2024-12-07

### Added
- New feature X
- New feature Y

### Fixed
- Bug fix A
- Bug fix B

### Changed
- Behavior change Z
```

### 3. Commit Changes

Commit the version update:

```bash
git add pyproject.toml CHANGELOG.md
git commit -m "Bump version to 0.2.0"
```

### 4. Create and Push a Git Tag

Create a tag matching the version (must start with `v`):

```bash
git tag -a v0.2.0 -m "Release version 0.2.0"
git push origin main
git push origin v0.2.0
```

This triggers the GitHub Actions workflow.

### 5. Monitor the GitHub Actions Workflow

1. Go to your repository on GitHub
2. Click **Actions** tab
3. Watch the **Build and Publish Release** workflow
4. The workflow will:
   - Build executables for macOS (arm64 & x86_64), Linux, and Windows
   - Publish the package to PyPI (requires `PYPI_API_TOKEN` secret set in repo)
   - Create a GitHub Release with all executables attached

### 6. Verify the Release

- **PyPI**: Check [PyPI.org](https://pypi.org/project/image-interlacement/) to confirm the package is published
- **GitHub Releases**: Go to your repo's **Releases** page and verify:
  - Release notes are visible
  - All executables are attached and have correct names
  - Download links are working

### 7. (Optional) Announce the Release

Update README or create a release announcement in your issue tracker or docs.

## Troubleshooting

### PyPI Publishing Fails

**Cause**: `PYPI_API_TOKEN` secret is not set in the GitHub repository.

**Solution**:
1. Generate a PyPI API token:
   - Go to [PyPI Account Settings → API Tokens](https://pypi.org/manage/account/token/)
   - Create a new token scoped to this project
2. Add it as a GitHub Actions secret:
   - Go to your repo **Settings → Secrets and variables → Actions**
   - Click **New repository secret**
   - Name: `PYPI_API_TOKEN`
   - Value: (paste your PyPI token)
   - Click **Add secret**

### Executables Won't Build

**Cause**: PyInstaller may have issues with hidden imports or binary dependencies.

**Solution**:
1. Try building locally first:
   ```bash
   pip install pyinstaller
   pyinstaller image-interlacement.spec
   # Check dist/ for the executable
   ```
2. If it works locally but fails in CI, inspect the GitHub Actions logs for error details
3. Update `image-interlacement.spec` or `.github/workflows/release.yml` as needed
4. Test the spec file again locally before pushing

### GitHub Release Not Created

**Cause**: One or more build jobs failed before reaching the release creation step.

**Solution**:
1. Check the **Actions** log for which job failed
2. Fix the issue (e.g., Python version incompatibility, missing dependency)
3. Delete the tag and re-push after fixing:
   ```bash
   git tag -d v0.2.0
   git push origin :v0.2.0  # Delete remote tag
   # (fix the issue)
   git tag -a v0.2.0 -m "Release version 0.2.0"
   git push origin v0.2.0
   ```

## Local Testing Before Release

To test the release process locally without pushing to GitHub:

### Test PyInstaller Build

```bash
# Install build dependencies
pip install pyinstaller

# Build the executable
pyinstaller image-interlacement.spec

# Test the executable
./dist/image-interlacement --help
./dist/image-interlacement composite test_image1.png test_image2.png test_output.png --mode rows
```

### Test PyPI Package Build

```bash
# Install build tools
pip install build

# Build the package
python -m build

# Check the built files
ls -la dist/
```

## Maintaining the Release Process

### Regular Maintenance Tasks

- **Monthly**: Check for dependency updates in `pyproject.toml` and update versions as needed
- **Before each release**: Run the full test suite and linting
- **After each release**: Monitor user issues and feedback on GitHub

### Updating Dependencies

When updating dependencies (e.g., newer Pillow or NumPy):

1. Update `pyproject.toml` and `requirements.txt` with new versions
2. Test locally: `./.venv/bin/python src/main.py composite ... `
3. Run the full test suite: `pytest -v`
4. Create a new release following the steps above

### Handling Patch Releases (e.g., 0.1.1 → 0.1.2)

For bug fixes, follow the same process but with a PATCH version bump:

```bash
# Update pyproject.toml: version = "0.1.2"
git add pyproject.toml
git commit -m "Bump version to 0.1.2 (bug fix)"
git tag -a v0.1.2 -m "Release version 0.1.2"
git push origin main
git push origin v0.1.2
```

## FAQ

**Q: How do non-technical users download the executable?**  
A: They go to the GitHub **Releases** page, find the latest release, and download the executable for their OS.

**Q: Can I undo a release?**  
A: Yes, but it's not ideal:
   1. Delete the tag: `git tag -d v0.2.0` && `git push origin :v0.2.0`
   2. Delete from PyPI (via [PyPI Admin Panel](https://pypi.org/manage/) if needed)
   3. Unpublish the GitHub Release on GitHub
   4. Fix the issue and create a new release

**Q: What if a test fails in CI but passes locally?**  
A: This often indicates environment differences (Python version, OS). Check the GitHub Actions logs and try running the workflow for that specific OS locally (e.g., use Docker for Linux testing on macOS).

**Q: How do I handle pre-releases (alpha, beta)?**  
A: Use version tags like `v0.2.0-alpha.1` and mark the GitHub Release as a **Pre-release** (checkbox in Release editor). Update `.github/workflows/release.yml` to match your pre-release tag format if needed.

## Support

For issues with the release process or GitHub Actions:
- Check the [GitHub Actions logs](https://docs.github.com/en/actions/learn-github-actions/understanding-github-actions)
- Consult [PyPA Publishing Guidelines](https://packaging.python.org/en/latest/guides/publishing-package-distribution-releases-to-pypi-using-github-actions-ci-cd-workflows/)
- Review [PyInstaller Documentation](https://pyinstaller.org/)

---

**Last Updated**: December 7, 2025
