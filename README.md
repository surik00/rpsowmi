# Remote PowerShell over WMI


## Built-in support for Cyrillic (DOS) encoding!


Remote PowerShell over WMI (RPSoWMI) enables you to run PowerShell code with support of STDIN, STDOUT, STDERR and return code through Windows Management Instrumentation (WMI) on remote host.

Communication with your PowerShell code is done through 2 named pipes (one for outbound and another for inbound) being created on **executor's** machine, that means your access rights must be enough privileged not only for creation of new process on remote machine but also for access to the named pipes on executor's machine from remote machine.

Getting started:

```powershell
pip install WMI
pip install git+https://github.com/surik00/rpsowmi.git

# OR:
# 1. download archive: https://github.com/surik00/rpsowmi/releases/tag/v0.1
# 2. Unarchive, go to dir
python .\setup.py install
```


### How to use RPSoWMI:

```python
from rpsowmi import RemotePowerShellOverWmi as RPSoWMI
from wmi import WMI  # https://pypi.python.org/pypi/WMI/

rps = RPSoWMI(WMI())
r = rps.execute("Write-Host 'Hello, world'")
print(r.stdout)  # Just showing 'Hello, world'.
```


### Advanced example for Exchange:

```python
from rpsowmi import RemotePowerShellOverWmi as RPSoWMI
from wmi import WMI


PS_PATH = 'C:\\WINDOWS\\system32\\WindowsPowerShell\\v1.0\\powershell.exe'

PS_GET_DIST_GROUPS_CMD = '''Import-Module activedirectory
ForEach ($group in (Get-ADGroup -Properties Name, Mail -Filter 'groupcategory -eq "distribution"' -Server "{DC}")) {{
    Write-Host ($group.Name, $group.Mail) -Separator "{sep}"
}}
'''

command = PS_GET_DIST_GROUPS_CMD.format(DC='DC.my_domain.com', sep=' | ')

rps = RPSoWMI(WMI(), ps_cmd=PS_PATH)
r = rps.execute(command)
print(r.stdout)
```


For more details, read pydoc of rpsowmi.RemotePowerShellOverWmi.

**Known problems**

* Length of your PowerShell code is limited up to around 2,800 characters because the code is tranfered as a part of command line arguments.
* Line separators - CR, LF and CRLF are unified to LF (`\\n`) somewhere in communication between RPSoWMI and your PowerShell code.
* Line separator - LF (`\\n`) may be appended to STDOUT and STDERR even though your PowerShell code doesn't do it.
