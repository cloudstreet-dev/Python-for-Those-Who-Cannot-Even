# Chapter 13: Package Management (pip and the Dependency Hell)

*"Python package management is like a box of chocolates: you never know what versions you're gonna get, and half of them will break your environment."* - Forrest Gump, if he did DevOps

Python has more package managers than ways to format strings. There's pip, conda, poetry, pipenv, pdm, pip-tools, and probably three new ones since I started writing this sentence. Let's explore this chaos together.

## pip: Installing Packages and Breaking Things

```bash
# The basics everyone knows
pip install requests  # Install latest version
pip install requests==2.28.1  # Specific version
pip install requests>=2.25.0  # Minimum version
pip install requests~=2.28.0  # Compatible release (~=2.28.0 means >=2.28.0, <2.29.0)

# The basics nobody remembers
pip install --upgrade requests  # Update package
pip install --force-reinstall requests  # Reinstall even if it exists
pip install --no-deps requests  # Don't install dependencies (chaos mode)
pip install --no-cache-dir requests  # Don't use cache (for when cache is corrupted)

# Installing from places that aren't PyPI
pip install git+https://github.com/user/repo.git  # From git
pip install -e .  # Editable install from current directory
pip install /path/to/local/package  # From local directory
pip install http://example.com/package.tar.gz  # From URL (security nightmare)

# The requirements.txt approach (old reliable)
pip freeze > requirements.txt  # Save current environment
pip install -r requirements.txt  # Recreate environment

# But pip freeze includes everything!
# Your requirements.txt now has 200 packages
# You only directly use 5
# Good luck figuring out which ones
```

## Virtual Environments: Because Global Installation is Too Easy

```bash
# Create virtual environment (the old way)
python -m venv myenv

# Activate it (platform-specific, because standards are hard)
source myenv/bin/activate  # Linux/Mac
myenv\Scripts\activate.bat  # Windows (Command Prompt)
myenv\Scripts\Activate.ps1  # Windows (PowerShell)

# Now you're in a virtual environment
which python  # Points to myenv/bin/python
pip install requests  # Installs only in myenv

# Deactivate
deactivate

# The venv module (built-in since Python 3.3)
python -m venv myenv --prompt="MyProject"  # Custom prompt
python -m venv myenv --system-site-packages  # Access system packages
python -m venv myenv --clear  # Delete existing environment

# virtualenv (the third-party predecessor)
pip install virtualenv
virtualenv myenv  # Slightly different from venv
# Why do both exist? Because Python.

# virtualenvwrapper (making virtual envs bearable)
pip install virtualenvwrapper
mkvirtualenv myproject  # Create and activate
workon myproject  # Switch to environment
deactivate  # Leave environment
rmvirtualenv myproject  # Delete environment

# pyenv (for Python version management)
# Not a virtual environment tool, but everyone confuses them
pyenv install 3.11.0  # Install Python version
pyenv local 3.11.0  # Set local Python version
pyenv virtualenv 3.11.0 myenv  # Create venv with specific Python
```

## Poetry: Pipenv's Replacement's Replacement

```bash
# Poetry: Modern dependency management
pip install poetry

# Initialize new project
poetry new myproject  # Creates project structure
# Or in existing project
poetry init  # Interactive setup

# pyproject.toml (the new hotness)
```

```toml
[tool.poetry]
name = "myproject"
version = "0.1.0"
description = "Another Python project"
authors = ["Your Name <you@example.com>"]

[tool.poetry.dependencies]
python = "^3.11"
requests = "^2.28.0"
pandas = "^1.5.0"

[tool.poetry.dev-dependencies]
pytest = "^7.2.0"
black = "^22.0.0"

[build-system]
requires = ["poetry-core>=1.0.0"]
build-backend = "poetry.core.masonry.api"
```

```bash
# Poetry commands
poetry install  # Install dependencies
poetry add requests  # Add dependency
poetry add --dev pytest  # Add dev dependency
poetry remove requests  # Remove dependency
poetry update  # Update dependencies
poetry lock  # Update lock file
poetry show  # Show installed packages
poetry run python script.py  # Run with poetry's environment
poetry shell  # Activate virtual environment

# Poetry creates poetry.lock
# Like package-lock.json for Python
# Ensures everyone gets exact same versions
```

## Pipenv: The Tool That Promise to Fix Everything

```bash
# Pipenv: Pip + virtualenv
pip install pipenv

# Create Pipfile and virtual environment
pipenv install requests  # Add dependency
pipenv install --dev pytest  # Add dev dependency

# Pipfile (yet another format)
```

```toml
[[source]]
url = "https://pypi.org/simple"
verify_ssl = true
name = "pypi"

[packages]
requests = "*"
django = ">=4.0"

[dev-packages]
pytest = "*"

[requires]
python_version = "3.11"
```

```bash
# Pipenv commands
pipenv install  # Install from Pipfile
pipenv install --dev  # Include dev dependencies
pipenv lock  # Generate Pipfile.lock
pipenv sync  # Install from lock file
pipenv run python script.py  # Run in environment
pipenv shell  # Activate shell
pipenv graph  # Show dependency graph
pipenv check  # Security vulnerability check
pipenv uninstall requests  # Remove package
pipenv --rm  # Remove virtual environment

# The problem with Pipenv:
# - Slow
# - Complex dependency resolver
# - Abandoned, then not, then maybe?
# - Everyone switched to Poetry
```

## Conda: For When You Give Up on Pure Python

```bash
# Conda: Package manager for everything, not just Python
# Install Miniconda or Anaconda (3GB download!)

# Create environment with specific Python
conda create -n myenv python=3.11
conda activate myenv  # Works the same on all platforms!
conda deactivate

# Install packages
conda install numpy pandas scipy  # From conda repository
conda install -c conda-forge some-package  # From conda-forge channel
pip install some-pure-python-package  # Still can use pip

# Environment management
conda env list  # List environments
conda env export > environment.yml  # Export environment
conda env create -f environment.yml  # Create from file
conda env remove -n myenv  # Delete environment

# The good:
# - Handles non-Python dependencies (C libraries, etc.)
# - Consistent across platforms
# - Great for data science

# The bad:
# - Huge
# - Slow
# - Separate ecosystem from PyPI
# - Mixing conda and pip is dangerous
```

## The Dependency Resolution Problem

```python
# Why dependency resolution is hard

# Package A requires: B>=1.0, C>=2.0
# Package B requires: C<2.0
# You want to install A and B
# Conflict! No valid solution

# requirements.txt (doesn't handle conflicts)
packageA>=1.0
packageB>=2.0
packageC>=3.0  # But what if A needs C<3.0?

# pip's old resolver (before 20.3)
# Would happily install conflicting versions
# Your code would crash at runtime

# pip's new resolver (20.3+)
pip install --use-feature=2020-resolver  # Now default
# Actually checks for conflicts
# Much slower
# Sometimes can't find solution that exists

# Poetry's resolver
# Uses SAT solver (like solving sudoku)
# Can find complex solutions
# Even slower

# Common conflict scenarios:

# The Diamond Dependency
# Your app -> LibraryA -> SharedLib==1.0
#          -> LibraryB -> SharedLib==2.0
# Can't have both versions of SharedLib!

# The Transitive Trap
# You: pip install tensorflow
# TensorFlow: needs numpy>=1.19
# Also you: pip install old-science-lib
# old-science-lib: needs numpy<1.19
# Boom!

# The Version Pin Paradise
# Every package pins exact versions for "stability"
# Nothing can be upgraded
# Security vulnerabilities everywhere
```

## Package Development and Distribution

```python
# setup.py (the old way, still everywhere)
from setuptools import setup, find_packages

setup(
    name="my-package",
    version="0.1.0",
    packages=find_packages(),
    install_requires=[
        "requests>=2.28.0",
        "click>=8.0.0",
    ],
    extras_require={
        "dev": ["pytest>=7.0", "black>=22.0"],
        "docs": ["sphinx>=4.0"],
    },
    entry_points={
        "console_scripts": [
            "my-command=mypackage.cli:main",
        ],
    },
    python_requires=">=3.8",
)

# pyproject.toml (the new way, PEP 517/518)
```

```toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "my-package"
version = "0.1.0"
dependencies = [
    "requests>=2.28.0",
    "click>=8.0.0",
]
requires-python = ">=3.8"

[project.optional-dependencies]
dev = ["pytest>=7.0", "black>=22.0"]
docs = ["sphinx>=4.0"]

[project.scripts]
my-command = "mypackage.cli:main"
```

```bash
# Building and distributing
python setup.py sdist bdist_wheel  # Old way
python -m build  # New way (needs 'build' package)

# Upload to PyPI
twine upload dist/*  # Need to register on pypi.org

# Upload to Test PyPI (for testing)
twine upload --repository testpypi dist/*

# Install from Test PyPI
pip install --index-url https://test.pypi.org/simple/ my-package
```

## Other Tools That Promise to Fix Everything

```bash
# pip-tools: Separate requirements from lock files
pip install pip-tools

# requirements.in (what you actually want)
django
requests
pandas

# Generate requirements.txt with exact versions
pip-compile requirements.in  # Creates requirements.txt
pip-sync requirements.txt  # Install exact versions

# PDM: PEP 582 (no virtualenv needed!)
pip install pdm
pdm init  # Create pyproject.toml
pdm add requests  # Add dependency
pdm install  # Install dependencies
pdm run python script.py  # Run with PDM

# Hatch: The new new thing
pip install hatch
hatch new my-project  # Create project
hatch env create  # Create environment
hatch run python script.py  # Run in environment

# Rye: Rust-based Python manager (because Rust makes everything faster)
curl -sSf https://rye-up.com/get | bash
rye init my-project
rye add requests
rye sync

# uv: Even faster Rust-based pip replacement
pip install uv
uv pip install requests  # 10-100x faster than pip!
uv venv  # Create venv
uv pip compile requirements.in  # Like pip-tools but faster
```

## The State of Python Packaging

```python
# Standards (too many)
# - PEP 517: Build system specification
# - PEP 518: pyproject.toml
# - PEP 621: Project metadata in pyproject.toml
# - PEP 660: Editable installs for PEP 517
# - PEP 582: Python local packages directory

# File formats (also too many)
# - setup.py (legacy but everywhere)
# - setup.cfg (configuration for setup.py)
# - pyproject.toml (the future)
# - requirements.txt (pip)
# - Pipfile/Pipfile.lock (pipenv)
# - poetry.lock (poetry)
# - environment.yml (conda)

# The problems:
# 1. No standard way to manage dependencies
# 2. Virtual environments are confusing
# 3. System Python vs user Python vs project Python
# 4. Binary packages are platform-specific
# 5. Dependency resolution is NP-complete
# 6. Everyone has their own solution

# The reality:
"""
Developer: I just want to install packages
Python: Here are 12 tools, 5 file formats, and 3 competing standards
Developer: I'll just use requirements.txt
Python: That doesn't lock transitive dependencies
Developer: I'll use Poetry
Python: That's not standard
Developer: I'll use pip-tools
Python: That's not integrated
Developer: I'll use Conda
Python: That's a different ecosystem
Developer: I'll use Docker
Python: Now you have two problems
"""
```

## Best Practices (Good Luck)

```bash
# For applications:
# - Use Poetry or Pipenv for dependency management
# - Lock your dependencies
# - Use virtual environments always
# - Don't use system Python
# - Keep dev dependencies separate

# For libraries:
# - Use pyproject.toml
# - Don't pin exact versions (use ranges)
# - Test with multiple Python versions
# - Don't include lock files

# Project structure:
my_project/
├── src/
│   └── my_package/
│       ├── __init__.py
│       └── main.py
├── tests/
│   └── test_main.py
├── pyproject.toml  # or setup.py
├── README.md
├── LICENSE
├── .gitignore
└── requirements.txt  # or Pipfile or poetry.lock

# .gitignore for Python:
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
ENV/
.venv
pip-log.txt
pip-delete-this-directory.txt
.tox/
.coverage
.coverage.*
.cache
.pytest_cache/
.mypy_cache/
.hypothesis/
*.egg-info/
dist/
build/
```

## Exercises: Try Not to Cry

1. **Dependency Conflict Resolution**: Create two packages with conflicting dependencies. Try to install both with pip, Poetry, and Pipenv. Which one handles it best?

2. **Virtual Environment Explorer**: Create virtual environments with venv, virtualenv, Poetry, and Conda. Compare their structures and activation methods.

3. **Package Publisher**: Create a simple package and publish it to Test PyPI. Include dependencies, console scripts, and optional dependencies.

4. **Lock File Analysis**: Create the same project with Poetry, Pipenv, and pip-tools. Compare the lock files they generate.

5. **Speed Test**: Install the same large package (like tensorflow) with pip, conda, and uv. Time each one.

6. **Migration Challenge**: Take a project using requirements.txt and migrate it to Poetry, preserving exact versions.

## Summary: What We've Learned

1. **pip is the standard** (but it's not enough)
2. **Virtual environments are necessary** (but confusing)
3. **requirements.txt is inadequate** (no lock file)
4. **Poetry is popular** (but not standard)
5. **Pipenv promised to fix everything** (it didn't)
6. **Conda is its own ecosystem** (great for data science)
7. **pyproject.toml is the future** (maybe)
8. **Dependency resolution is hard** (NP-complete hard)
9. **There are too many tools** (and more coming)
10. **Nobody agrees on best practices** (welcome to Python)

Next chapter: Testing, where we'll learn why your code definitely has bugs and how pytest makes finding them slightly less painful.

---

*"Python package management is like organizing a garage sale: everyone has a system, none of them are compatible, and you'll end up with stuff you don't need."* - Developer with 5 different virtual environment managers installed