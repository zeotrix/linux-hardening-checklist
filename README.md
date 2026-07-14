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
      
- [ ] 
- [ ] Download [v2ray](https://github.com/2dust/v2rayN/releases) and config it: start at boot,etc

      ```
      sudo apt update
      sudo dpkg -i v2ray.deb
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
  ```
- [ ] htop<br/>
  
  ```
  sudo apt install htop
  ```
- [ ] Install Telegram:<br/>

      ```
      sudo snap install telegram-desktop
      ```
      
- [ ] install vlc if not installed:<br/>

      ```
      sudo apt update
      sudo apt install vlc
      ```
      
- [ ] 
- [ ] 
- [ ] persian fonts
- [ ] 
- [ ] install [sublime](https://www.sublimetext.com/docs/linux_repositories.html)<br/>

      ```
      wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo tee /etc/apt/keyrings/sublimehq-pub.asc > /dev/null
      echo -e 'Types: deb\nURIs: https://download.sublimetext.com/\nSuites: apt/stable/\nSigned-By: /etc/apt/keyrings/sublimehq-pub.asc' | sudo tee /etc/apt/sources.list.d/sublime-text.sources
      sudo apt-get update
      sudo apt-get install sublime-text
      ```
      
- [ ] 
- [ ] istall wireshark
- [ ] install Burp <br/>      

      ```
      add `alias burp='java -jar $HOME/Documents/Burp.Suite.Professional.2025.6.5/BurpLoaderKeygen117.jar'` to `~/.bashrc`
      ```
      
      

      
- [ ] install go and set path for it
- [ ] pyenv
- [ ] docker
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
      
