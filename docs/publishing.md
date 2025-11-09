# Publishing jellycoder to PyPI

This guide assumes you are working from the project root on Windows PowerShell and that you have maintainer access to the [`jellycoder`](https://pypi.org/project/jellycoder/) package namespace.

## 1. Prerequisites

- Python 3.10 or newer with the virtual environment activated (`.\.venv\Scripts\Activate.ps1`).
- Latest tooling installed: `pip install --upgrade pip build twine` (already included in `dev` extras).
- A PyPI account with an API token stored in the Windows credential manager or available as an environment variable (e.g. `setx PYPI_TOKEN "pypi-xxxxxxxx"`).

## 2. Release checklist

1. **Update version**: edit `pyproject.toml` and bump the `version` field. Follow semantic versioning.
2. **Update changelog/readme**: ensure `README.md` reflects the current feature set.
3. **Commit changes**: commit the version bump and documentation updates.
4. **Run quality gates**:
   ```powershell
   python -m pytest
   flake8
   ```
5. **Tag the release** once quality gates pass:
   ```powershell
   git tag vX.Y.Z
   git push origin vX.Y.Z
   ```

## 3. Build the distribution artifacts

```powershell
# Clean any previous build outputs
Remove-Item dist,build -Recurse -Force -ErrorAction SilentlyContinue

# Build sdist (.tar.gz) and wheel (.whl)
python -m build
```

Confirm both files exist in the `dist/` directory.

## 4. Publish to TestPyPI (optional but recommended)

```powershell
$env:TWINE_USERNAME = "__token__"
$env:TWINE_PASSWORD = $env:TEST_PYPI_TOKEN  # create this beforehand

twine upload --repository testpypi dist/*
```

Install from TestPyPI in a throwaway environment to validate installation:

```powershell
python -m pip install --index-url https://test.pypi.org/simple/ --extra-index-url https://pypi.org/simple jellycoder
```

## 5. Publish to PyPI

After verifying TestPyPI, upload to the main index:

```powershell
$env:TWINE_USERNAME = "__token__"
$env:TWINE_PASSWORD = $env:PYPI_TOKEN  # ensure this is set

twine upload dist/*
```

Twine will report the URLs for the uploaded files once the upload succeeds.

## 6. Post-release tasks

- Verify the published metadata on https://pypi.org/project/jellycoder/.
- Bump the version locally to the next development cycle (e.g. `0.1.1-dev`) and commit.
- Regenerate documentation or release notes if applicable.

## 7. Troubleshooting

- `HTTPError: 403` during upload usually means the API token lacks the required scope or the version already exists.
- `twine upload` refuses to upload if the version was previously published—bump the version and rebuild.
- Use `twine check dist/*` if Twine warns about metadata issues before uploading.
