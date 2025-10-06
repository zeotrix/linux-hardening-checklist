## For personal os:

### 1.Install UBUNTU
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
      /
      ```
- [ ] Download [v2ray](https://github.com/2dust/v2rayN/releases) and config it: start at boot,etc
      
      ```
      sudo apt update
      sudo dpkg -i v2ray.deb
      ```
- [ ] install snap:
      ```
      sudo apt update
      sudo apt install snapd
      ```
- [ ] 
      
