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



## Steps Followed

1) Planned the hosting by estimating users, cost and resources required for this project

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
13) Verify that two keys are made. The one with .pub extension is a public key. The other is a private key. We already know who this works from classes. Hence we cat and read the public key.

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

### Thanks and acknowledgements
Thanks to Udacity staff, Udacity Instrutors and Udacity Mentors for helping out.
Thanks to Knowledge hub, and class room chat, I will be closing my open questions shortly on all of them. Thanks to stackoverflow, a ready reference.
I used various books, sites, youtube videos for references, thanks to them all. Thanks to github of iliketomatoes, stueken and chuanqin too. I will be adding more to the list as i improve on this time to time.
Thanks for reading my readme.md.
