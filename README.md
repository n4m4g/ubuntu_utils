Linux tips
==========

Contents
--------
- [nvidia](#nvidia)
- [ssh](#ssh)
- [git](#git)
- [vim](#vim)
- [terminal shortcut](#terminal-shortcut)
- [docker](#docker)
- [virtualenv](#virtualenv)
- [apt tools](#apt-tools)

nvidia
------
    
### Remove nvidia driver

    $ sudo apt purge nvidia*
    $ sudo apt purge libnvidia*
    $ dpkg -l | grep -i nvidia | awk '{print $2}' | xargs sudo dpkg -r
    $ sudo reboot

### cuda

if you are going to install using the deb installer (which is what was installing 387 driver etc.) then you could just do:

```
sudo apt-get install cuda-toolkit-X-Y
```

instead of

```
sudo apt-get install cuda
```

and that should skip the driver install.


### cudnn

    $ tar -xzvf cudnn-*.tgz
    $ sudo cp cuda/include/cudnn.h /usr/local/cuda/include
    $ sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
    $ sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
    
### ldconfig

    $ sudo ldconfig 
    /sbin/ldconfig.real: /usr/local/cuda/lib64/libcudnn.so.x is not a symbolic link

    $ cd /usr/local/cuda/lib64
    $ ls -lha libcudnn*

    $ sudo rm libcudnn.so
    $ sudo rm libcudnn.so.x
    $ sudo ln libcudnn.so.x.y.z libcudnn.so.x
    $ sudo ln libcudnn.so.x libcudnn.so
    
    $ sudo ldconfig

### ldconfig using python3 script

```
#!/usr/bin/python3
import os
from glob import glob
from subprocess import Popen, PIPE

def main():
    VERBOSE = True
    path = '/usr/local/cuda/lib64/libcudnn.so*'
    cudnn_files = sorted(glob(path))
    assert len(cudnn_files) == 3

    # c0: libcudnn.so
    # c1: libcudnn.so.x
    # c2: libcudnn.so.x.y.z
    c0, c1, c2 = cudnn_files
    
    cmd1 = f"sudo rm {c0}"
    cmd2 = f"sudo rm {c1}"
    cmd3 = f"sudo ln {c2} {c1}"
    cmd4 = f"sudo ln {c1} {c0}"
    cmd5 = f"sudo ldconfig"

    print("Start...")
    for cmd in [cmd1, cmd2, cmd3, cmd4]:
        p = Popen(cmd.split(" "), stdout=PIPE)
        out = p.communicate()
        if out[1] != None:
            print(out[1])
            assert False
        else:
            if VERBOSE:
                print(f"\t{cmd}")

    # check no warning from ldconfig
    p = Popen(cmd5.split(" "), stdout=PIPE)
    out = p.communicate()
    assert out[0] == b''

    print("Done...")

if __name__ == "__main__":
    main()
```

Example output
```
Start...                                                                                                  
        sudo rm /usr/local/cuda/lib64/libcudnn.so                                                         
        sudo rm /usr/local/cuda/lib64/libcudnn.so.7                                                       
        sudo ln /usr/local/cuda/lib64/libcudnn.so.7.6.5 /usr/local/cuda/lib64/libcudnn.so.7               
        sudo ln /usr/local/cuda/lib64/libcudnn.so.7 /usr/local/cuda/lib64/libcudnn.so                     
Done...
```

### GPU fan speed control

    $ sudo nvidia-xconfig
    $ sudo nvidia-xconfig --cool-bits=28
    
    Restart the computer and search for NVIDIA X Server Settings in the Dash. 
    There should be an option to change fan speed under Thermal Settings.

ssh
---

### Generate ssh key

    $ ssh-keygen -t rsa -C <email> -b 4096
    
### Set ssh alive

Add follow command to ~/.ssh/config

    Host *
    ServerAliveInterval 300
    
and

    $ chmod 600 ~/.ssh/config

### Set ssh alias

Add following commands to ~/.ssh/config

    # for proxy
    Host <alias proxy name>
        Hostname <proxy ip>
        User <uname>
        ForwardX11 yes
        ForwardX11Trusted yes
        
    # for server
    Host <alias server name>
        Hostname <server name>
        User <uname>
        ProxyCommand ssh <proxy name> -W %h:%p
        ProxyJump <proxy name>

### ssh without password

    $ ssh-copy-id <host name>

git
---

### command alias

    git config --global user.name "<name>"
    git config --global user.email <email>
    git config --global core.editor vim
    git config --global alias.ci commit
    git config --global alias.st status
    git config --global alias.co checkout
    git config --global alias.br branch
    git config --list # show git config
 
### show tracked files

    git ls-tree -r master --name-only

vim
---

### Setting

Add follow command to ~/.vimrc

    set tabstop=4
    set shiftwidth=4
    set nu
    set hlsearch
    set smartindent
    set scrolloff=5
    "set expandtab

terminal shortcut
-----------------

- ctrl + U: Clear **whole** command line.
- ctrl + W: Clear words to the **left** of the cursor
- ctrl + K: Clear words to the **right** of the cursor
- ctrl + L: Clear words **above** the cursor.

docker
------

### Show docker images

    $ docker images

### Show all containers

    $ docker ps -a
    
### Add user to docker group

    $ sudo usermod -aG docker $USER

### Create container

    $ docker run -it [image name] [shell]
    $ docker run -it ubuntu bash

### Enter the container

    $ docker exec -it <container id> bash
    $ docker exec -it c357 bash

virtualenv
----------

### Install virtualenv

    $ pip3 install --user virtualenv
    
### Create a virtualenv

    $ python3 -m virtualenv -p python3 [name]

apt tools
---------

    $ sudo apt install ncdu # NCurses Disk Usage
    $ sudo apt install pragha # support m4a format
    # sudo apt install blueman # gui bluetooth manager
