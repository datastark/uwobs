# uwobs
Underwater Observatory System

This public repository contains documentation, code and system configuration
for the data collection and processing system at <a href="spiddal.marine.ie">
spiddal.marine.ie</a>.

The target audience is uwobs system adminstrators.

# Server Configurations

This uwobs documentation is maintained on each of the servers, with a script
provided to backup or restore configurations. (note proxy server is different
on spiddal hosts)

    cd ~/dev/uwobs
    https_proxy=10.0.5.55:80 git pull

To add/remove configuration files to be use add it to files.txt and use
[backup_config.sh](bin/backup_config.sh). <b>Be careful not to commit any
secrets to public git archive.</b> 

    vi servers/$(hostname)/files.txt

To backup some the configuration to git:

    cd ~/dev/uwobs
    https_proxy=10.0.5.55:80 git pull
    bin/backup_config.sh
    git commit -a -m 'latest configuration'
    https_proxy=10.0.5.55:80 git push

To restore some configuration file from git use [install_file.sh](bin/install_file.sh). For example, to recover haproxy configuration:

    sudo bin/install_file.sh /etc/haproxy/haproxy.cfg
    sudo service haproxy reload

# Shared Configurations
haproxy configuration is shared across cluster01-05. The following steps can be used to change the configuration on cluster01 and copy it to the other nodes.

    sudo vi /etc/haproxy/haproxy.cfg
    sudo service haproxy reload
    bin/backup_config.sh
    for server in 02 03 04 05; do cp servers/cluster01/files/etc/haproxy/haproxy.cfg servers/cluster${server}/files/etc/haproxy/haproxy.cfg; done
    git commit -a -m 'latest configuration'
    https_proxy=10.0.5.55:80 git push
    for item in 2 3 4 5 ; do ssh -t cluster0$item "cd dev/uwobs && https_proxy=10.0.5.55:80 git pull && sudo bin/install_file.sh /etc/haproxy/haproxy.cfg && sudo service haproxy reload" ; done



# General Overview
The system runs across two data centers, the shore station in Spiddal and the
main data center in Oranmore, having network connectivity between the zones.

Instruments are connected to the shore station in Spiddal where we have
4 uwobs primary servers:

## Servers in Spiddal
  * [spidvid](servers/spidvid/) a physical server with video card
  * [gconode01](servers/gconode01/) ubuntu vm running data collectors
  * [gcoinstsrv01](servers/gcoinstsrv01/) windows vm with vendor software
  * [gcoinstsrv02](servers/gcoinstsrv02/) windows vm with vendor software

## Servers in Oranmore:
  * [dockerub](servers/dockerub/) nginx server
  * [kafka01](servers/kafka01/) kafka server
  * [kafka02](servers/kafka02/) kafka server
  * [kafka03](servers/kafka03/) kafka server
  * [data01](servers/data01/) cassandra, elasticsearch server
  * [data02](servers/data02/) cassandra, elasticsearch server
  * [data03](servers/data03/) cassandra, elasticsearch server
  * [cluster01](servers/cluster01) haproxy, applications server
  * [cluster02](servers/cluster02) haproxy, applications server
  * [cluster03](servers/cluster03) haproxy, applications server
  * [cluster04](servers/cluster04) haproxy, applications server
  * [cluster05](servers/cluster05) haproxy, applications server

## Other servers also in Spiddal
There are a number of other servers in Spiddal, see spreadsheet from Damian for full list.
The following are referenced in these documents:
  * Asus2 Laptop 172.16.255.16
  * NMS Server 172.16.255.15
  * Hypervisor gco1 172.16.255.230

## Other servers also in Oranmore:
  * Hypervisor dmzdmvhost 172.17.1.84

## Other servers:
  * spidvid.cloudapp.net nginx video caching vm in azure.

# Controlled Restart
This section captures the steps to shutdown and restart the
complete system of instruments and virtual machines. Note that if physical
servers (the NMS, hypervisors, spidvid) are stopped then manual intervention
may be required to restart those. That scenario is not considered here.

## Controlled Shutdown

### 1. Turn off the underwater devices
Remote desktop to the NMS Server, and use the espy client to stop all the devices. 
    * Logon to espy as "System Administrator". If user is already connected from another desktop
you will need to connect to that other desktop.
    * Hebida

### 2. Turn off Spiddal virtual machines
Shut down the windows and linux vms by connecting to them and issuing the appropriate commands.
  * [gcoinstsrv01](servers/gcoinstsrv01/)
  * [gcoinstsrv02](servers/gcooinstsrv02/)
  * [gconode01](servers/gconode01/)

### 3. Turn off Oranmore virtual machines
From a shell on [cluster01](servers/cluster01) you can easily shutdown the servers with the following
command which will prompt for passwords if required.

    for server in kafka01 kafka02 kafka03 data01 data02 data03 cluster05 cluster04 cluster03 cluster02
    do echo $server && ssh -t $server 'sudo shutdown -h now'
    done
    sudo shutdown -h now

### 4. Turn off the web server [dockerub](servers/dockerub/)
<b>NB:</b> Note this vm is in a main dmz hypervisor controlled by operations. When power cycling [dockerub](servers/dockerub/) use `shutdown -r now` unless Keith or Colin are available to restart that vm from the hypervisor.

## Controlled Startup
### 1. Start the dockerub server first so the web interface becomes available.
  * [dockerub](servers/dockerub/)

### 2. Turn on the underwater devices
Remote desktop to the NMS Server, and logon to the Espy Client as user System Administrator. Navigate the menu:
  * Object Administrator
    * Managed Elements Catalog
      * Galway PNC Node T...
        * CEE1-NODE-1`
          * (Configuration Tab)

Power on the following science ports:
  * 1 (Hydrophone)
  * 6 (CTD)
  * 7 (Turb/Fluor)
  * 10 (Vemco)
  * 12 (ADCP)
  * 18 (HDTV)

### 3. Turn on Spiddal virtual machines
The Spiddal VM's can be started from hypervisor gco1 172.16.255.230
  * [gcoinstsrv01](servers/gcoinstsrv01/)
  * [gcoinstsrv02](servers/gcooinstsrv02/)
  * [gconode01](servers/gconode01/)

### 4. Adjust time on Devices
Remote desktop to [gcoinstsrv01](servers/gcoinstsrv01/)
  * Start the Idronaut Terminal software and connect to com 7 at 9600 baud. Stop the feed using `ctrl-c`,
then use the menu options to set the date and time. Verify, then restart streaming.
  * Use Chrome to connect to the [hydrophone](http://172.16.255.253/operations.html) using the bookmarked link. The device may
have continued running on battery and have the correct time, otherwise use the web interface to reset the time.

### 5. Turn on Oranmore virtual machines
Now the Spiddal VM's can be started from hypervisor dmzdmvhost 172.17.1.84
  * [cluster01](servers/cluster01)
  * [cluster02](servers/cluster02)
  * [cluster03](servers/cluster03)
  * [cluster04](servers/cluster04)
  * [cluster05](servers/cluster05)
  * [kafka01](servers/kafka01/)
  * [kafka02](servers/kafka02/)
  * [kafka03](servers/kafka03/)
  * [data01](servers/data01/)
  * [data02](servers/data02/)
  * [data03](servers/data03/)

### 6. Adjust the video
Remote desktop to the NMS Server 172.16.255.15
  * Start the Kongsberg software using the “Kconbsberg Camera Control” shortcut on Desktop
  * Start VLC media player using “VLC Spiddal Video” shortcut on Desktop. (Note it takes a couple of seconds to show the video)
  * The Kongsberg software can now be used to control the video, which will show adjust with about a 2 second delay.  At the moment we are using an upward looking position currently stored at “Preset 7” (Preset 3 during biofoul episode Nov 2015).
  * If the picture remains black or very dark (possibly due to biofouling), it may be necessary to change the exposure from automatic to shutter, and slow the shutter speed until picture becomes visible.
  * If all else fails set auto focus and try various presets or resort to manually searching for something visible.

### 7. Restart some added services
  * On [cluster01](servers/cluster01) `docker start erddap`
  * On [cluster03](servers/cluster03) `docker start geonetwork_postgis` then after 10 seconds or so `docker start geonetwork`
