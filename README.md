# Running Node JS with Systemd and SELinux
Files to setup Node JS process running on Systemd and SELinux

## Why
Running Node via Systemd is great way to automatically restart on fails, capture logs and secure your app. SELinux adds an additional layer of security on top of Linux. It is very important if you want to keep your app secure. Read more about [SELinux here](https://www.digitalocean.com/community/tutorials/an-introduction-to-selinux-on-centos-7-part-1-basic-concepts).

There are various sites that show how to setup Systemd with Node, but I haven't found a single tutorial on using Node with SELinux. Definitely no example files. Most people just say disable it, which is **a very bad idea**. SELinux is very important to securing an application.

### Assumptions
##### These files are based the entry to the node js application living in `/var/www/node/process.js`. 
That file name can be changed via find and replace, but it must be in `/var/www/*` for the files to work. I will make a script to make it easier
##### Must use `#!/usr/bin/env node` at the top of JS file and be executable
DO NOT change the nodejs.service file to execute `node file.js`. The JS needs to be standalone executable
##### The SELinux module assumes that the Node server listens on port 3000
Read below on how to allow other ports

### How to use
The SELinux folder has the files to create an SELinux policy for Node. Running the `nodejs.sh` file will build the module and install it on your sytem.

The Systemd folder contains a service file. If you put the file in `/etc/systemd/system` folder, you will be able to run `systemctl` command to start, stop and restart it.
```
systemctl start nodejs
```
or
```
systemctl stop nodejs
```
All logs are in /var/log/messages. Check there if you get any errors.

### Customizing
In addition to changing file locations, your app may need other permissions. The provided files are just the base. It is recommended that you put SELinux in permissive mode to start
```shell
semanage permissive -a nodejs_t
```
This will allow your app to run, but will log all permission errors to `/var/log/audit/audit.log`. There is a great SELinux tool, `audit2allow`,  to aggregate those errors give you the correct lines for a policy.
```shell
audit2allow -a | grep node
```
Running that command will output statements like the following
```
#============= nodejs_t ==============
allow nodejs_t http_cache_port_t:tcp_socket name_bind;
```
Place the allow statements in the nodejs.te file and rerun nodejs.sh. When you are done turn permissive mode off
```
semanage permissive -d nodejs_t
```
