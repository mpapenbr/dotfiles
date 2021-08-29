# Devcontainer

## ssh agent

**Use case:** I want my devcontainers to have ssh access to selected targets out of the box. No configuration inside the container should be necessary.

**Solution:** ssh-agent forwarding

**Implementation:**
In my WSL I already have configured my ssh keys within the *~/.ssh* directory. Using this in the WSL works fine.  
Forwarding those keys to the dev container was the hard part even though in theory it is just simple. But there some traps regarding the "right" place and time when/how to setup the ssh-agent.

The crucial part is to setup the system in way that the *SSH_AUTH_SOCK* env var is passed to the devcontainer. As a bonus target I want only one ssh-agent to be started. 


- WSL configuration
  -  `sudo apt-get install socat`  
     It turned out that this tool is needed by VS Code to access the SSH_AUTH_SOCK unix socket from the devcontainer

  - setup ssh-agent in _.bashrc_ (put it to the very beginning, see note below). 
    ```
    ssh-add -l &>/dev/null
    if [ "$?" == 2 ]; then
        test -r ~/.ssh-agent && \
            eval "$(<~/.ssh-agent)" >/dev/null

        ssh-add -l &>/dev/null
        if [ "$?" == 2 ]; then
            (umask 066; ssh-agent > ~/.ssh-agent)
            eval "$(<~/.ssh-agent)" >/dev/null
            ssh-add
        fi
    fi
    ```

    The default _.bashrc_ contains a section for early leave. For non-interactive sessions the processing ends here. In order to get the ssh-agent started in any case we have to take care about that before this point.
    ```
    case $- in
        *i*) ;;
        *) return;;
    esac
    ```
  
- VS Code configuration  
  There are two important settings: 
  - remote.SSH.enableAgentForwarding = true
  - terminal.integrated.inheritEnv = true


**How to check:** Inside the devcontainer enter `ssh-add -l`. 
This lists the fingerprints of the current keys managed by the ssh-agent. It should produce an output like this:
```
node@b4b3bb0688e8:/workspaces/tmp$ ssh-add -l
2048 SHA256:zbJVPXmeofchakaoUZxg4lF/N7dGu7h0/GL4F4zc5yQ /home/mpapenbr/.ssh/git/id_rsa_git_markus (RSA)
```
A more hands-on test is to access github.com by `ssh -T git@github.com`. This should produce this
```
Hi mpapenbr! You've successfully authenticated, but GitHub does not provide shell access.
```


If no *SSH_AUTH_SOCK* var is set you will see something like this:  
```
Could not open a connection to your authentication agent
```

**Technic**

SSH_AUTH_SOCK=/tmp/vscode-ssh-auth-f7fc8024a3c4afb00042f711d2f553d78f4a30a0.sock


First few lines from DevContainer-Log (in VS-Code: F1 -> Remote Containers: Show Container Log)
```
[348 ms] Remote-Containers 0.191.1 in VS Code 1.59.1 (3866c3553be8b268c8a7f8c0482c0c0177aa8bfa).
[348 ms] Start: Run: wsl -d Ubuntu-20.04 -e wslpath -u \\wsl$\Ubuntu-20.04\home\mpapenbr\tmp
[461 ms] Start: Resolving Remote
[464 ms] Start: Run: wsl -d Ubuntu-20.04 -e wslpath -u \\wsl$\Ubuntu-20.04\home\mpapenbr\tmp
[570 ms] Start: Run: wsl -d Ubuntu-20.04
[677 ms] Setting up container for folder or workspace: /home/mpapenbr/tmp
[679 ms] Start: Check Docker is running
[679 ms] Start: Run: wsl -d Ubuntu-20.04 -e /bin/sh -c cd '/home/mpapenbr/tmp' && DISPLAY='1' ELECTRON_RUN_AS_NODE='1' SSH_ASKPASS='c:\Users\mpapenbr\.vscode\extensions\ms-vscode-remote.remote-containers-0.191.1\scripts\ssh-askpass.bat' VSCODE_SSH_ASKPASS_NODE='C:\Users\mpapenbr\AppData\Local\Programs\Microsoft VS Code\Code.exe' VSCODE_SSH_ASKPASS_MAIN='c:\Users\mpapenbr\.vscode\extensions\ms-vscode-remote.remote-containers-0.191.1\dist\common\sshAskpass.js' VSCODE_SSH_ASKPASS_HANDLE='\\.\pipe\ssh-askpass-32d8f4d1651251025338771f195d4815e4e945f9-sock' DOCKER_CONTEXT='default' SSH_AUTH_SOCK='/tmp/ssh-xFCJWwgBDvfa/agent.16472' VSCODE_SSH_ASKPASS_COUNTER='1' docker 'version' '--format' '{{.Server.APIVersion}}'
[833 ms] Server API version: 1.41
```
When VS Code connects to the container it uses a non-interactive shell. This is why the setup of the ssh-agent in _.bashrc_ has to be done prior to the early-exit.

References:
- https://github.com/microsoft/vscode-remote-release/issues/3902
- https://rabexc.org/posts/pitfalls-of-ssh-agents (ssh-agent startup script)





