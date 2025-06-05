# We Have Python at Home: How to Develop Python on Your Desktop

You can write and run Python code right in your browser using tools like Google Colab or Jupyter Notebooks. But if you want to build larger projects and learn more, it's best to set up Python on your own machine. This guide walks you through installing Python, managing project environments, installing libraries, and running scripts — all in a straightforward way for new coders.

---

## Installing Python on Your Machine

### macOS

1. Open Terminal.
2. If you don't have Homebrew installed, install it by running:

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

   You can learn more about Homebrew at [brew.sh](https://brew.sh).

3. Install Python 3:

   ```bash
   brew install python
   ```

   After this, you'll have `python3` and `pip3` commands ready.

### Linux (Ubuntu/Debian)

1. Open Terminal.
2. Update packages:

   ```bash
   sudo apt update
   ```

3. Install Python 3, pip, and virtual environment tools:

   ```bash
   sudo apt install python3 python3-venv python3-pip
   ```

   > Note: On some Linux distros, the `python3-venv` package must be installed separately.

### Windows

1. Download and install Python 3 from [python.org](https://www.python.org/downloads/).
2. During installation, check **"Add Python to PATH"**.
3. After installation, open Command Prompt or PowerShell and verify Python with:

   ```bash
   python --version
   # or
   python3 --version
   ```

---

## Use `python3` and `pip3` Explicitly

Many systems come with both Python 2 and Python 3 installed. To avoid confusion, always use `python3` and `pip3` commands so you know you're running Python 3.

---

## Virtual Environments: Why and How?

Different projects often require different versions of libraries. Installing everything globally can cause conflicts. Virtual environments let you isolate project-specific libraries so they don't interfere.

The `venv` tool comes bundled with Python 3.3+ by default, so no extra install is needed — except on some Linux distros, where you might need to install the `python3-venv` package.

### Create a Virtual Environment

```bash
python3 -m venv my_project_env
```

- This creates a folder named `my_project_env` containing a separate Python environment for your project.

If you are pushing your code up to a repo remember to add .env to your .gitignore files so you don't upload this extra code (known as dependencies). Depencies are documented via a requirements.txt file and re-installed when running the code on a new machine (or new environment).

You will learn more about this in the installing libraries section.

### Activate the Virtual Environment

- On macOS/Linux:

  ```bash
  source my_project_env/bin/activate
  ```

- On Windows PowerShell:

  ```bash
  .\my_project_env\Scripts\Activate.ps1
  ```

Once activated, your terminal prompt will change to show the environment name. Now, `python3` and `pip3` work within this isolated environment.

> For more details, check the [official Python venv docs](https://docs.python.org/3/library/venv.html).

---

## Running Your Python Script with `__main__`

Use this pattern in your scripts to run code only when the file is executed directly, not when imported:

```python
def main():
    print("Hello, world!")

if __name__ == "__main__":
    main()
```

---

## Installing Libraries with `pip3`

Libraries are packages of reusable code written by others that save you time.

To install a library:

```bash
pip3 install requests
```

This installs the `requests` library in your active virtual environment. Always use `pip3` for Python 3.

---

## Managing Dependencies with `requirements.txt`

To keep track of your project's libraries and versions:

- Save your current libraries to a file:

  ```bash
  pip3 freeze > requirements.txt
  ```

- Later, reinstall all the libraries from that file:

  ```bash
  pip3 install -r requirements.txt
  ```

---

## Choosing Good Libraries & Staying Safe

Use [PyPI](https://pypi.org/) to find Python libraries.

Look for libraries that:

- Are updated regularly (at least once a year)
- Have many downloads (millions is great)

Use tools like [Snyk](https://snyk.io/) or `pip-audit` to check for known vulnerabilities.

Always install specific library versions, for example:

```bash
pip3 install flask==2.2.2
```

Only include libraries you explicitly install in `requirements.txt` to avoid clutter.

---

## Environment Confusion? Here's the Best Practice

Sometimes the command `python` points to Python 2 or a different version than you expect. Workarounds like aliases or symbolic links exist but can cause confusion or system issues.

To keep things simple and safe:

- Always use `python3` explicitly.
- Use virtual environments to define the exact Python and libraries your project needs.

---

## Wrapping Up

Developing Python locally means:

- Installing Python 3 on your computer
- Using virtual environments to isolate projects
- Installing and managing libraries safely with `pip3`
- Keeping track of dependencies with `requirements.txt`

It takes a bit of setup but will make your Python development smoother and your projects easier to share.

Thanks for reading and good luck with your python projects!
Please comment any questions or share some of the projects you are working on.
