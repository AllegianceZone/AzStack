Backups:
```
  Trac: 
    sudo trac-admin /opt/trac hotcopy /tmp/trachotcopy
    sudo tar -cvzf ~/trachotcopy.tar.gz /tmp/trachotcopy
  Discourse: 
    login as admin to website and use the backup gui
  AZ CGI:
```

Swap:
```
  sudo install -o root -g root -m 0600 /dev/null /swapfile
  sudo dd if=/dev/zero of=/swapfile bs=1k count=1024k
  sudo mkswap /swapfile
  sudo swapon /swapfile
  echo "/swapfile       swap    swap    auto      0       0" | sudo tee -a /etc/fstab
  sudo sysctl -w vm.swappiness=10
  echo vm.swappiness = 10 | sudo tee -a /etc/sysctl.conf
```
Discourse:
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
    login to website as admin and enable restore, restore latest bacukup in this repo via gui
    reboot
```
Trac:
```
  cd /var/discourse
  sudo ./launcher enter app
  sudo apt-get install python-dev
  wget https://bootstrap.pypa.io/ez_setup.py -O - | sudo python
  easy_install psycopg2
  easy_install Trac
  sudo -u postgres createdb trac
  sudo -u postgres psql template1 -c "CREATE USER tracuser WITH PASSWORD 'allegdb'; GRANT ALL PRIVILEGES ON DATABASE trac to tracuser;" 
  sudo pico /etc/postgresql/9.3/main/pg_hba.conf
    change:
      local all   all   peer
    to:
      local all   all   trust
  sudo sv restart postgres
  sudo trac-admin /opt/trac initenv
    use connection string:
      postgres://tracuser:allegdb@/trac
  sudo -u postgres psql trac -f trac-notify.sql
    get trac-notify.sql from this repo
  cd /etc/service
  sudo mkdir tracd
  cd tracd
  sudo nano run
    #!/bin/sh
    exec 2>&1
    OAUTHLIB_INSECURE_TRANSPORT=1 PYTHON_EGG_CACHE=/var/www/.python-eggs exec chpst -u www-data -U www-data /usr/local/bin/tracd -s -auth="*,/opt/trac/.htpasswd,/opt/trac" --port 8000 /opt/trac
  chmod +x run
  cd /opt/trac
  sudo nano .htpasswd
    Imago:$apr1$n4fw6CCM$qTdtelsxSqA4japDs5U6m0
    builder:$apr1$.SjXPbsw$lDmxsS/983hPrNst1SqkL/
  chown www-data:www-data .htpasswd
  cd /var/www
  mkdir .python-eggs
  chown www-data:www-data .python-eggs
  
```
AZ CGI:
```
  WIP
```

AllegZoneBot:
```

```

