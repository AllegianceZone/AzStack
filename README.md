1. Swap:
```
  sudo install -o root -g root -m 0600 /dev/null /swapfile
  sudo dd if=/dev/zero of=/swapfile bs=1k count=1024k
  sudo mkswap /swapfile
  sudo swapon /swapfile
  echo "/swapfile       swap    swap    auto      0       0" | sudo tee -a /etc/fstab
  sudo sysctl -w vm.swappiness=10
  echo vm.swappiness = 10 | sudo tee -a /etc/sysctl.conf
```
2. Discourse:
```
  sudo wget -qO- https://get.docker.com/ | sh
  sudo usermod -aG docker Imago
  sudo mkdir /var/discourse
  sudo git clone https://github.com/discourse/discourse_docker.git /var/discourse
  cd /var/discourse
  sudo cp samples/standalone.yml containers/app.yml
  sudo nano containers/app.yml
    see app.yml in this repo
  sudo ./launcher bootstrap app
  sudo ./launcher start app
    login as admin and enable restore, restore latest bacukup in this repo
    reboot
```
3. Trac:
```
  cd /var/discourse
  sudo ./launcher enter app
  sudo apt-get install python-dev
  wget https://bootstrap.pypa.io/ez_setup.py -O - | sudo python
  easy_install psycopg2
  easy_install Trac
  sudo -u postgres createdb trac
  sudo trac-admin /opt/trac initenv
    use connection string:
      postgres://tracuser:allegdb@/trac
```
AZ CGI:
```
  WIP
```

