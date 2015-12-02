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
  * [gcoinstsrv02](servers/gcooinstsrv02/) windows vm with vendor software



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

## Other servers also in Spiddal:
  * Asus2 Laptop
  * NMS Server
  * Hypervisor gco1
  * Hypervisor gco2

## Other servers also in Oranmore:
  * Hypervisor 172.17.1.84

## Other servers:
  * spidvid.cloudapp.net nginx video caching vm in azure.

# Controlled Restart
This section captures the steps to shutdown and restart the
complete system of instruments and virtual machines. Note that if physical
servers (the NMS, hypervisors, spidvid) are stopped then manual intervention
may be required to restart those. That scenario is not considered here.
## Controlled Shutdown

After using the pnc client to stop all the devices, do a soft shutdown of vms in the following order:
  * [gcoinstsrv01](servers/gcoinstsrv01/)
  * [gcoinstsrv02](servers/gcooinstsrv02/)
  * [gconode01](servers/gconode01/)
  * [kafka01](servers/kafka01/)
  * [kafka02](servers/kafka02/)
  * [kafka03](servers/kafka03/)
  * [data01](servers/data01/)
  * [data02](servers/data02/)
  * [data03](servers/data03/)
  * [cluster05](servers/cluster05)
  * [cluster04](servers/cluster04)
  * [cluster03](servers/cluster03)
  * [cluster02](servers/cluster02)
  * [cluster01](servers/cluster01)
  * [dockerub](servers/dockerub/)


## Controlled Startup
Start the vm's in the following order, then use the pnc client to restart the devices
  * [dockerub](servers/dockerub/)
  * [gcoinstsrv01](servers/gcoinstsrv01/)
  * [gcoinstsrv02](servers/gcooinstsrv02/)
  * [gconode01](servers/gconode01/)
  (devices could be restarted at any point from now)
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

### Post Startup Tasks

  * On [gcoinstsrv01](servers/gcoinstsrv01/) stop the ctd  data feed, update
    the timestamp, then restart data feed. (TODO: more detail here)
  * On [gcoinstsrv01](servers/gcoinstsrv01/) in chrome, reset the hydrophone
    timestamp. (TODO: more detail here)
  * Adjust the video camera (see below)

# Adjusting the Video Camera

  * Remote desktop to Asus laptop 172.16.255.16
  * From there remote desktop to the NMS 172.16.255.15
  * Start the Kongsberg software using the “Kconbsberg Camera Control” shortcut on Desktop
  * Start VLC media player using “VLC Spiddal Video” shortcut on Desktop. (Note it takes a couple of seconds to show the video)
  * The Kongsberg software can now be used to control the video, which will show adjust with about a 2 second delay.  At the moment we are using an upward looking position currently stored at “Preset 7”.
  * If the picture remains black or very dark (possibly due to biofouling), it may be necessary to change the exposure from automatic to shutter, and slow the shutter speed until picture becomes visible.
  * If all else fails set auto focus and try various presets.

