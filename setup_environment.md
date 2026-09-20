# Environment Setup — Machine Learning Zoomcamp 2026

This guide documents how to set up a clean, reproducible Python environment for this project on **macOS (Apple Silicon / M1)** using **uv** for Python and dependency management, with **VS Code** + the **Jupyter extension** for running notebooks.

---

## Why uv?

macOS ships with its own system Python (via Xcode Command Line Tools), and it's easy to end up with multiple conflicting Python installs (Homebrew, python.org, pyenv, etc.) on top of that. This leads to confusing errors like:

```
Running cells with 'Python X.X.X' requires the ipykernel package.
```

...where the interpreter VS Code picked isn't the one you actually installed packages into.

**uv** solves this by managing its own isolated Python versions and per-project virtual environments, independent of whatever else is on your system. This guide uses uv exclusively so there's only ever one source of truth for "which Python is running."

---

## Prerequisites

- macOS on Apple Silicon (M1/M2/M3)
- [VS Code](https://code.visualstudio.com/) installed
- VS Code extensions: **Python** (ms-python.python) and **Jupyter** (ms-toolsai.jupyter)
- Terminal access (zsh, the macOS default)

You do **not** need Homebrew Python, pyenv, or a python.org installer for this setup. If you have them installed already, they won't interfere as long as you always select the project's `.venv` interpreter explicitly in VS Code (see [Troubleshooting](#troubleshooting)).

---

## 1. Install uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Or via Homebrew:

```bash
brew install uv
```

Restart your terminal, then verify:

```bash
uv --version
```

---

## 2. Install a Python version with uv

This project uses **Python 3.11**:

```bash
uv python install 3.11
```

Check what uv has available/installed at any time:

```bash
uv python list
```

---

## 3. Clone/create the project and initialize it

```bash
git clone <this-repo-url>
cd ml-zoomcamp-2026
```

If starting fresh instead of cloning:

```bash
mkdir ml-zoomcamp-2026 && cd ml-zoomcamp-2026
uv init
```

`uv init` creates a `pyproject.toml`, which lets you use `uv add` going forward instead of manually managing `pip install` commands.

---

## 4. Create the virtual environment

From the project root:

```bash
uv venv --python 3.11
```

This creates a `.venv/` folder containing an isolated Python 3.11 environment scoped to this project only.

Verify it was created with the right version:

```bash
cat .venv/pyvenv.cfg
```

You should see a Python version line matching 3.11.x.

---

## 5. Install project dependencies

```bash
uv pip install ipykernel jupyter pandas numpy scikit-learn matplotlib
```

Or, if you initialized with `uv init` and have a `pyproject.toml`:

```bash
uv add ipykernel jupyter pandas numpy scikit-learn matplotlib
```

`ipykernel` is required specifically so VS Code can run Jupyter notebook cells against this environment — without it, cell execution fails with the "requires the ipykernel package" error.

Confirm the install:

```bash
uv pip list
```

---

## 6. Connect the environment to VS Code

1. Open the project folder in VS Code (`code .` from the terminal, or File → Open Folder).
2. Open any `.ipynb` notebook.
3. Click the **kernel selector** in the top-right corner of the notebook.
4. Choose **Select Another Kernel** → **Python Environments**.
5. Select the entry whose path points inside this project's `.venv`, e.g.:
   ```
   ml-zoomcamp-2026/.venv/bin/python3.11
   ```

> ⚠️ **Watch out:** VS Code sometimes lists more than one interpreter with the *same version number* (e.g. two "Python 3.11.16" entries) — one might be a system/global install, the other your project's `.venv`. Always check the **file path** shown under each entry, not just the version number, and pick the one inside `.venv`.

If the `.venv` interpreter doesn't appear in the list, choose **Enter interpreter path...** and manually type or browse to:

```
<project-folder>/.venv/bin/python3.11
```

---

## 7. Verify everything is wired up correctly

Run this in a notebook cell:

```python
import sys
print(sys.executable)
```

The output should be a path ending in `.venv/bin/python3.11`. If it points anywhere else (e.g. `/usr/bin/python3`, `/opt/homebrew/bin/python3.12`, or `~/.local/bin/python3.11`), the wrong kernel is selected — go back to step 6.

---

## Day-to-day usage

- **Adding a new package:** `uv add <package>` (or `uv pip install <package>` without a `pyproject.toml`)
- **Running a script:** `uv run python script.py` — no need to manually activate the venv
- **Running Jupyter from the terminal instead of VS Code:** `uv run jupyter notebook`
- **Activating the venv manually (optional):** `source .venv/bin/activate`

---

## Troubleshooting

### "Running cells with 'Python X.X.X' requires the ipykernel package"
The kernel VS Code selected doesn't have `ipykernel` installed. Either:
- Install it into that exact interpreter: `uv pip install ipykernel`, or
- Switch to the correct `.venv` kernel via the kernel picker (step 6).

### Kernel dies immediately with `No module named ipykernel_launcher`
The interpreter VS Code launched has no `ipykernel` module at all — usually because it's a *different* Python than the one you installed packages into (e.g. a Homebrew Python vs. your uv-managed one). Fix: select the `.venv` interpreter explicitly (step 6), don't rely on auto-detection.

### Two kernel entries show the identical version number
This happens when a system-level Python (e.g. `~/.local/bin/python3.11`) and your project's `.venv/bin/python3.11` happen to be the same version. VS Code will list both — check the **path**, not the version, to pick the right one.

### `python3 --version` shows an old/unexpected version in the terminal
You're likely hitting the macOS system stub at `/usr/bin/python3` (tied to Xcode Command Line Tools) rather than your project's venv. This is expected outside an activated `.venv` — don't try to "upgrade" it. Either activate the venv (`source .venv/bin/activate`) or use `uv run python --version` to check the project's actual interpreter.

### Want to start completely clean
```bash
rm -rf .venv
uv venv --python 3.11
uv pip install ipykernel jupyter pandas numpy scikit-learn matplotlib
```
Then re-select the kernel in VS Code (step 6).

---

## Reference: what NOT to touch

- **`/usr/bin/python3`** — Apple's Command Line Tools Python stub. Some system tooling may depend on it; don't delete or modify it.
- **Homebrew Python(s)** — not required for this setup. If present from a previous setup, they can coexist safely as long as you always explicitly select the project's `.venv` kernel in VS Code rather than relying on auto-detection.