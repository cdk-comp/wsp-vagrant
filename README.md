vagrant-serverpilot
========================

A lemp stack with Serverpilot, Ubuntu 16.04, vagrant, nginx, apache, php-5-7, php-fpm, mysql 5.7, git and composer.

Install
=======

1. copy `wsp-conf.yaml.example` to `wsp-conf.yaml`
 ```bash
 $ mv wsp-conf.yaml.example wsp-conf.yaml
 ```
 - Change ip
 - Change max RAM memory
 - Change max CPU's
 - Change hostname/servername
 - Set serverpilot cliend id
 - Set serverpilot api key
 - Change the ssh keys path
 - Set aliases
2. choose your virtualization product
 - install virtualbox >= 5.1.12
 - install parallels 10
3. install vagrant 1.8.7 (there is a bug with 1.9 and 1.9.1 regarding RedHat based systems and networking)
4. install the necessary plugins for vagrant, if not yet happened
 ```bash
 $ vagrant plugin install vagrant-hostmanager
 $ vagrant plugin install vagrant-cachier
 $ vagrant plugin install vagrant-winnfsd # only for Windows
 ```

 Hostmanager is needed to add/remove entries in your local /etc/hosts file. To support development domains
 Cachier is needed to prevent downloading rpm´s again. This is usefull during setting up a vm, when you have online internet  via cellphone like inside a train :-)
 
 If you're using parallels you also have to install the vagrant plugin
 ```bash
 $ vagrant plugin install vagrant-parallels
 ```

4. start vagrant with virtual box
 ```bash
 $ vagrant up
 ```
 or with parallels
 ```bash
 $ vagrant up --provider=parallels
 ```
 
 5. After installation Mount sync folders by changing of flag true to false:
 
 ```ruby
# unmounted
config.vm.synced_folder folder["map"], folder["to"], type: "nfs", owner: "serverpilot", group: "serverpilot", disabled: true
# mounted
config.vm.synced_folder folder["map"], folder["to"], type: "nfs", owner: "serverpilot", group: "serverpilot", disabled: false
 ```

Config Option
=============

You can setup dedicated virtual hosts, sync folders, VM hardware in 

```
wsp-conf.yaml
```

How it works
============

if you call http://project.test it will search for a index.php inside the app/public folder. It is really easy to start with any application.

Special
==========
Switch to the `serverpilot` user

```bash
$ sudo -i
$ su serverpilot
$ cd ~/apps/APPNAME

```

TODO
==========
- [ ] Admin app - for custom app installation/removing
- [ ] Make app with git repo on install and another features by wizard
- [ ] SSH access to serverpilot:serverpilot after serverpilot installation
- [X] Nice link for the sh scripts
- [ ] Mount sync folders automatically after the first reload

Change Log
==========
