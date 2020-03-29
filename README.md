# dotfiles

## Current status
By now this version is for *nix enviroments. The primary target is a remote-container in VS Code.
A variant for using some/most of the features with git-bash-for-windows is thinkable but not yet ready.  
Main problems:
- bash-it is way to slow when used full featured due to the Shell-Wrapper-Stuff. (it takes very long to get the prompt back when in git repository files - even if the most slowing down features are already turned off)
- dotbot requires python. This is considered a too-big-requirement

Based on https://www.anishathalye.com/2014/08/03/managing-your-dotfiles/


## Use with .devcontainer

### Prerequisites
The base image are described here: https://hub.docker.com/_/microsoft-vscode-devcontainers

These images may have to be adapted a little bit.
- ssh  
  ```echo "StrictHostKeyChecking no" >> /root/.ssh/config```  
  Otherwise checking out a dotfile repository on github/bitbucket would fail
- git  
  ```git config --global autocrlf true```  
  DevContainer would use the  .gitconfig settings of the host if there is no .gitconfig there. This may cause trouble on Windows hosts (core.autocrlf setting for example). 

*Note*: by now these changes are handled by the Dockerfile in .devcontainer.

### Accessing private repositories

The preferred access method to private repository is via ssh. This is very nice once the "right" ssh-agent is set up feed with keys.
See VSCode documentation at https://code.visualstudio.com/docs/remote/troubleshooting#_setting-up-the-ssh-agent

### .gitconfig

The main `.gitconfig` is located in this repository. This file contains the common git settings used in every system. Local settings are stored in `.gitconfig.local`. That file is not part of this repository by design. 

### .bashrc

There is standard .bashrc provided by the container. This is replaced by a link to bashrc.bash.