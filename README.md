## For personal os:

- [ ] Download the latest [Ubuntu Desktop](https://ubuntu.com/download/desktop)
- [ ] Create bootable:<br/>
      in Windows use [rufus](https://rufus.ie/en/#download)<br/>
      in Linux use dd:<br/>
      
      ```
      1- lsblk >> recognize the disk: /dev/sdx
      2- sudo umount /dev/sdx
      3- sudo dd if=/path/to/ubunto.iso of=/dev/sdx bs=16M      (bs=BlockSize use 4 or 8 or 16. i used 16)     
      ```
- [ ] i use this partitioning:<br/>

      ```
      /boot/efi      512Mb
      /              80Gb
      /home          400Gb   
      ```

- [ ] First Configs:<br/>

      ```
      sudo apt update
      sudo apt upgrade -y
      sudo apt autoremove
      sudo add-apt-repository universe
      sudo apt install -y build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev curl llvm libncurses5-dev libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev make jq tree htop unzip libffi-dev liblzma-dev traceroute vim 
      ```
      
- [ ] GNOME Tweaks:<br/>

      ```
      sudo apt install gnome-tweaks
      sudo apt install gnome-shell-extensions
      ```
- [ ] google chrome:<br/>

      ```
      Download from https://www.google.com/chrome/
      sudo dpkg -i google-chrome-stable_current_amd64.deb
      ```
      
- [ ] HomeBrew:<br/>

      ```
      https://brew.sh/
      ```
      
- [ ] Download [v2ray](https://github.com/2dust/v2rayN/releases) and config it: start at boot,etc

      ```
      sudo apt update
      sudo dpkg -i v2ray.deb
      cd ~/.local/share/v2rayN/bin/xray
      wget https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geosite.dat
      wget https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geoip.dat
      ```
- [ ] install Snap:<br/>

      ```
      sudo apt update
      sudo apt install snapd
      sudo snap install core
      sudo snap refresh
      ```

- [ ] Git<br/>
  
  ```
  sudo apt install git
  git --version
  git config --global user.name "username"
  git config --global user.email "your_email@example.com"
  git config --global --list
  git config --global init.defaultBranch master
  git config --global color.ui auto
  git config --global pull.rebase false
  git config --global core.editor "vim"
  ```
- [ ] Install Telegram:<br/>

      ```
      sudo snap install telegram-desktop
      ```
      
- [ ] install vlc if not installed:<br/>

      ```
      sudo apt update
      sudo apt install vlc ubuntu-restricted-extras ffmpeg yt-dlp
      ```
      
- [ ] Install Java:<br/>

      ```
      sudo apt install openjdk-26-jdk -y
      java -version
      javac -version
      ```
      
- [ ] VSCode:<br/>

      ```
      download it from: (https://code.visualstudio.com/docs/setup/linux)
      
      
      ```
            
- [ ] Android Studio:<br/>

      ```

      ```
- [ ] Flutter stable:<br/>

      ```

      ```
            
- [ ] persian fonts
- [ ] 
- [ ] install [sublime](https://www.sublimetext.com/docs/linux_repositories.html)<br/>

      ```
      wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo tee /etc/apt/keyrings/sublimehq-pub.asc > /dev/null
      echo -e 'Types: deb\nURIs: https://download.sublimetext.com/\nSuites: apt/stable/\nSigned-By: /etc/apt/keyrings/sublimehq-pub.asc' | sudo tee /etc/apt/sources.list.d/sublime-text.sources
      sudo apt-get update
      sudo apt-get install sublime-text
      ```
      
- [ ] Node:<br/>

      ```
      sudo apt install -y curl build-essential
      curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
      source ~/.bashrc
      nvm --version
      nvm install --lts
      node -v
      npm -v
      nvm alias default lts/*
      npm install -g npm@latest
      npm install -g yarn pnpm
      yarn -v
      pnpm -v
      ```
      
- [ ] istall wireshark
- [ ] install Burp <br/>      

      ```
      add `alias burp='java -jar $HOME/Documents/Burp.Suite.Professional.2025.6.5/BurpLoaderKeygen117.jar'` to `~/.bashrc`
      ```
      
      

      
- [ ] install go and set path for it
- [ ] pyenv:<br/>

      ```
      curl -fsSL https://pyenv.run | bash
      cd ~/.pyenv && src/configure && make -C src
      ```
      
- [ ] Docker:<br/>

      ```
      sudo apt install -y ca-certificates curl gnupg
      sudo install -m 0755 -d /etc/apt/keyrings
      curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
      sudo chmod a+r /etc/apt/keyrings/docker.gpg
      echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
      sudo apt update
      sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
      docker --version
      sudo usermod -aG docker $USER
      logging out and back in.
      ```
      
- [ ] discord
- [ ] xmind
- [ ] pycharm
- [ ] phpstorm
- [ ] cline
- [ ] crunch
- [ ] ffuf
- [ ] jwt-tool
- [ ] jython
- [ ] sqlmap
- [ ] tplmap
- [ ] 
      
