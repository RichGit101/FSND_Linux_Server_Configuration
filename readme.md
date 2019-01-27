# Linux Server Configuration

##### Udacity Full Stack Nano Degree Project


A full stack scalable multi-tier capable Item Catalog is demonstrated through restaurant listing for a city

This project uses Amazon lightsail to host its web, application and db tiers.

This project is fully tested and verified. Test metrics are recorded.

## How to Use
Please copy and paste this URL on to a browser

Link
http://ec2-35-177-135-137.eu-west-2.compute.amazonaws.com

or

Click on URL <a> http://ec2-35-177-135-137.eu-west-2.compute.amazonaws.com</a>

IP address is provide for rubric only, please do not use this to reach this application. This is due to recent changes to google Oauth. IP is : 35.177.135.137

### Rubric Checks. Test usecases cases report .
* SSH key as grader works - OK
* root remote logon disabled -OK
* grader can run with sudo as equivalent to root -OK
* Port Check -SSH (port 2200), HTTP (port 80), and NTP (port 123) -OK
* Key-based SSH authentication is enforced -OK
* Ubuntu update and upgrade done - OK
* SSH is hosted on non-default port - moved to 2200. - OK
* The web server app responds on port 80. - OK
* Database server has been configured to serve data (PostgreSQL is used ) - OK
* Web server has been configured to serve the Item Catalog application as a WSGI app. - OK
* A README file is included in the GitHub repo containing the following information: IP address, URL, summary of software installed, summary of configurations made, and a list of third-party resources used to complete this project. - OK (ref below)

### For review 2
OS updated. 
<img src="/resources_static/OS_uptodate.png" >
## Steps Followed

1) Planned the hosting by estimating users, cost and resources required for this project.

Side Note : I made multiple VMS, multiple tests and configurations, finally got it to work.

2) Went to Amazon light sail, signed up to Amazon light sail and created a VM on plain ubuntu as seen in the screenshot below

![Alt text](/resources_static/1First_VM.png "First Look")

3) Connected to VM and verified that purchased VM worked as expected ie Ubuntu out of the box was working as expected.

<img src="/resources_static/2connect_VM.png" >

4) Since Full stack is a iterative process and we have a VM started the modern devops practice of taking VM snapshots.

 <img src="/resources_static/3backup.png" >

5) Now VM OOTB Ubuntu provided by Amazon will have to be secured. First step is to make sure OS is updated and upgraded.

used
<em>
    * $ sudo apt-get update
    * $ sudo apt-get upgrade
    * $ sudo apt-get install finger
</em>


<img src="/resources_static/4upgrade_vm.png" >

6) Updated VM is snapshotted.

7) According to rubric new user Grader is created
<em>
    * sudo adduser grader
    * sudo vi /etc/sudoers.d/grader

    There we add
    grader ALL=(ALL:ALL) ALL
</em>

We also add a fix for loopback issue by editing /etc/hosts file
<em>
    * $ sudo vi /etc/hosts
     and add 127.0.1.1 ip-10-20-37-65 after 127.0.1.1:localhost
</em>




8) While we could have used SSH keys to login as Ubuntu from our local machine, we did not do it. While in reality i used it few times but it is not to be mentioned as step because it is better use grader as most of the rubric leaves us to think that grader is supposed to be the admin or operator for for the application administration. Hence I will go on to sort out grader, give grader to sudo access, shift ports for ssh to 2200 and use grader public and provide RSA keys which i will generate to setup a direct ssh from my home machine.

9) I decided to attach a static Ip to my Amazon Lightsail as it is provided with 5 static addresses for the same VM bill

10) Made sure ip and reverse look up worked and verified it.
11) Added VM firewall port in VM homepage. Added port 2200 as custom port for tcp. We will be configuring ssh on 2200 and close the default later.

12) Now in my local machine i made key pair for grader .
<em>
    * $ ssh-keygen -f ~/.ssh/udacity_key.rsa

</em>
* Verify that two keys are made. The one with .pub extension is a public key. The other is a private key. We already know how this works from udacity classes. Hence we cat and read the public key.

<em>
    * $ cat ~/.ssh/udacity_key.rsa.pub
</em>

13) Now we go to Amazon light sail VM and let us work on two things ie ssh from my local machine to grader on amazon light sail and granting sudo to grader. With both of this we get a local terminal access which is passwordless since it uses keys and it becomes easy for administration and operations.

14) On Amazon light sail VM
<em>
    * $ cd /home/grader
    * $ mkdir .ssh
    * $ vi .ssh/authorized_keys
</em>
Now copy and paste your local public key generated from keygen to this authorised_keys file.
Same operation could be done through scp ie remote secure copy. Tried both and both worked fine.
While using scp use -i for key file and -P for port.

15) Now let us sort out permissions to ssh direcory and keys file. Change ownership to grader
<em>
    * $ sudo chmod 700 /home/grader/.ssh
    * $ sudo chmod 644 /home/grader/.ssh/authorized_keys
    * $ sudo chown -R grader:grader /home/grader/.ssh
</em>

16) Restart the ssh service
<em>
    * $ sudo service ssh restart
</em>
17) Disconnect from vm

18) From local machine (in my case a old mac ) open a terminal.
Log into the server as grader
<em>
    * $ ssh -i ~/.ssh/udacity_key.rsa grader@13.58.109.116
</em>
Next time we will change over ssh service from default port to 2200 by using -p so make a note of the command above.


 <img src="/resources_static/6_multiple_logons.png" >

19) Let us secure passwordless authentication. Mostly it may be already present as No after changes but we go to verify.
<em>
    * $ sudo vi /etc/ssh/sshd_config
</em>
Now locate line with
PasswordAuthentication line and change it to no

20) We change over ssh secure port from default to 2200 as noted by rubric. It is a best practice.
<em>
$ sudo vi /etc/ssh/ssdh_config
</em>

 Find the Port line and change 22 to 2200.
 Restart ssh
<em>
  * $ sudo service ssh restart
</em>

21) From Local Machine, disconnect and reconnect on to 2200 by

<em>
    * ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@13.58.109.116
</em>

22)
Disable ssh login for root user, as required by Udacity

<em>
    * $ sudo vi /etc/ssh/sshd_config
</em>  
 Find the PermitRootLogin line and edit to no

23) Now Configure "Uncomplicated Firewall"
<em>
    * $ sudo ufw allow 2200/tcp
    * $ sudo ufw allow 80/tcp
    * $ sudo ufw allow 123/udp
    * $ sudo ufw enable
</em>

25) Now Let us install Webserver
I have selected Apache, to prepare for deployment we install git. Surpringly git may be already there because its 2019.
<em>
    * $ sudo apt-get install apache2
    * $ sudo apt-get install libapache2-mod-wsgi python-dev
    * $ sudo apt-get install git
</em>

Verify apache from a browsers in your local machine by entering the public IP which you got from amazon VM. In my case i decided to go for a static ip and attach it to VM. It presents with default Apache page.

26) Now, got to mention, i managed to make VM snapshots after every major milestones, hence by now i have about 7 snapshots.

27) To prepare Apache for deployment we do the following
<em>
    * $ cd /var/www
    * $ sudo mkdir catalog
    * $ sudo chown -R grader:grader catalog
    * $ cd catalog
</em>

28) Time to deployment, Git to git clone from my github, its is reviewed project from previous exercise ie Restaurants.
<em>
$ git clone <my git>
</em>

29) There are two files to link Apache to application, one is on apache configuration end and another on application end. We already deployed wsgi module which give the capability for apache to handle python flask project. Now on application end.

Create a .wsgi file
<em>
    * $sudo vi catalog.wsgi
</em>

To maintain application session we use a key. This is presented.
We edit the file as following.
<em>

    import sys
    import logging
    logging.basicConfig(stream=sys.stderr)
    sys.path.insert(0, "/var/www/catalog/")
    from catalog import app as application
    application.secret_key = 'super_secret_key'
</em>

30) To prepare for deplyment we rename project.py
to __init__.py

31) As with any python project, we always maitain a virtual python environment, mostly with pip or with conda (conda if project extended beyond core python, sometime even mini conda)

In this case, went for pip and virtual env
<em>


    $ sudo pip install virtualenv
    $ sudo virtualenv venv
    $ source venv/bin/activate
    $ sudo chmod -R 777 venv
</em>

32) We now have a virtual environment, so we installed all the required module imports and dependencies from .py file

<em>


    $ sudo apt-get install python-pip
    $ sudo pip install Flask
    $ sudo pip install httplib2  (and one by one )
</em>

33) There is a issue with relative and absolute path when apache runs on python flask project hence edit real path in the web server to  __init__.py for the client_secrets.json line to /var/www/catalog/catalog/client_secrets.json

34) Now edit the rest for deployment ie host with our public static ip and port with 80 in the .py file.

35) Now from apache end, we will configure virtual host.
<em>


sudo vi /etc/apache2/sites-available/catalog.conf

    <VirtualHost *:80>
        ServerName [ My Light sail Static Ip]
        ServerAlias [ HOST NAME by reverse lookup]
        ServerAdmin admin@[my lightsail static ip]
        WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
        WSGIProcessGroup catalog
        WSGIScriptAlias / /var/www/catalog/catalog.wsgi
        <Directory /var/www/catalog/catalog/>
            Order allow,deny
            Allow from all
        </Directory>
        Alias /static /var/www/catalog/catalog/static
        <Directory /var/www/catalog/catalog/static/>
            Order allow,deny
            Allow from all
        </Directory>
        ErrorLog ${APACHE_LOG_DIR}/error.log
        LogLevel warn
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>


</em>

35) Database setup is done
<em>

    $ sudo apt-get install libpq-dev python-dev
    $ sudo apt-get install postgresql postgresql-contrib
    $ sudo -u postgres -i

</em>

36) Setup DB and user. Edit access.
<em>


    $ CREATE USER catalog WITH PASSWORD [your password];
    $ ALTER USER catalog CREATEDB;
    $ CREATE DATABASE catalog WITH OWNER catalog;
    Connect to database $ \c catalog
    $ REVOKE ALL ON SCHEMA public FROM public;
    $ GRANT ALL ON SCHEMA public TO catalog;
    Quit the postgrel command line: $ \c and then $ exit
</em>

37) Now driver for DB and connection strings will have to be edited. There are three edits required for this project. One for main __init.py__ and for DB setup for models and lots of menus for sample data.

<em>


    engine = create_engine('postgresql://catalog:[your password]@localhost/catalog
</em>

38) Now apache services are to be restarted.
<em>
sudo service apache2 restart
</em>

39)We had encountered issues do Google OAuth console was checked and modified to remove old entries and make sure it is ready.

<img src="/resources_static/Final_oauth.png" >

40) Template login html is verified.
41) We verified the JSON manually through a JSON editor
<img src="/resources_static/verify_json.png" >

42) Final working and Unit testing was done. Metrics were measured. Browser level and web logs were checked. DB was checked.
<img src="/resources_static/Final_working.png" >

43) Rubrics were checked and project was prepared for submission

Firewall reads
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW IN    Anywhere                  
80/tcp                     ALLOW IN    Anywhere                  
123/udp                    ALLOW IN    Anywhere                  
2200/tcp (v6)              ALLOW IN    Anywhere (v6)             
80/tcp (v6)                ALLOW IN    Anywhere (v6)             
123/udp (v6)               ALLOW IN    Anywhere (v6)             





### Thanks and acknowledgements
Thanks to Udacity staff, Udacity Instrutors and Udacity Mentors for helping out. I made multiple VMS, multiple tests and configurations, finally got it to work. Thanks to all the help
Thanks to Knowledge hub, and class room chat, I will be closing my open questions shortly on all of them. Thanks to stackoverflow, a ready reference.
I used various books, sites, youtube videos for references, thanks to them all. Thanks to github of iliketomatoes, stueken and chuanqin too. I will be adding more to the list as i improve on this time to time.
Thanks for reading my readme.md.


----------

#### Extras


#### 1) List of all Pip modules 
certifi==2018.11.29
chardet==3.0.4
Click==7.0
Flask==1.0.2
httplib2==0.12.0
idna==2.8
itsdangerous==1.1.0
Jinja2==2.10
jsonify==0.5
MarkupSafe==1.1.0
oauth2client==4.1.3
psycopg2==2.7.7
psycopg2-binary==2.7.7
pyasn1==0.4.5
pyasn1-modules==0.2.3
requests==2.21.0
rsa==4.0
six==1.12.0
SQLAlchemy==1.2.16
urllib3==1.24.1
Werkzeug==0.14.1

### 2) List of all Linux packages

(venv) grader@ip-172-26-0-84:/var/www/catalog/venv/bin$ apt list --installed
Listing... Done
accountsservice/xenial-updates,now 0.6.40-2ubuntu11.3 amd64 [installed]
acl/xenial,now 2.2.52-3 amd64 [installed]
acpid/xenial,now 1:2.0.26-1ubuntu2 amd64 [installed]
adduser/xenial,now 3.113+nmu3ubuntu4 all [installed]
apache2/xenial-updates,now 2.4.18-2ubuntu3.9 amd64 [installed]
apache2-bin/xenial-updates,now 2.4.18-2ubuntu3.9 amd64 [installed,automatic]
apache2-data/xenial-updates,now 2.4.18-2ubuntu3.9 all [installed,automatic]
apache2-utils/xenial-updates,now 2.4.18-2ubuntu3.9 amd64 [installed,automatic]
apparmor/xenial-updates,xenial-security,now 2.10.95-0ubuntu2.10 amd64 [installed]
apport/xenial-updates,xenial-security,now 2.20.1-0ubuntu2.18 all [installed]
apport-symptoms/xenial,now 0.20 all [installed]
apt/xenial-updates,xenial-security,now 1.2.29ubuntu0.1 amd64 [installed]
apt-transport-https/xenial-updates,xenial-security,now 1.2.29ubuntu0.1 amd64 [installed]
apt-utils/xenial-updates,xenial-security,now 1.2.29ubuntu0.1 amd64 [installed]
at/xenial,now 3.1.18-2ubuntu1 amd64 [installed]
base-files/xenial-updates,now 9.4ubuntu4.7 amd64 [installed]
base-passwd/xenial,now 3.5.39 amd64 [installed]
bash/xenial-updates,xenial-security,now 4.3-14ubuntu1.2 amd64 [installed]
bash-completion/xenial-updates,now 1:2.1-4.2ubuntu1.1 all [installed]
bcache-tools/xenial,now 1.0.8-2 amd64 [installed]
bind9-host/xenial-updates,xenial-security,now 1:9.10.3.dfsg.P4-8ubuntu1.11 amd64 [installed]
binutils/xenial-updates,now 2.26.1-1ubuntu1~16.04.7 amd64 [installed,automatic]
bsdmainutils/xenial,now 9.0.6ubuntu3 amd64 [installed]
bsdutils/xenial-updates,now 1:2.27.1-6ubuntu3.6 amd64 [installed]
btrfs-tools/xenial-updates,now 4.4-1ubuntu1 amd64 [installed]
build-essential/xenial,now 12.1ubuntu2 amd64 [installed]
busybox-initramfs/xenial,now 1:1.22.0-15ubuntu1 amd64 [installed]
busybox-static/xenial,now 1:1.22.0-15ubuntu1 amd64 [installed]
byobu/xenial,now 5.106-0ubuntu1 all [installed]
bzip2/xenial,now 1.0.6-8 amd64 [installed]
ca-certificates/xenial-updates,now 20170717~16.04.2 all [installed]
cloud-guest-utils/xenial-updates,now 0.27-0ubuntu25.1 all [installed]
cloud-init/xenial-updates,now 18.4-0ubuntu1~16.04.2 all [installed]
cloud-initramfs-copymods/xenial-updates,now 0.27ubuntu1.6 all [installed]
cloud-initramfs-dyn-netconf/xenial-updates,now 0.27ubuntu1.6 all [installed]
comerr-dev/xenial,now 2.1-1.42.13-1ubuntu1 amd64 [installed,automatic]
command-not-found/xenial-updates,now 0.3ubuntu16.04.2 all [installed]
command-not-found-data/xenial-updates,now 0.3ubuntu16.04.2 amd64 [installed]
console-setup/xenial-updates,now 1.108ubuntu15.4 all [installed]
console-setup-linux/xenial-updates,now 1.108ubuntu15.4 all [installed]
coreutils/xenial-updates,now 8.25-2ubuntu3~16.04 amd64 [installed]
cpio/xenial,now 2.11+dfsg-5ubuntu1 amd64 [installed]
cpp/xenial,now 4:5.3.1-1ubuntu1 amd64 [installed,automatic]
cpp-5/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed,automatic]
cron/xenial,now 3.0pl1-128ubuntu2 amd64 [installed]
cryptsetup/xenial-updates,now 2:1.6.6-5ubuntu2.1 amd64 [installed]
cryptsetup-bin/xenial-updates,now 2:1.6.6-5ubuntu2.1 amd64 [installed]
curl/xenial-updates,xenial-security,now 7.47.0-1ubuntu2.11 amd64 [installed]
dash/xenial,now 0.5.8-2.1ubuntu2 amd64 [installed]
dbus/xenial-updates,now 1.10.6-1ubuntu3.3 amd64 [installed]
debconf/xenial,now 1.5.58ubuntu1 all [installed]
debconf-i18n/xenial,now 1.5.58ubuntu1 all [installed]
debianutils/xenial,now 4.7 amd64 [installed]
dh-python/xenial-updates,now 2.20151103ubuntu1.1 all [installed]
diffutils/xenial,now 1:3.3-3 amd64 [installed]
distro-info-data/xenial-updates,xenial-security,now 0.28ubuntu0.9 all [installed]
dmeventd/xenial,now 2:1.02.110-1ubuntu10 amd64 [installed]
dmidecode/xenial-updates,now 3.0-2ubuntu0.1 amd64 [installed]
dmsetup/xenial,now 2:1.02.110-1ubuntu10 amd64 [installed]
dns-root-data/xenial-updates,xenial-security,now 2018013001~16.04.1 all [installed]
dnsmasq-base/xenial-updates,xenial-security,now 2.75-1ubuntu0.16.04.5 amd64 [installed]
dnsutils/xenial-updates,xenial-security,now 1:9.10.3.dfsg.P4-8ubuntu1.11 amd64 [installed]
dosfstools/xenial-updates,xenial-security,now 3.0.28-2ubuntu0.1 amd64 [installed]
dpkg/xenial-updates,now 1.18.4ubuntu1.5 amd64 [installed]
dpkg-dev/xenial-updates,now 1.18.4ubuntu1.5 all [installed,automatic]
e2fslibs/xenial,now 1.42.13-1ubuntu1 amd64 [installed]
e2fsprogs/xenial,now 1.42.13-1ubuntu1 amd64 [installed]
eatmydata/xenial,now 105-3 all [installed]
ed/xenial,now 1.10-2 amd64 [installed]
eject/xenial-updates,xenial-security,now 2.1.5+deb1+cvs20081104-13.1ubuntu0.16.04.1 amd64 [installed]
ethtool/xenial,now 1:4.5-1 amd64 [installed]
fakeroot/xenial,now 1.20.2-1ubuntu1 amd64 [installed,automatic]
file/xenial-updates,xenial-security,now 1:5.25-2ubuntu1.1 amd64 [installed]
findutils/xenial,now 4.6.0+git+20160126-2 amd64 [installed]
finger/xenial,now 0.17-15 amd64 [installed]
fonts-ubuntu-font-family-console/xenial,now 1:0.83-0ubuntu2 all [installed]
friendly-recovery/xenial-updates,now 0.2.31ubuntu2 all [installed]
ftp/xenial,now 0.17-33 amd64 [installed]
fuse/xenial-updates,now 2.9.4-1ubuntu3.1 amd64 [installed]
g++/xenial,now 4:5.3.1-1ubuntu1 amd64 [installed,automatic]
g++-5/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed,automatic]
gawk/xenial,now 1:4.1.3+dfsg-0.1 amd64 [installed]
gcc/xenial,now 4:5.3.1-1ubuntu1 amd64 [installed,automatic]
gcc-5/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed,automatic]
gcc-5-base/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed]
gcc-6-base/xenial,now 6.0.1-0ubuntu1 amd64 [installed]
gdisk/xenial,now 1.0.1-1build1 amd64 [installed]
geoip-database/xenial,now 20160408-1 all [installed]
gettext-base/xenial-updates,xenial-security,now 0.19.7-2ubuntu3.1 amd64 [installed]
gir1.2-glib-2.0/xenial,now 1.46.0-3ubuntu1 amd64 [installed]
git/xenial-updates,xenial-security,now 1:2.7.4-0ubuntu1.6 amd64 [installed]
git-man/xenial-updates,xenial-security,now 1:2.7.4-0ubuntu1.6 all [installed]
gnupg/xenial-updates,xenial-security,now 1.4.20-1ubuntu3.3 amd64 [installed]
gpgv/xenial-updates,xenial-security,now 1.4.20-1ubuntu3.3 amd64 [installed]
grep/xenial-updates,now 2.25-1~16.04.1 amd64 [installed]
groff-base/xenial,now 1.22.3-7 amd64 [installed]
grub-common/xenial-updates,now 2.02~beta2-36ubuntu3.20 amd64 [installed,automatic]
grub-gfxpayload-lists/xenial,now 0.7 amd64 [installed,automatic]
grub-legacy-ec2/xenial-updates,now 18.4-0ubuntu1~16.04.2 all [installed]
grub-pc/xenial-updates,now 2.02~beta2-36ubuntu3.20 amd64 [installed,automatic]
grub-pc-bin/xenial-updates,now 2.02~beta2-36ubuntu3.20 amd64 [installed,automatic]
grub2-common/xenial-updates,now 2.02~beta2-36ubuntu3.20 amd64 [installed,automatic]
gzip/xenial,now 1.6-4ubuntu1 amd64 [installed]
hdparm/xenial-updates,now 9.48+ds-1ubuntu0.1 amd64 [installed]
hibagent/xenial-updates,now 1.0.1-0ubuntu1~16.04.1 all [installed]
hostname/xenial,now 3.16ubuntu2 amd64 [installed]
ifenslave/xenial,now 2.7ubuntu1 all [installed]
ifupdown/xenial-updates,now 0.8.10ubuntu1.4 amd64 [installed]
info/xenial,now 6.1.0.dfsg.1-5 amd64 [installed]
init/xenial-updates,now 1.29ubuntu4 amd64 [installed]
init-system-helpers/xenial-updates,now 1.29ubuntu4 all [installed]
initramfs-tools/xenial-updates,xenial-security,now 0.122ubuntu8.14 all [installed]
initramfs-tools-bin/xenial-updates,xenial-security,now 0.122ubuntu8.14 amd64 [installed]
initramfs-tools-core/xenial-updates,xenial-security,now 0.122ubuntu8.14 all [installed]
initscripts/xenial,now 2.88dsf-59.3ubuntu2 amd64 [installed]
insserv/xenial,now 1.14.0-5ubuntu3 amd64 [installed]
install-info/xenial,now 6.1.0.dfsg.1-5 amd64 [installed]
iproute2/xenial-updates,now 4.3.0-1ubuntu3.16.04.4 amd64 [installed]
iptables/xenial,now 1.6.0-2ubuntu3 amd64 [installed]
iputils-ping/xenial,now 3:20121221-5ubuntu2 amd64 [installed]
iputils-tracepath/xenial,now 3:20121221-5ubuntu2 amd64 [installed]
irqbalance/xenial,now 1.1.0-2ubuntu1 amd64 [installed]
isc-dhcp-client/xenial-updates,now 4.3.3-5ubuntu12.10 amd64 [installed]
isc-dhcp-common/xenial-updates,now 4.3.3-5ubuntu12.10 amd64 [installed]
iso-codes/xenial,now 3.65-1 all [installed]
kbd/xenial-updates,now 1.15.5-1ubuntu5 amd64 [installed]
keyboard-configuration/xenial-updates,now 1.108ubuntu15.4 all [installed]
klibc-utils/xenial-updates,now 2.0.4-8ubuntu1.16.04.4 amd64 [installed]
kmod/xenial-updates,now 22-1ubuntu5.1 amd64 [installed]
krb5-locales/xenial-updates,xenial-security,now 1.13.2+dfsg-5ubuntu2.1 all [installed]
krb5-multidev/xenial-updates,xenial-security,now 1.13.2+dfsg-5ubuntu2.1 amd64 [installed,automatic]
language-selector-common/xenial-updates,now 0.165.4 all [installed]
less/xenial-updates,now 481-2.1ubuntu0.2 amd64 [installed]
libaccountsservice0/xenial-updates,now 0.6.40-2ubuntu11.3 amd64 [installed]
libacl1/xenial,now 2.2.52-3 amd64 [installed]
libalgorithm-diff-perl/xenial,now 1.19.03-1 all [installed,automatic]
libalgorithm-diff-xs-perl/xenial,now 0.04-4build1 amd64 [installed,automatic]
libalgorithm-merge-perl/xenial,now 0.08-3 all [installed,automatic]
libapache2-mod-wsgi/xenial,now 4.3.0-1.1build1 amd64 [installed]
libapparmor-perl/xenial-updates,xenial-security,now 2.10.95-0ubuntu2.10 amd64 [installed]
libapparmor1/xenial-updates,xenial-security,now 2.10.95-0ubuntu2.10 amd64 [installed]
libapr1/xenial,now 1.5.2-3 amd64 [installed,automatic]
libaprutil1/xenial,now 1.5.4-1build1 amd64 [installed,automatic]
libaprutil1-dbd-sqlite3/xenial,now 1.5.4-1build1 amd64 [installed,automatic]
libaprutil1-ldap/xenial,now 1.5.4-1build1 amd64 [installed,automatic]
libapt-inst2.0/xenial-updates,xenial-security,now 1.2.29ubuntu0.1 amd64 [installed]
libapt-pkg5.0/xenial-updates,xenial-security,now 1.2.29ubuntu0.1 amd64 [installed]
libasan2/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed,automatic]
libasn1-8-heimdal/xenial-updates,xenial-security,now 1.7~git20150920+dfsg-4ubuntu1.16.04.1 amd64 [installed]
libasprintf0v5/xenial-updates,xenial-security,now 0.19.7-2ubuntu3.1 amd64 [installed]
libatm1/xenial,now 1:2.5.1-1.5 amd64 [installed]
libatomic1/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed,automatic]
libattr1/xenial,now 1:2.4.47-2 amd64 [installed]
libaudit-common/xenial-updates,now 1:2.4.5-1ubuntu2.1 all [installed]
libaudit1/xenial-updates,now 1:2.4.5-1ubuntu2.1 amd64 [installed]
libbind9-140/xenial-updates,xenial-security,now 1:9.10.3.dfsg.P4-8ubuntu1.11 amd64 [installed]
libblkid1/xenial-updates,now 2.27.1-6ubuntu3.6 amd64 [installed]
libbsd0/xenial,now 0.8.2-1 amd64 [installed]
libbz2-1.0/xenial,now 1.0.6-8 amd64 [installed]
libc-bin/xenial-updates,xenial-security,now 2.23-0ubuntu10 amd64 [installed]
libc-dev-bin/xenial-updates,xenial-security,now 2.23-0ubuntu10 amd64 [installed,automatic]
libc6/xenial-updates,xenial-security,now 2.23-0ubuntu10 amd64 [installed]
libc6-dev/xenial-updates,xenial-security,now 2.23-0ubuntu10 amd64 [installed,automatic]
libcap-ng0/xenial,now 0.7.7-1 amd64 [installed]
libcap2/xenial,now 1:2.24-12 amd64 [installed]
libcap2-bin/xenial,now 1:2.24-12 amd64 [installed]
libcc1-0/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed,automatic]
libcilkrts5/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed,automatic]
libcomerr2/xenial,now 1.42.13-1ubuntu1 amd64 [installed]
libcryptsetup4/xenial-updates,now 2:1.6.6-5ubuntu2.1 amd64 [installed]
libcurl3-gnutls/xenial-updates,xenial-security,now 7.47.0-1ubuntu2.11 amd64 [installed]
libdb5.3/xenial-updates,xenial-security,now 5.3.28-11ubuntu0.1 amd64 [installed]
libdbus-1-3/xenial-updates,now 1.10.6-1ubuntu3.3 amd64 [installed]
libdbus-glib-1-2/xenial,now 0.106-1 amd64 [installed]
libdebconfclient0/xenial,now 0.198ubuntu1 amd64 [installed]
libdevmapper-event1.02.1/xenial,now 2:1.02.110-1ubuntu10 amd64 [installed]
libdevmapper1.02.1/xenial,now 2:1.02.110-1ubuntu10 amd64 [installed]
libdns-export162/xenial-updates,xenial-security,now 1:9.10.3.dfsg.P4-8ubuntu1.11 amd64 [installed]
libdns162/xenial-updates,xenial-security,now 1:9.10.3.dfsg.P4-8ubuntu1.11 amd64 [installed]
libdpkg-perl/xenial-updates,now 1.18.4ubuntu1.5 all [installed,automatic]
libdrm-common/xenial-updates,now 2.4.91-2~16.04.1 all [installed,automatic]
libdrm2/xenial-updates,now 2.4.91-2~16.04.1 amd64 [installed]
libdumbnet1/xenial,now 1.12-7 amd64 [installed]
libeatmydata1/xenial,now 105-3 amd64 [installed]
libedit2/xenial,now 3.1-20150325-1ubuntu2 amd64 [installed]
libelf1/xenial-updates,xenial-security,now 0.165-3ubuntu1.1 amd64 [installed]
liberror-perl/xenial,now 0.17-1.2 all [installed]
libestr0/xenial,now 0.1.10-1 amd64 [installed]
libevent-2.0-5/xenial-updates,xenial-security,now 2.0.21-stable-2ubuntu0.16.04.1 amd64 [installed]
libexpat1/xenial-updates,xenial-security,now 2.1.0-7ubuntu0.16.04.3 amd64 [installed]
libexpat1-dev/xenial-updates,xenial-security,now 2.1.0-7ubuntu0.16.04.3 amd64 [installed,automatic]
libfakeroot/xenial,now 1.20.2-1ubuntu1 amd64 [installed,automatic]
libfdisk1/xenial-updates,now 2.27.1-6ubuntu3.6 amd64 [installed]
libffi6/xenial,now 3.2.1-4 amd64 [installed]
libfile-fcntllock-perl/xenial,now 0.22-3 amd64 [installed,automatic]
libfreetype6/xenial-updates,xenial-security,now 2.6.1-0.1ubuntu2.3 amd64 [installed,automatic]
libfribidi0/xenial,now 0.19.7-1 amd64 [installed]
libfuse2/xenial-updates,now 2.9.4-1ubuntu3.1 amd64 [installed]
libgcc-5-dev/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed,automatic]
libgcc1/xenial,now 1:6.0.1-0ubuntu1 amd64 [installed]
libgcrypt20/xenial-updates,xenial-security,now 1.6.5-2ubuntu0.5 amd64 [installed]
libgdbm3/xenial,now 1.8.3-13.1 amd64 [installed]
libgeoip1/xenial,now 1.6.9-1 amd64 [installed]
libgirepository-1.0-1/xenial,now 1.46.0-3ubuntu1 amd64 [installed]
libglib2.0-0/xenial-updates,xenial-security,now 2.48.2-0ubuntu4.1 amd64 [installed]
libglib2.0-data/xenial-updates,xenial-security,now 2.48.2-0ubuntu4.1 all [installed]
libgmp10/xenial,now 2:6.1.0+dfsg-2 amd64 [installed]
libgnutls-openssl27/xenial-updates,now 3.4.10-4ubuntu1.4 amd64 [installed]
libgnutls30/xenial-updates,now 3.4.10-4ubuntu1.4 amd64 [installed]
libgomp1/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed,automatic]
libgpg-error0/xenial,now 1.21-2ubuntu1 amd64 [installed]
libgpm2/xenial,now 1.20.4-6.1 amd64 [installed]
libgssapi-krb5-2/xenial-updates,xenial-security,now 1.13.2+dfsg-5ubuntu2.1 amd64 [installed]
libgssapi3-heimdal/xenial-updates,xenial-security,now 1.7~git20150920+dfsg-4ubuntu1.16.04.1 amd64 [installed]
libgssrpc4/xenial-updates,xenial-security,now 1.13.2+dfsg-5ubuntu2.1 amd64 [installed,automatic]
libhcrypto4-heimdal/xenial-updates,xenial-security,now 1.7~git20150920+dfsg-4ubuntu1.16.04.1 amd64 [installed]
libheimbase1-heimdal/xenial-updates,xenial-security,now 1.7~git20150920+dfsg-4ubuntu1.16.04.1 amd64 [installed]
libheimntlm0-heimdal/xenial-updates,xenial-security,now 1.7~git20150920+dfsg-4ubuntu1.16.04.1 amd64 [installed]
libhogweed4/xenial-updates,xenial-security,now 3.2-1ubuntu0.16.04.1 amd64 [installed]
libhx509-5-heimdal/xenial-updates,xenial-security,now 1.7~git20150920+dfsg-4ubuntu1.16.04.1 amd64 [installed]
libicu55/xenial-updates,xenial-security,now 55.1-7ubuntu0.4 amd64 [installed]
libidn11/xenial-updates,xenial-security,now 1.32-3ubuntu1.2 amd64 [installed]
libisc-export160/xenial-updates,xenial-security,now 1:9.10.3.dfsg.P4-8ubuntu1.11 amd64 [installed]
libisc160/xenial-updates,xenial-security,now 1:9.10.3.dfsg.P4-8ubuntu1.11 amd64 [installed]
libisccc140/xenial-updates,xenial-security,now 1:9.10.3.dfsg.P4-8ubuntu1.11 amd64 [installed]
libisccfg140/xenial-updates,xenial-security,now 1:9.10.3.dfsg.P4-8ubuntu1.11 amd64 [installed]
libisl15/xenial,now 0.16.1-1 amd64 [installed,automatic]
libitm1/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed,automatic]
libjson-c2/xenial,now 0.11-4ubuntu2 amd64 [installed]
libk5crypto3/xenial-updates,xenial-security,now 1.13.2+dfsg-5ubuntu2.1 amd64 [installed]
libkadm5clnt-mit9/xenial-updates,xenial-security,now 1.13.2+dfsg-5ubuntu2.1 amd64 [installed,automatic]
libkadm5srv-mit9/xenial-updates,xenial-security,now 1.13.2+dfsg-5ubuntu2.1 amd64 [installed,automatic]
libkdb5-8/xenial-updates,xenial-security,now 1.13.2+dfsg-5ubuntu2.1 amd64 [installed,automatic]
libkeyutils1/xenial,now 1.5.9-8ubuntu1 amd64 [installed]
libklibc/xenial-updates,now 2.0.4-8ubuntu1.16.04.4 amd64 [installed]
libkmod2/xenial-updates,now 22-1ubuntu5.1 amd64 [installed]
libkrb5-26-heimdal/xenial-updates,xenial-security,now 1.7~git20150920+dfsg-4ubuntu1.16.04.1 amd64 [installed]
libkrb5-3/xenial-updates,xenial-security,now 1.13.2+dfsg-5ubuntu2.1 amd64 [installed]
libkrb5support0/xenial-updates,xenial-security,now 1.13.2+dfsg-5ubuntu2.1 amd64 [installed]
libldap-2.4-2/xenial-updates,now 2.4.42+dfsg-2ubuntu3.4 amd64 [installed]
liblocale-gettext-perl/xenial,now 1.07-1build1 amd64 [installed]
liblsan0/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed,automatic]
liblua5.1-0/xenial,now 5.1.5-8ubuntu1 amd64 [installed,automatic]
liblvm2app2.2/xenial,now 2.02.133-1ubuntu10 amd64 [installed]
liblvm2cmd2.02/xenial,now 2.02.133-1ubuntu10 amd64 [installed]
liblwres141/xenial-updates,xenial-security,now 1:9.10.3.dfsg.P4-8ubuntu1.11 amd64 [installed]
liblxc1/xenial-updates,now 2.0.8-0ubuntu1~16.04.2 amd64 [installed]
liblz4-1/xenial,now 0.0~r131-2ubuntu2 amd64 [installed]
liblzma5/xenial,now 5.1.1alpha+20120614-2ubuntu2 amd64 [installed]
liblzo2-2/xenial,now 2.08-1.2 amd64 [installed]
libmagic1/xenial-updates,xenial-security,now 1:5.25-2ubuntu1.1 amd64 [installed]
libmnl0/xenial,now 1.0.3-5 amd64 [installed]
libmount1/xenial-updates,now 2.27.1-6ubuntu3.6 amd64 [installed]
libmpc3/xenial,now 1.0.3-1 amd64 [installed,automatic]
libmpdec2/xenial,now 2.4.2-1 amd64 [installed]
libmpfr4/xenial,now 3.1.4-1 amd64 [installed]
libmpx0/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed,automatic]
libmspack0/xenial-updates,xenial-security,now 0.5-1ubuntu0.16.04.3 amd64 [installed]
libncurses5/xenial,now 6.0+20160213-1ubuntu1 amd64 [installed]
libncursesw5/xenial,now 6.0+20160213-1ubuntu1 amd64 [installed]
libnetfilter-conntrack3/xenial,now 1.0.5-1 amd64 [installed]
libnettle6/xenial-updates,xenial-security,now 3.2-1ubuntu0.16.04.1 amd64 [installed]
libnewt0.52/xenial,now 0.52.18-1ubuntu2 amd64 [installed]
libnfnetlink0/xenial,now 1.0.1-3 amd64 [installed]
libnih1/xenial,now 1.0.3-4.3ubuntu1 amd64 [installed]
libnuma1/xenial-updates,xenial-security,now 2.0.11-1ubuntu1.1 amd64 [installed]
libp11-kit0/xenial-updates,now 0.23.2-5~ubuntu16.04.1 amd64 [installed]
libpam-modules/xenial-updates,now 1.1.8-3.2ubuntu2.1 amd64 [installed]
libpam-modules-bin/xenial-updates,now 1.1.8-3.2ubuntu2.1 amd64 [installed]
libpam-runtime/xenial-updates,now 1.1.8-3.2ubuntu2.1 all [installed]
libpam-systemd/xenial-updates,xenial-security,now 229-4ubuntu21.15 amd64 [installed]
libpam0g/xenial-updates,now 1.1.8-3.2ubuntu2.1 amd64 [installed]
libparted2/xenial-updates,now 3.2-15ubuntu0.1 amd64 [installed]
libpcap0.8/xenial,now 1.7.4-2 amd64 [installed]
libpci3/xenial-updates,now 1:3.3.1-1.1ubuntu1.2 amd64 [installed]
libpcre3/xenial,now 2:8.38-3.1 amd64 [installed]
libperl5.22/xenial-updates,xenial-security,now 5.22.1-9ubuntu0.6 amd64 [installed]
libpipeline1/xenial,now 1.4.1-2 amd64 [installed]
libplymouth4/xenial-updates,now 0.9.2-3ubuntu13.5 amd64 [installed]
libpng12-0/xenial-updates,xenial-security,now 1.2.54-1ubuntu1.1 amd64 [installed]
libpolkit-agent-1-0/xenial-updates,xenial-security,now 0.105-14.1ubuntu0.4 amd64 [installed]
libpolkit-backend-1-0/xenial-updates,xenial-security,now 0.105-14.1ubuntu0.4 amd64 [installed]
libpolkit-gobject-1-0/xenial-updates,xenial-security,now 0.105-14.1ubuntu0.4 amd64 [installed]
libpopt0/xenial,now 1.16-10 amd64 [installed]
libpq-dev/xenial-updates,xenial-security,now 9.5.14-0ubuntu0.16.04 amd64 [installed]
libpq5/xenial-updates,xenial-security,now 9.5.14-0ubuntu0.16.04 amd64 [installed,automatic]
libprocps4/xenial-updates,xenial-security,now 2:3.3.10-4ubuntu2.4 amd64 [installed]
libpython-all-dev/xenial-updates,now 2.7.12-1~16.04 amd64 [installed,automatic]
libpython-dev/xenial-updates,now 2.7.12-1~16.04 amd64 [installed,automatic]
libpython-stdlib/xenial-updates,now 2.7.12-1~16.04 amd64 [installed,automatic]
libpython2.7/xenial-updates,xenial-security,now 2.7.12-1ubuntu0~16.04.4 amd64 [installed,automatic]
libpython2.7-dev/xenial-updates,xenial-security,now 2.7.12-1ubuntu0~16.04.4 amd64 [installed,automatic]
libpython2.7-minimal/xenial-updates,xenial-security,now 2.7.12-1ubuntu0~16.04.4 amd64 [installed,automatic]
libpython2.7-stdlib/xenial-updates,xenial-security,now 2.7.12-1ubuntu0~16.04.4 amd64 [installed,automatic]
libpython3-stdlib/xenial,now 3.5.1-3 amd64 [installed]
libpython3.5/xenial-updates,xenial-security,now 3.5.2-2ubuntu0~16.04.5 amd64 [installed,automatic]
libpython3.5-minimal/xenial-updates,xenial-security,now 3.5.2-2ubuntu0~16.04.5 amd64 [installed]
libpython3.5-stdlib/xenial-updates,xenial-security,now 3.5.2-2ubuntu0~16.04.5 amd64 [installed]
libquadmath0/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed,automatic]
libreadline5/xenial,now 5.2+dfsg-3build1 amd64 [installed]
libreadline6/xenial,now 6.3-8ubuntu2 amd64 [installed]
libroken18-heimdal/xenial-updates,xenial-security,now 1.7~git20150920+dfsg-4ubuntu1.16.04.1 amd64 [installed]
librtmp1/xenial-updates,xenial-security,now 2.4+20151223.gitfa8646d-1ubuntu0.1 amd64 [installed]
libsasl2-2/xenial-updates,now 2.1.26.dfsg1-14ubuntu0.1 amd64 [installed]
libsasl2-modules/xenial-updates,now 2.1.26.dfsg1-14ubuntu0.1 amd64 [installed]
libsasl2-modules-db/xenial-updates,now 2.1.26.dfsg1-14ubuntu0.1 amd64 [installed]
libseccomp2/xenial-updates,now 2.3.1-2.1ubuntu2~16.04.1 amd64 [installed]
libselinux1/xenial,now 2.4-3build2 amd64 [installed]
libsemanage-common/xenial,now 2.3-1build3 all [installed]
libsemanage1/xenial,now 2.3-1build3 amd64 [installed]
libsensors4/xenial,now 1:3.4.0-2 amd64 [installed,automatic]
libsepol1/xenial,now 2.4-2 amd64 [installed]
libsigsegv2/xenial,now 2.10-4 amd64 [installed]
libslang2/xenial-updates,now 2.3.0-2ubuntu1.1 amd64 [installed]
libsmartcols1/xenial-updates,now 2.27.1-6ubuntu3.6 amd64 [installed]
libsqlite3-0/xenial,now 3.11.0-1ubuntu1 amd64 [installed]
libsqlite3-dev/xenial,now 3.11.0-1ubuntu1 amd64 [installed]
libss2/xenial,now 1.42.13-1ubuntu1 amd64 [installed]
libssl-dev/xenial-updates,xenial-security,now 1.0.2g-1ubuntu4.14 amd64 [installed,automatic]
libssl-doc/xenial-updates,xenial-security,now 1.0.2g-1ubuntu4.14 all [installed,automatic]
libssl1.0.0/xenial-updates,xenial-security,now 1.0.2g-1ubuntu4.14 amd64 [installed]
libstdc++-5-dev/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed,automatic]
libstdc++6/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed]
libsystemd0/xenial-updates,xenial-security,now 229-4ubuntu21.15 amd64 [installed]
libtasn1-6/xenial-updates,xenial-security,now 4.7-3ubuntu0.16.04.3 amd64 [installed]
libtext-charwidth-perl/xenial,now 0.04-7build5 amd64 [installed]
libtext-iconv-perl/xenial,now 1.7-5build4 amd64 [installed]
libtext-wrapi18n-perl/xenial,now 0.06-7.1 all [installed]
libtinfo5/xenial,now 6.0+20160213-1ubuntu1 amd64 [installed]
libtsan0/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed,automatic]
libubsan0/xenial-updates,now 5.4.0-6ubuntu1~16.04.11 amd64 [installed,automatic]
libudev1/xenial-updates,xenial-security,now 229-4ubuntu21.15 amd64 [installed]
libusb-0.1-4/xenial,now 2:0.1.12-28 amd64 [installed]
libusb-1.0-0/xenial,now 2:1.0.20-1 amd64 [installed]
libustr-1.0-1/xenial,now 1.0.4-5 amd64 [installed]
libutempter0/xenial,now 1.1.6-3 amd64 [installed]
libuuid1/xenial-updates,now 2.27.1-6ubuntu3.6 amd64 [installed]
libwind0-heimdal/xenial-updates,xenial-security,now 1.7~git20150920+dfsg-4ubuntu1.16.04.1 amd64 [installed]
libwrap0/xenial,now 7.6.q-25 amd64 [installed]
libx11-6/xenial-updates,xenial-security,now 2:1.6.3-1ubuntu2.1 amd64 [installed]
libx11-data/xenial-updates,xenial-security,now 2:1.6.3-1ubuntu2.1 all [installed]
libxau6/xenial,now 1:1.0.8-1 amd64 [installed]
libxcb1/xenial,now 1.11.1-1ubuntu1 amd64 [installed]
libxdmcp6/xenial,now 1:1.1.2-1.1 amd64 [installed]
libxext6/xenial,now 2:1.3.3-1 amd64 [installed]
libxml2/xenial-updates,xenial-security,now 2.9.3+dfsg1-1ubuntu0.6 amd64 [installed]
libxmuu1/xenial,now 2:1.1.2-2 amd64 [installed]
libxslt1.1/xenial-updates,xenial-security,now 1.1.28-2.1ubuntu0.1 amd64 [installed,automatic]
libxtables11/xenial,now 1.6.0-2ubuntu3 amd64 [installed]
libyaml-0-2/xenial,now 0.1.6-3 amd64 [installed]
linux-aws/xenial-updates,xenial-security,now 4.4.0.1074.76 amd64 [installed]
linux-aws-headers-4.4.0-1052/xenial-updates,xenial-security,now 4.4.0-1052.61 all [installed]
linux-aws-headers-4.4.0-1074/xenial-updates,xenial-security,now 4.4.0-1074.84 all [installed,automatic]
linux-base/xenial-updates,xenial-security,now 4.5ubuntu1~16.04.1 all [installed]
linux-headers-4.4.0-1052-aws/xenial-updates,xenial-security,now 4.4.0-1052.61 amd64 [installed]
linux-headers-4.4.0-1074-aws/xenial-updates,xenial-security,now 4.4.0-1074.84 amd64 [installed,automatic]
linux-headers-aws/xenial-updates,xenial-security,now 4.4.0.1074.76 amd64 [installed]
linux-image-4.4.0-1052-aws/xenial-updates,xenial-security,now 4.4.0-1052.61 amd64 [installed]
linux-image-4.4.0-1074-aws/xenial-updates,xenial-security,now 4.4.0-1074.84 amd64 [installed,automatic]
linux-image-aws/xenial-updates,xenial-security,now 4.4.0.1074.76 amd64 [installed]
linux-libc-dev/xenial-updates,xenial-security,now 4.4.0-141.167 amd64 [installed,automatic]
locales/xenial-updates,xenial-security,now 2.23-0ubuntu10 all [installed]
login/xenial-updates,xenial-security,now 1:4.2-3.1ubuntu5.3 amd64 [installed]
logrotate/xenial-updates,now 3.8.7-2ubuntu2.16.04.2 amd64 [installed]
lsb-base/xenial-updates,now 9.20160110ubuntu0.2 all [installed]
lsb-release/xenial-updates,now 9.20160110ubuntu0.2 all [installed]
lshw/xenial-updates,now 02.17-1.1ubuntu3.5 amd64 [installed]
lsof/xenial,now 4.89+dfsg-0.1 amd64 [installed]
ltrace/xenial,now 0.7.3-5.1ubuntu4 amd64 [installed]
lvm2/xenial,now 2.02.133-1ubuntu10 amd64 [installed]
lxc-common/xenial-updates,now 2.0.8-0ubuntu1~16.04.2 amd64 [installed]
lxcfs/xenial-updates,now 2.0.8-0ubuntu1~16.04.2 amd64 [installed]
lxd/xenial-updates,now 2.0.11-0ubuntu1~16.04.4 amd64 [installed]
lxd-client/xenial-updates,now 2.0.11-0ubuntu1~16.04.4 amd64 [installed]
make/xenial,now 4.1-6 amd64 [installed,automatic]
makedev/xenial-updates,now 2.3.1-93ubuntu2~ubuntu16.04.1 all [installed]
man-db/xenial,now 2.7.5-1 amd64 [installed]
manpages/xenial,now 4.04-2 all [installed]
manpages-dev/xenial,now 4.04-2 all [installed,automatic]
mawk/xenial,now 1.3.3-17ubuntu2 amd64 [installed]
mdadm/xenial-updates,now 3.3-2ubuntu7.6 amd64 [installed]
mime-support/xenial,now 3.59ubuntu1 all [installed]
mlocate/xenial,now 0.26-1ubuntu2 amd64 [installed]
mount/xenial-updates,now 2.27.1-6ubuntu3.6 amd64 [installed]
mtr-tiny/xenial-updates,now 0.86-1ubuntu0.1 amd64 [installed]
multiarch-support/xenial-updates,xenial-security,now 2.23-0ubuntu10 amd64 [installed]
nano/xenial-updates,now 2.5.3-2ubuntu2 amd64 [installed]
ncurses-base/xenial,now 6.0+20160213-1ubuntu1 all [installed]
ncurses-bin/xenial,now 6.0+20160213-1ubuntu1 amd64 [installed]
ncurses-term/xenial,now 6.0+20160213-1ubuntu1 all [installed]
net-tools/xenial,now 1.60-26ubuntu1 amd64 [installed]
netbase/xenial,now 5.3 all [installed]
netcat-openbsd/xenial,now 1.105-7ubuntu1 amd64 [installed]
ntfs-3g/xenial-updates,xenial-security,now 1:2015.3.14AR.1-1ubuntu0.1 amd64 [installed]
open-iscsi/xenial-updates,now 2.0.873+git0.3b4b4500-14ubuntu3.7 amd64 [installed]
open-vm-tools/now 2:10.0.7-3227872-5ubuntu1~16.04.2 amd64 [installed,upgradable to: 2:10.2.0-3~ubuntu0.16.04.1]
openssh-client/xenial-updates,xenial-security,now 1:7.2p2-4ubuntu2.6 amd64 [installed]
openssh-server/xenial-updates,xenial-security,now 1:7.2p2-4ubuntu2.6 amd64 [installed]
openssh-sftp-server/xenial-updates,xenial-security,now 1:7.2p2-4ubuntu2.6 amd64 [installed]
openssl/xenial-updates,xenial-security,now 1.0.2g-1ubuntu4.14 amd64 [installed]
os-prober/xenial-updates,now 1.70ubuntu3.3 amd64 [installed,automatic]
overlayroot/xenial-updates,now 0.27ubuntu1.6 all [installed]
parted/xenial-updates,now 3.2-15ubuntu0.1 amd64 [installed]
passwd/xenial-updates,xenial-security,now 1:4.2-3.1ubuntu5.3 amd64 [installed]
pastebinit/xenial,now 1.5-1 all [installed]
patch/xenial-updates,xenial-security,now 2.7.5-1ubuntu0.16.04.1 amd64 [installed]
pciutils/xenial-updates,now 1:3.3.1-1.1ubuntu1.2 amd64 [installed]
perl/xenial-updates,xenial-security,now 5.22.1-9ubuntu0.6 amd64 [installed]
perl-base/xenial-updates,xenial-security,now 5.22.1-9ubuntu0.6 amd64 [installed]
perl-modules-5.22/xenial-updates,xenial-security,now 5.22.1-9ubuntu0.6 all [installed]
plymouth/xenial-updates,now 0.9.2-3ubuntu13.5 amd64 [installed]
plymouth-theme-ubuntu-text/xenial-updates,now 0.9.2-3ubuntu13.5 amd64 [installed]
policykit-1/xenial-updates,xenial-security,now 0.105-14.1ubuntu0.4 amd64 [installed]
pollinate/xenial-updates,now 4.33-0ubuntu1~16.04.1 all [installed]
popularity-contest/xenial,now 1.64ubuntu2 all [installed]
postgresql/xenial-updates,now 9.5+173ubuntu0.2 all [installed]
postgresql-9.5/xenial-updates,xenial-security,now 9.5.14-0ubuntu0.16.04 amd64 [installed,automatic]
postgresql-client-9.5/xenial-updates,xenial-security,now 9.5.14-0ubuntu0.16.04 amd64 [installed,automatic]
postgresql-client-common/xenial-updates,now 173ubuntu0.2 all [installed,automatic]
postgresql-common/xenial-updates,now 173ubuntu0.2 all [installed,automatic]
postgresql-contrib/xenial-updates,now 9.5+173ubuntu0.2 all [installed]
postgresql-contrib-9.5/xenial-updates,xenial-security,now 9.5.14-0ubuntu0.16.04 amd64 [installed,automatic]
powermgmt-base/xenial,now 1.31+nmu1 all [installed]
procps/xenial-updates,xenial-security,now 2:3.3.10-4ubuntu2.4 amd64 [installed]
psmisc/xenial,now 22.21-2.1build1 amd64 [installed]
python/xenial-updates,now 2.7.12-1~16.04 amd64 [installed,automatic]
python-all/xenial-updates,now 2.7.12-1~16.04 amd64 [installed,automatic]
python-all-dev/xenial-updates,now 2.7.12-1~16.04 amd64 [installed,automatic]
python-apt-common/xenial-updates,now 1.1.0~beta1ubuntu0.16.04.2 all [installed]
python-dev/xenial-updates,now 2.7.12-1~16.04 amd64 [installed]
python-minimal/xenial-updates,now 2.7.12-1~16.04 amd64 [installed,automatic]
python-pip/xenial-updates,now 8.1.1-2ubuntu0.4 all [installed]
python-pip-whl/xenial-updates,now 8.1.1-2ubuntu0.4 all [installed,automatic]
python-pkg-resources/xenial,now 20.7.0-1 all [installed,automatic]
python-setuptools/xenial,now 20.7.0-1 all [installed,automatic]
python-wheel/xenial,now 0.29.0-1 all [installed,automatic]
python2.7/xenial-updates,xenial-security,now 2.7.12-1ubuntu0~16.04.4 amd64 [installed,automatic]
python2.7-dev/xenial-updates,xenial-security,now 2.7.12-1ubuntu0~16.04.4 amd64 [installed,automatic]
python2.7-minimal/xenial-updates,xenial-security,now 2.7.12-1ubuntu0~16.04.4 amd64 [installed,automatic]
python3/xenial,now 3.5.1-3 amd64 [installed]
python3-apport/xenial-updates,xenial-security,now 2.20.1-0ubuntu2.18 all [installed]
python3-apt/xenial-updates,now 1.1.0~beta1ubuntu0.16.04.2 amd64 [installed]
python3-blinker/xenial,now 1.3.dfsg2-1build1 all [installed]
python3-cffi-backend/xenial,now 1.5.2-1ubuntu1 amd64 [installed]
python3-chardet/xenial,now 2.3.0-2 all [installed]
python3-commandnotfound/xenial-updates,now 0.3ubuntu16.04.2 all [installed]
python3-configobj/xenial,now 5.0.6-2 all [installed]
python3-cryptography/xenial-updates,xenial-security,now 1.2.3-1ubuntu0.2 amd64 [installed]
python3-dbus/xenial,now 1.2.0-3 amd64 [installed]
python3-debian/xenial,now 0.1.27ubuntu2 all [installed]
python3-distupgrade/xenial-updates,now 1:16.04.26 all [installed]
python3-gdbm/xenial,now 3.5.1-1 amd64 [installed]
python3-gi/xenial,now 3.20.0-0ubuntu1 amd64 [installed]
python3-idna/xenial,now 2.0-3 all [installed]
python3-jinja2/xenial,now 2.8-1 all [installed]
python3-json-pointer/xenial,now 1.9-3 all [installed]
python3-jsonpatch/xenial,now 1.19-3 all [installed]
python3-jwt/xenial-updates,xenial-security,now 1.3.0-1ubuntu0.1 all [installed]
python3-markupsafe/xenial,now 0.23-2build2 amd64 [installed]
python3-minimal/xenial,now 3.5.1-3 amd64 [installed]
python3-newt/xenial,now 0.52.18-1ubuntu2 amd64 [installed]
python3-oauthlib/xenial,now 1.0.3-1 all [installed]
python3-pkg-resources/xenial,now 20.7.0-1 all [installed]
python3-prettytable/xenial,now 0.7.2-3 all [installed]
python3-problem-report/xenial-updates,xenial-security,now 2.20.1-0ubuntu2.18 all [installed]
python3-pyasn1/xenial,now 0.1.9-1 all [installed]
python3-pycurl/xenial,now 7.43.0-1ubuntu1 amd64 [installed]
python3-requests/xenial-updates,xenial-security,now 2.9.1-3ubuntu0.1 all [installed]
python3-serial/xenial,now 3.0.1-1 all [installed]
python3-six/xenial,now 1.10.0-3 all [installed]
python3-software-properties/xenial-updates,now 0.96.20.8 all [installed]
python3-systemd/xenial,now 231-2build1 amd64 [installed]
python3-update-manager/xenial-updates,now 1:16.04.15 all [installed]
python3-urllib3/xenial-updates,now 1.13.1-2ubuntu0.16.04.2 all [installed]
python3-yaml/xenial,now 3.11-3build1 amd64 [installed]
python3.5/xenial-updates,xenial-security,now 3.5.2-2ubuntu0~16.04.5 amd64 [installed]
python3.5-minimal/xenial-updates,xenial-security,now 3.5.2-2ubuntu0~16.04.5 amd64 [installed]
readline-common/xenial,now 6.3-8ubuntu2 all [installed]
rename/xenial,now 0.20-4 all [installed]
resolvconf/xenial-updates,now 1.78ubuntu6 all [installed]
rsync/xenial-updates,xenial-security,now 3.1.1-3ubuntu1.2 amd64 [installed]
rsyslog/xenial,now 8.16.0-1ubuntu3 amd64 [installed]
run-one/xenial,now 1.17-0ubuntu1 all [installed]
screen/xenial,now 4.3.1-2build1 amd64 [installed]
sed/xenial,now 4.2.2-7 amd64 [installed]
sensible-utils/xenial-updates,xenial-security,now 0.0.9ubuntu0.16.04.1 all [installed]
sgml-base/xenial,now 1.26+nmu4ubuntu1 all [installed]
shared-mime-info/xenial-updates,now 1.5-2ubuntu0.2 amd64 [installed]
snapd/xenial-updates,now 2.34.2 amd64 [installed]
software-properties-common/xenial-updates,now 0.96.20.8 all [installed]
sosreport/xenial-updates,now 3.6-1ubuntu0.16.04.2 amd64 [installed]
squashfs-tools/xenial-updates,now 1:4.3-3ubuntu2.16.04.3 amd64 [installed]
ssh-import-id/xenial,now 5.5-0ubuntu1 all [installed]
ssl-cert/xenial,now 1.0.37 all [installed,automatic]
strace/xenial,now 4.11-1ubuntu3 amd64 [installed]
sudo/xenial-updates,now 1.8.16-0ubuntu1.5 amd64 [installed]
sysstat/xenial-updates,now 11.2.0-1ubuntu0.2 amd64 [installed,automatic]
systemd/xenial-updates,xenial-security,now 229-4ubuntu21.15 amd64 [installed]
systemd-sysv/xenial-updates,xenial-security,now 229-4ubuntu21.15 amd64 [installed]
sysv-rc/xenial,now 2.88dsf-59.3ubuntu2 all [installed]
sysvinit-utils/xenial,now 2.88dsf-59.3ubuntu2 amd64 [installed]
tar/xenial-updates,xenial-security,now 1.28-2.1ubuntu0.1 amd64 [installed]
tcpd/xenial,now 7.6.q-25 amd64 [installed]
tcpdump/xenial-updates,xenial-security,now 4.9.2-0ubuntu0.16.04.1 amd64 [installed]
telnet/xenial,now 0.17-40 amd64 [installed]
time/xenial,now 1.7-25.1 amd64 [installed]
tmux/xenial,now 2.1-3build1 amd64 [installed]
tzdata/xenial-updates,xenial-security,now 2018i-0ubuntu0.16.04 all [installed]
ubuntu-cloudimage-keyring/xenial,now 2013.11.11 all [installed]
ubuntu-core-launcher/xenial-updates,now 2.34.2 amd64 [installed]
ubuntu-keyring/xenial,now 2012.05.19 all [installed]
ubuntu-minimal/now 1.361.1 amd64 [installed,upgradable to: 1.361.2]
ubuntu-release-upgrader-core/xenial-updates,now 1:16.04.26 all [installed]
ubuntu-server/xenial-updates,now 1.361.2 amd64 [installed]
ubuntu-standard/xenial-updates,now 1.361.2 amd64 [installed]
ucf/xenial,now 3.0036 all [installed]
udev/xenial-updates,xenial-security,now 229-4ubuntu21.15 amd64 [installed]
ufw/xenial,now 0.35-0ubuntu2 all [installed]
uidmap/xenial-updates,xenial-security,now 1:4.2-3.1ubuntu5.3 amd64 [installed]
unattended-upgrades/xenial-updates,xenial-security,now 0.90ubuntu0.10 all [installed]
update-manager-core/xenial-updates,now 1:16.04.15 all [installed]
update-notifier-common/xenial-updates,now 3.168.10 all [installed]
ureadahead/xenial,now 0.100.0-19 amd64 [installed]
usbutils/xenial,now 1:007-4 amd64 [installed]
util-linux/xenial-updates,now 2.27.1-6ubuntu3.6 amd64 [installed]
uuid-runtime/xenial-updates,now 2.27.1-6ubuntu3.6 amd64 [installed]
vim/xenial-updates,xenial-security,now 2:7.4.1689-3ubuntu1.2 amd64 [installed]
vim-common/xenial-updates,xenial-security,now 2:7.4.1689-3ubuntu1.2 amd64 [installed]
vim-runtime/xenial-updates,xenial-security,now 2:7.4.1689-3ubuntu1.2 all [installed]
vim-tiny/xenial-updates,xenial-security,now 2:7.4.1689-3ubuntu1.2 amd64 [installed]
vlan/xenial-updates,now 1.9-3.2ubuntu1.16.04.5 amd64 [installed]
wget/xenial-updates,xenial-security,now 1.17.1-1ubuntu1.4 amd64 [installed]
whiptail/xenial,now 0.52.18-1ubuntu2 amd64 [installed]
xauth/xenial,now 1:1.0.9-1ubuntu2 amd64 [installed]
xdg-user-dirs/xenial-updates,now 0.15-2ubuntu6.16.04.1 amd64 [installed]
xfsprogs/xenial-updates,now 4.3.0+nmu1ubuntu1.1 amd64 [installed]
xkb-data/xenial,now 2.16-1ubuntu1 all [installed]
xml-core/xenial,now 0.13+nmu2 all [installed]
xz-utils/xenial,now 5.1.1alpha+20120614-2ubuntu2 amd64 [installed]
zerofree/xenial,now 1.0.3-1 amd64 [installed]
zlib1g/xenial-updates,now 1:1.2.8.dfsg-2ubuntu4.1 amd64 [installed]
zlib1g-dev/xenial-updates,now 1:1.2.8.dfsg-2ubuntu4.1 amd64 [installed,automatic]

* for updates also do sudo apt-get dist-upgrade (it will only do ht packages not the OS upgrade)
