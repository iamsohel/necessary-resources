install

Invoke-WebRequest -UseBasicParsing -Uri "https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1" -OutFile "./install-pyenv-win.ps1"; &"./install-pyenv-win.ps1"

Reopen PowerShell

Run pyenv --version to check if the installation was successful.

Run pyenv install -l to check a list of Python versions supported by pyenv-win  | pyenv install 3.5.2

Run pyenv install <version> to install the supported version  

Run pyenv global <version> to set a Python version as the global version | pyenv global 3.5.2

pyenv uninstall 3.5.2
