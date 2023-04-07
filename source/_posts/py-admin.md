---
title: "Python Power Move: Run Commands as Admin with These Simple Examples"
tags:
  - python
  - windows
  - batch
categories:
  - tech summary
date: 2022-03-19
description: Discover how to run Python commands with admin privileges using our step-by-step tutorial. Our guide provides practical examples and best practices for elevating your Python scripts and automating system-level tasks. Whether you're a Windows user or a Linux enthusiast, our tutorial will help you unlock the full potential of Python and enhance your productivity.
---

## Introduction

We often find ourselves needing to automate tasks on Windows machine, 
Although there are many Python libraries out there that supports some
common Windows operations and even cross-platforms. 
It is really hard to substitute Window's Command Prompt and PowerShell, 
as they are extremely useful in cases where we need to access different 
Windows components, configure settings and troubleshooting.

### User Account Control (UAC)

Standard user accounts are for day-to-day activities with less permission,
while the administrator account has elevated access for all features.

For my personal machine, I'm operating on admin
account all time (as the sole user). But Windows, for security reasons,
still treats most of my actions as standard account. 
It only elevates to admin privilege when my operations want to make internal changes
to Windows settings and my machine. 

The UAC feature when enabled, prompts the
user when such action occurred and request for admin access. Additionally, for standard user,
it means they need to ask for administrator account login.

<table style="width: 70%">
    <tr>
        <td>
            <img src="https://imgur.com/7PejmM2.png" alt="admin" width="500px">
            <br/>When the sign in is an <b>administrator type</b> account<br/>
        </td>
        <td>
            <img src="https://www.adminbyrequest.com/Images/Blogs/UAC%20Prompt.png" alt="std" width="500px">
            <br/>When the sign in is a <b>standard type</b> account<br/>
        </td>
    </tr>
</table>

Now, in terms of task automation in Python, we'll also want to 
figure out how to run certain operations with Admin privilege.

### Using `subprocess`

Like most of us, I have been using `subprocess` to evoke `cmd.exe` or `powershell.exe` as desired
by passing arguments as list into the function.

- Command Prompt: `command = ['cmd.exe', '/c', <argument>]`
- PowerShell: `command = ['powershell.exe', '-command', <argument>]`

```python
import subprocess

def runCmd(*args):
    p = subprocess.Popen(
        *args,
        stdout=subprocess.PIPE,
        stderr=subprocess.STDOUT
    )
    out, error = p.communicate()
    return out, error
```

But this doesn't grant the process with admin privilege, and won't notify us with UAC.

How to achieve this? here are some ways to do it.

## Using ShellExecute `runas`

```python
import ctypes
commands = u'/k echo hi'
ctypes.windll.shell32.ShellExecuteW(
        None,
        u"runas",
        u"cmd.exe",
        commands,
        None,
        1
    )
```

`runas` from the [Windows API](https://docs.microsoft.com/en-us/windows/win32/api/shellapi/nf-shellapi-shellexecutew) 
launches an application as Administrator. 
User Account Control (UAC) will prompt the user for consent to run the application 
elevated or enter the credentials of an administrator account used to run the application.

What about a more pythonic approach?

## Using `runas` in Command Prompt

`runas` application runs command
as a different user; it is most commonly used but not limited to perform operation
with administrator account for granting admin access.

> Note: the password is handled outside UAC, which may not
be the desired behavior

```python
command = ['cmd.exe', '/c', 'runas', '/user:administrator', 'regedit']
p = subprocess.Popen(command, stdin=subprocess.PIPE)
p.stdin.write('password')
p.communicate()
```

## Using PowerShell `-Verb Runas`

This is my preferred method, since it is most flexible and also
evokes UAC for admin access.

- **Start-Process**
    ```shell
    Start-Process <executable> -argumentlist <arugments> -Verb Runas
    ```

- **Call operator (&)**

    by adding call operator, we are able to run commands not limited in the environment path,
also not need to worry about spaces in our path.

    ```shell
    & {Start-Process <executable> -argumentlist <arugments> -Verb Runas}
    ```

- **-ExecutionPolicy Bypass**

    Sometimes, a security setting will prevent PowerShell running a _.ps1_ file,
    and we'll need to bypass execution policy:
    
    ```shell
    Start-Process <executable> -ExecutionPolicy Bypass -File <file>
    ```

It is very obvious from the above example,
we can basically use PowerShell to wrap around
anything, including Command Prompt.

```shell
Start-Process cmd.exe -argumentlist '/k "dir"' -Verb Runas
```

To bundle everything together, a working example would look like this in Python:

```python
ps_command = "& {{Start-Process cmd.exe -argumentlist '/k \"dir\"' -Verb Runas}}"
command = ['powershell.exe', '-command', ps_command]
runCmd(command)
```

essentially using `subprocess` to run PowerShell in admin using `-Verb Runas`
to execute command `/k dir` in Command Prompt (which also has elevated access).


### Redirect Output

By doing the above, we are running an executable within a process.
the sacrifice is that it is difficult to pass the output, 
but here's a few workarounds:

**Output results to a file**

```shell
Start-Process cmd.exe -argumentlist '/c "dir"' -redirectStandardOutput "C:\Users\xlei\Desktop\temp.txt"
```

**Output result to the console**

Just the exit code:

```shell
$output = Start-Process cmd.exe -argumentlist '/c "dir"' -PassThru -Wait
$output.ExitCode
```

A neat function I found [here](https://stackoverflow.com/a/52629677/15214802)

```shell
function Start-ProcessWithOutput
{
    param ([string]$Path,[string[]]$ArgumentList)
    $Output = New-Object -TypeName System.Text.StringBuilder
    $Error = New-Object -TypeName System.Text.StringBuilder
    $psi = New-object System.Diagnostics.ProcessStartInfo 
    $psi.CreateNoWindow = $true 
    $psi.UseShellExecute = $false 
    $psi.RedirectStandardOutput = $true 
    $psi.RedirectStandardError = $true 
    $psi.FileName = $Path
    if ($ArgumentList.Count -gt 0)
    {
        $psi.Arguments = $ArgumentList
    }
    $process = New-Object System.Diagnostics.Process 
    $process.StartInfo = $psi 
    [void]$process.Start()
    do
    {
       if (!$process.StandardOutput.EndOfStream)
       {
           [void]$Output.AppendLine($process.StandardOutput.ReadLine())
       }
       if (!$process.StandardError.EndOfStream)
       {
           [void]$Error.AppendLine($process.StandardError.ReadLine())
       }
       Start-Sleep -Milliseconds 10
    } while (!$process.HasExited)

    #read remainder
    while (!$process.StandardOutput.EndOfStream)
    {
        #write-verbose 'read remaining output'
        [void]$Output.AppendLine($process.StandardOutput.ReadLine())
    }
    while (!$process.StandardError.EndOfStream)
    {
        #write-verbose 'read remaining error'
        [void]$Error.AppendLine($process.StandardError.ReadLine())
    }

    return @{ExitCode = $process.ExitCode; Output = $Output.ToString(); Error = $Error.ToString(); ExitTime=$process.ExitTime}
}


$p = Start-ProcessWithOutput cmd.exe -argumentlist '/c "dir"'
$p.ExitCode
$p.Output
$p.Error
```

## Run Python as Admin

Running the whole python script in Admin, meaning that the subsequent processes
will have admin access, if this is the behaviour you prefer.

<script src="https://gist.github.com/leixingyu/a2cc64cec76638d2367cd4c2fc1b94ec.js"></script>


## References

[Stack Exchange - Run .exe file via Python as Administrator](https://superuser.com/questions/615654)

[Stack Overflow - How do I capture the output into a variable from an external process in PowerShell?](https://stackoverflow.com/questions/8097354/)

[Stack Overflow - Redirection of standard and error output appending to the same log file](https://stackoverflow.com/questions/8925323/)

[Stack Overflow - Powershell: Capturing standard out and error with Process object](https://stackoverflow.com/questions/11531068/)

[Windows Commandline - Windows runas command syntax and examples](https://www.windows-commandline.com/windows-runas-command-prompt/)


