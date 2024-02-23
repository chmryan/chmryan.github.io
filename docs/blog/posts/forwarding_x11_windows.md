---
title: X11 Forwarding with VS Code in Windows
date:
  created: 2023-10-14
  updated: 2024-02-23
# authors:
#   - chrisryan
categories:
- Software Development
tags:
- ssh
- remote-development
- vscode
- x11
- windows
slug: forwarding-x11-windows
---

# Problem
How to forward X11 from a remote server to a local Windows machine using VS Code.

# How I solved it

## Systems

- Remote: Ubuntu Server LTS 22.04.3 (headless)
- Local: Windows 10 with VS Code installed

## Steps

1.  Install VS Code [Remote-SSH](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh) extension
    
2.  Install Xming and start Xming (Default Display:0.0)
    
3.  Add a new environment variable to Windows with the scope *User*, e.g. with PowerShell: `[System.Environment]::SetEnvironmentVariable('DISPLAY','localhost:0.0', 'User')`
    
4.  Log out or restart to properly set the environment variable outside of the current PowerShell session (simply entering the cmd `refreshenv` does not seem to work properly).
    
5.  If not done yet, enable remote login with a public/private key pair:
    
    1.  Create a key pair: `ssh-keygen -t rsa -b 4096`
    2.  Upload the key to the server: `ssh-copy-id -i ~/.ssh/key_name.pub -p 22 user_name@remote_host`
    3.  Create an authorized keys file: `cat ~/.ssh/key_name.pub | ssh -p 22 user_name@remote_host “mkdir -p .ssh; cat >> ~/.ssh/authorized_keys”`
    4.  Authenticate: `ssh -p 22 -i ~/.ssh/key_name user_name@remote_host`
6.  On the **server**, edit `/etc/ssh/sshd_config`:
    
    ```bash
    AllowAgentForwarding yes
    AllowTcpForwarding yes
    X11Forwarding yes
    X11DisplayOffset 10
    X11UseLocalhost no
    ```
    
7.  On the **server**, restart the sshd daemon and exit:
    
    ```
    sudo service sshd restart && exit
    ```
    
8.  In VS Code: `F1` -> `Remote SSH: Open SSH Configuration File`
    
    , add the following content:
    
    ```bash
    Host your_remote_host
      HostName your_remote_host
      User user_name
      Port 22
      IdentityFile /c:/Users/user_name/.ssh/id_rsa
      ForwardAgent yes
      ForwardX11 yes
      ForwardX11Trusted yes
    ```
    
9.  Connect to the remote server in VS Code, check the output of the following cmd: `bash -c "echo DISPLAY=$DISPLAY`. The output should be `your_remote_host=dev-vm:10.0`.
    
10. Install the `xclock` package for testing: `sudo apt upgrade && sudo apt install x11-apps`
    
11. In the remote console, enter `xclock` and press enter. When the Xming Server is running, a popup window should appear after a few seconds with a clock.


**Sources**

- <https://code.visualstudio.com/docs/remote/ssh>
- <https://stackoverflow.com/questions/65468655/vs-code-remote-x11-cant-get-display-while-connecting-to-remote-server>
- <https://stackoverflow.com/questions/66160457/vscode-remote-ssh-cannot-identify-private-key-file>
- <https://stackoverflow.com/questions/19589844/set-up-x11-forwarding-over-ssh#23033038>
- <https://blog.radwebhosting.com/how-to-setup-ssh-login-with-public-key-authentication/>