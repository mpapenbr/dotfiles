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
  DevContainer would use the  .gitconfig settings of the host when checking out. 

