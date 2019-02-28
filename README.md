# roger-skyline-1.5


## Install Ubuntu updates
```
sudo apt-get update           # Fetches the list of available updates
sudo apt-get upgrade -y       # Strictly upgrades the current packages
sudo apt-get dist-upgrade -y  # Installs updates (new ones)
sudo apt-get autoremove -y    # removes packages that were automatically installed to satisfy dependencies for some package and that are nomore needed.
sudo apt-get autoclean -y     # clears out the local repository of retrieved package files. The difference is that it only removes package files that can no longer be downloaded, and are largely useless. This allows a cache to be maintained over a long period without it growing out of control.
sudo reboot                   # if needed
    -y, --yes, --assume-yes
               Automatic yes to prompts;
```
