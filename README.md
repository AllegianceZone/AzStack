Backups:
```
  Trac: 
    sudo tar -czvf /home/imago/dist-packages.tar.gz /usr/local/lib/python2.7/dist-packages
    sudo -u postgres pg_dump trac > /home/imago/trac.sql
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
  sudo -u postgres dropdb trac
  sudo -u postgres createdb trac
  sudo -u postgres psql trac -f trac.sql
    get trac.sql from backup
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
  tar -xzf /home/imago/dist-packages.tar.gz /usr/local/lib/python2.7
  put trac.conf in /etc/nginx/conf.d
  sv start tracd
```
AZ CGI:
```
sudo /etc/nginx/conf.d/cgi.conf


server {
root  /var/www;
        server_name azforum.cloudapp.net ;
    listen              80;
    listen              443 ssl;
    ssl_certificate     azforum.cloudapp.net.crt;
    ssl_certificate_key azforum.cloudapp.net.key;

  location / {
      index  index.html index.cgi;
  }

  location ~ \.pl|cgi$ {
      try_files $uri =404;
      gzip off;
      fastcgi_pass  127.0.0.1:8999;
      fastcgi_index index.cgi;
      fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
      include fastcgi_params;
      }
}

sudo nano /etc/nginx/azforum.cloudapp.net.crt
-----BEGIN CERTIFICATE-----
MIICaTCCAdICCQDf3L2xBJW/+TANBgkqhkiG9w0BAQsFADB5MQswCQYDVQQGEwJV
UzETMBEGA1UECAwKV2FzaGluZ3RvbjEQMA4GA1UEBwwHUmVkbW9uZDEkMCIGA1UE
CgwbQWxsZWdpYW5jZSBab25lIERldmVsb3BtZW50MR0wGwYDVQQDDBRhemZvcnVt
LmNsb3VkYXBwLm5ldDAeFw0xNDA5MjQxNjAwMTZaFw0xNzA5MjMxNjAwMTZaMHkx
CzAJBgNVBAYTAlVTMRMwEQYDVQQIDApXYXNoaW5ndG9uMRAwDgYDVQQHDAdSZWRt
b25kMSQwIgYDVQQKDBtBbGxlZ2lhbmNlIFpvbmUgRGV2ZWxvcG1lbnQxHTAbBgNV
BAMMFGF6Zm9ydW0uY2xvdWRhcHAubmV0MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCB
iQKBgQCzOgStxMYXN3gNSMNTwa0tZWyTUgKOCDDTx9h1LVxvqMwQITVyhw4xpPWL
I3XSxBBqRMz/Sv8LucCmyS5zp6mMn4sxddmSudAYQNsFJmmqp0E9uuNI+L2rzG39
xER5QvB7deDtBc32anzGbCZAprR74IcRtubCMMtBpS/5El1wewIDAQABMA0GCSqG
SIb3DQEBCwUAA4GBAEf/qlf7SikbNWIcTSxAh1kmaSUKpGlRDx7BuBfeFTiWqWDs
EB+Lf3QOw+Fj1IgDptL8e12PI8Jwrd2qwigYpsWcGk+rcU02aRVZSyjahMQbdOXc
KO0R4As6fMlwK3/AdFYTs7CC9NoJR7UG9meautMXMwt/L6vD15Vt2aLBgZum
-----END CERTIFICATE-----

sudo nano /etc/nginx/azforum.cloudapp.net.csr
-----BEGIN CERTIFICATE REQUEST-----
MIIBuTCCASICAQAweTELMAkGA1UEBhMCVVMxEzARBgNVBAgMCldhc2hpbmd0b24x
EDAOBgNVBAcMB1JlZG1vbmQxJDAiBgNVBAoMG0FsbGVnaWFuY2UgWm9uZSBEZXZl
bG9wbWVudDEdMBsGA1UEAwwUYXpmb3J1bS5jbG91ZGFwcC5uZXQwgZ8wDQYJKoZI
hvcNAQEBBQADgY0AMIGJAoGBALM6BK3Exhc3eA1Iw1PBrS1lbJNSAo4IMNPH2HUt
XG+ozBAhNXKHDjGk9YsjddLEEGpEzP9K/wu5wKbJLnOnqYyfizF12ZK50BhA2wUm
aaqnQT2640j4vavMbf3ERHlC8Ht14O0FzfZqfMZsJkCmtHvghxG25sIwy0GlL/kS
XXB7AgMBAAGgADANBgkqhkiG9w0BAQsFAAOBgQCtOlw/TRsBoRwbty2jmYluYe0Z
okCS/uGeY2iGq6q0MZc471Yej0JEeuhO8jM4kCyM2M39Odv54g1eod6LoGqh3w98
2SeGJD5mDUERF4QqIlUjT/zUh6e+cFjSzYTtzHS5bS8+YnIfqS+Wf5tbYAcgP3yj
5mb1r+JPyknbZTD2GQ==
-----END CERTIFICATE REQUEST-----

sudo nano /etc/nginx/azforum.cloudapp.net.key

-----BEGIN RSA PRIVATE KEY-----
MIICXgIBAAKBgQCzOgStxMYXN3gNSMNTwa0tZWyTUgKOCDDTx9h1LVxvqMwQITVy
hw4xpPWLI3XSxBBqRMz/Sv8LucCmyS5zp6mMn4sxddmSudAYQNsFJmmqp0E9uuNI
+L2rzG39xER5QvB7deDtBc32anzGbCZAprR74IcRtubCMMtBpS/5El1wewIDAQAB
AoGAMxOkw7ThUzp+nyKOb+8xIE/YSn/DtKCG8cPxXyuHPVcYmLwuFC6DEAjX5Ug8
ys0PdImY9mR0HO5aBe4tq84rViqKhKOtYDsIQaP0iC0/CEW5ldtiMJxNJ4mjkyax
KpN9q4x2Emnr0nm+ILeMs5IISYNExJzO1k12BNBV/jsEZOECQQDqK2BpimMtZQIj
9r84zomEOEAkbHDKFaa3LilYYkr9FRo3ooAR1qi+nHmsxDjLE9IxlG8u+W7IZ/Qj
MaRKkH8rAkEAw+9kS+T8zPvf7WjJpw4DCk9oWYKJTqMkeWOQ5giDxCkqMftHN/Kx
SK+tA59y4irqMXWJFVw9KF+LGc+/gCyr8QJBAMHoa2zZ8Kh9dRlM0Sn8NXmsjsja
l0dwSF51tjz/H/OUjuI1CPI8m/1DW6pJznGzlyIBNVRjrm37Tvn5uH8aSoMCQQCn
VfV/h9KLsnVdSggmnyXjkUvaXkysF1LYRTuD6jO2vT1nMGZzltbn7/lARdYU6HfY
w7vOvVcR+v9OjQnNCtnRAkEAmsU2fniVPkd9Q2OYB9ewLrR8U+5iuO2nIF1qyDRP
IQL47yI+ABP5RTZDVjNcvbrP3O1gLu+V6tC0KPsg5QIuWA==
-----END RSA PRIVATE KEY-----

cd /var/www
mkdir inputmaps
chown www-data:www-data inputmaps

sudo nano /usr/local/bin/fastcgi.pl

#!/usr/bin/perl

use FCGI;
use Socket;
use POSIX qw(setsid);

require 'syscall.ph';

#&daemonize;

#this keeps the program alive or something after exec'ing perl scripts
END() { } BEGIN() { }
*CORE::GLOBAL::exit = sub { die "fakeexit\nrc=".shift()."\n"; };
eval q{exit};
if ($@) {
        exit unless $@ =~ /^fakeexit/;
};

&main;

sub daemonize() {
    chdir '/'                 or die "Can't chdir to /: $!";
    defined(my $pid = fork)   or die "Can't fork: $!";
    exit if $pid;
    setsid                    or die "Can't start a new session: $!";
    umask 0;
}

sub main {
        $socket = FCGI::OpenSocket( "127.0.0.1:8999", 10 ); #use IP sockets
        $request = FCGI::Request( \*STDIN, \*STDOUT, \*STDERR, \%req_params, $socket );
        if ($request) { request_loop()};
            FCGI::CloseSocket( $socket );
}

sub request_loop {
        while( $request->Accept() >= 0 ) {

           #processing any STDIN input from WebServer (for CGI-POST actions)
           $stdin_passthrough ='';
           $req_len = 0 + $req_params{'CONTENT_LENGTH'};
           if (($req_params{'REQUEST_METHOD'} eq 'POST') && ($req_len != 0) ){
                my $bytes_read = 0;
                while ($bytes_read < $req_len) {
                        my $data = '';
                        my $bytes = read(STDIN, $data, ($req_len - $bytes_read));
                        last if ($bytes == 0 || !defined($bytes));
                        $stdin_passthrough .= $data;
                        $bytes_read += $bytes;
                }
            }

            #running the cgi app
            if ( (-x $req_params{SCRIPT_FILENAME}) &&  #can I execute this?
                 (-s $req_params{SCRIPT_FILENAME}) &&  #Is this file empty?
                 (-r $req_params{SCRIPT_FILENAME})     #can I read this file?
            ){
                pipe(CHILD_RD, PARENT_WR);
                my $pid = open(KID_TO_READ, "-|");
                unless(defined($pid)) {
                        print("Content-type: text/plain\r\n\r\n");
                        print "Error: CGI app returned no output - ";
                        print "Executing $req_params{SCRIPT_FILENAME} failed !\n";
                        next;
                }
                if ($pid > 0) {
                        close(CHILD_RD);
                        print PARENT_WR $stdin_passthrough;
                        close(PARENT_WR);

                        while(my $s = <KID_TO_READ>) { print $s; }
                        close KID_TO_READ;
                        waitpid($pid, 0);
                } else {
                        foreach $key ( keys %req_params){
                           $ENV{$key} = $req_params{$key};
                        }
                        # cd to the script's local directory
                        if ($req_params{SCRIPT_FILENAME} =~ /^(.*)\/[^\/]+$/) {
                                chdir $1;
                        }

                        close(PARENT_WR);
                        close(STDIN);
                        #fcntl(CHILD_RD, F_DUPFD, 0);
                        syscall(&SYS_dup2, fileno(CHILD_RD), 0);
                        #open(STDIN, "<&CHILD_RD");
                        exec($req_params{SCRIPT_FILENAME});
                        die("exec failed");
                }
            }
            else {
                print("Content-type: text/plain\r\n\r\n");
                print "Error: No such CGI app - $req_params{SCRIPT_FILENAME} may not ";
                print "exist or is not executable by this process.\n";
            }

        }
}

chmod +x /usr/local/bin/fastcgi.pl
cd /etc/service
mkdir fastcgi
cd fastcgi
sudo nano run

#!/bin/sh
exec chpst -u www-data -U www-data /usr/bin/perl /usr/local/bin/fastcgi.pl

chmod +x run

copy *.cgi and *.h to /var/www
chown www-data:www-data *.cgi
chown www-data:www-data *.h
chmod +x *.cgi

cpan install common::sense
cpan install File::Slurp
cpan install Convert::Binary::C
cpan install JSON
cpan install DBI
cpan install DBD::Pg
```

AllegZoneBot:
```
copy over the code to ./allegbot
chmod +x allegbot
cd /etc/service
mkdir allegbot
cd allegbot
sudo nano run
#!/bin/sh
exec /home/imago/allegbot

cpan install AnyEvent
cpan install AnyEvent::IRC::Client
cpan install AnyEvent::JSONRPC::Lite::Client
cpan install AnyEvent::JSONRPC::Lite::Server
cpan install RPC::XML::Client
cpan install String::IRC
cpan install WWW::Mechanize

sudo nano /home/imago/pass.txt
allegdb

```

Installer:
```
in /etc/nginx/conf.d/installer.conf"

server {
    listen   80;
    server_name installer.allegiancezone.com installer.spacetechnology.net installer;
    location / {
      rewrite ^ http://cdn.allegiancezone.com/install/AllegSetup_211.exe permanent;
    }
    location /latest {
      rewrite ^ http://cdn.allegiancezone.com/install/AllegSetup_211.exe permanent;
    }
    location /latest.exe {
      rewrite ^ http://cdn.allegiancezone.com/install/AllegSetup_211.exe permanent;
    }
}
```
