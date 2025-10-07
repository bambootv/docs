
1. Google Chrome
```
wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
echo 'deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main' | sudo tee /etc/apt/sources.list.d/google-chrome.list
sudo apt update 
sudo apt install google-chrome-stable
```

2. [Zsh](https://viblo.asia/p/cai-oh-my-zsh-powerlevel10k-toi-uu-va-su-dung-phim-tat-cho-terminal-ORNZqowM50n#_4-tim-hieu-zsh-8)
```
sudo apt install zsh
chsh -s $(which zsh)
logout

cd
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
  ![Screenshot from 2023-05-06 15-57-02](https://user-images.githubusercontent.com/18675907/236614202-4d356ae2-20d8-40b6-865a-a697e0e3e697.png)

  [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions#installation)
 
```
nano ~/.zshrc
plugins=(git zsh-autosuggestions)

export PATH=/opt/homebrew/bin:$PATH

ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE="fg=#5c6370"

prompt_dir() {
	prompt_segment blue $CURRENT_FG '%c'
}

build_prompt () {
	RETVAL=$?
	prompt_status
	prompt_virtualenv
	prompt_aws
#	prompt_context
	prompt_dir
	prompt_git
	prompt_bzr
	prompt_hg
	prompt_end
}

```

3. Ibus
```
sudo apt-get install ibus-unikey
ibus-daemon -d
ibus restart
```


4. docker
```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu `lsb_release -cs` test"
sudo apt update
sudo apt install docker-ce

or shoter

curl -fsSL https://get.docker.com/ | sh
```

```
mkdir /etc/docker
sudo nano /etc/docker/daemon.json
{
  "log-driver": "json-file", # none
  "log-opts": {
    "max-size": "1m",
    "max-file": "3"
  }
}
sudo service docker restart
sudo systemctl stop docker.socket // If can not restart and restart again
```

[Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall)
```
sudo groupadd docker
sudo usermod -aG docker $USER
logout
newgrp docker
sudo chmod 666 /var/run/docker.sock
reboot

or shorter
sudo usermod -aG docker $(whoami)
reboot
```


5. nvm
```
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash
source ~/.szhrc
nvm install 16.18.0
nvm use 16.18.0
nvm alias default 16.18.0
```

6. Visual Studio Code
```
sudo apt update && sudo apt upgrade -y
sudo apt install software-properties-common apt-transport-https wget
wget -q https://packages.microsoft.com/keys/microsoft.asc -O- | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://packages.microsoft.com/repos/vscode stable main"
sudo apt install code
```

7. Postman
```
sudo snap install postman
```

8. Adminer
```
sudo apt install adminer
sudo a2enconf adminer
systemctl reload apache2
http://localhost/adminer
sudo nano /etc/adminer/adminer.css
```

9. Mongodb Compass
```
wget https://downloads.mongodb.com/compass/mongodb-compass_1.40.4_amd64.deb
sudo dpkg -i mongodb-compass_1.40.4_amd64.deb
```

10. alia ssh
```
cat ~/.ssh/config 
Host name
    HostName abc.abc.abc.abc
    User root
    Port 22
```

11. zip
```
zip -r images.zip images/
```

12. Prevent adult conent

[CleanBrowsing](https://cleanbrowsing.org/help/docs/configure-dns-filtering-on-linux/)
```
sudo nano /etc/resolv.conf
nameserver 185.228.168.168
sudo chattr +i /etc/resolv.conf
```

13. IP
```
ip route
... 192.168.xxx.xxx 
# Can use it to DNS (CloudFlare, ...) but is is flexiable
# Can buy static ip from provider
```

14. Redis
```
redis-cli
> AUTH YourPasswordHere
```

15. Error mounting
```
sudo ntfsfix /dev/nvme0n1p7
```

16. Monitoring
```
sudo ss -tulpn | grep :80
sudo kill -QUIT 882397
du -h / -ah
```
