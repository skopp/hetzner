Hosting setup for a Hetzner Root-Server (virtualization with KVM)
=================================================================

[Hetzner Online AG](http://www.hetzner.de) is in my opinion a very good web hosting provider.
I do not work for Hetzner. I accept no responsibility for possible errors in the script and
instructions (The risk is yours!). The reason for using server virtualization was to have multiple
independent systems for development, test, staging and production.

What do you need from Hetzner?

- Root Server _(49€ per month for an [EX4](http://www.hetzner.de/hosting/produktmatrix/rootserver-produktmatrix-ex) with 16GB RAM & 2x3TB HDD or 59€ per month for an [EX4S](http://www.hetzner.de/hosting/produktmatrix/rootserver-produktmatrix-ex) with 32GB RAM)_
- Flexi Pack is necessary to order IP subnet. _(15€ per month)_
- Subnet /29 _(5.40€ per month)_ with the following setup you can start max 5 VMs.
- Subnet /28 _(11.20€ per month)_ with the following setup you can start max 13 VMs.

The price of the setup is awesome and you get on top 100GB backup space (SFTP). (Prices from June 2012)

Installation and configuration
------------------------------

Hetzner example configuration:

* **OS:** Ubuntu 12.04 LTS
* **Main Server IP:** 177.10.0.8

* **Subnet:** 79.48.232.8 /29
    * **Maske:** 255.255.255.248
    * **Broadcast:** 79.48.232.15
    * **Available IP addresses:** 79.48.232.9 upto 79.48.232.14

### Install steps for host system:
##### 1. Push your public ssh-key to the server and login over SSH
Copy public ssh-key with ssh-copy-id to the remote server. Please install [homebrew](https://github.com/mxcl/homebrew/wiki/installation) before on your Mac. _(Mac only)_

    brew install ssh-copy-id
    ssh-copy-id -i ~/.ssh/sp-admin.pub root@177.10.0.8

The identity filename can be different on your system. If you are not a Mac user, you should add your public ssh-key to ``~/.ssh/authorized_keys`` on the remote server.

Now you can login over SSH without password:

    ssh root@177.10.0.8 -A

Update package informations and install make

    apt-get update && apt-get install make
    
##### 2. I recommend to run the installation in a [screen session](http://de.wikipedia.org/wiki/GNU_Screen), because it will take at least 15 minutes.

    apt-get update && apt-get -y install screen

* ``screen``: Start a new screen session. Detach the session with ``ctrl+a d``
* ``screen -r``: Reattach to a detached screen process.

##### 3. Download and extract the setup script

    cd /root && wget http://www.spider-network.net/downloads/hetzner-host.tar.gz && tar xvf hetzner-host.tar.gz

##### 4. Start the installation

    cd /root/hetzner/host/install && make install

##### 5. Adapt installation configuration

    cd /root/hetzner/host/install && cp config/node.EXAMPLE.json config/node.server-001.json
    vi /root/hetzner/host/install/config/node.server-001.json

[See example config](host/install/config/node.EXAMPLE.json)


##### 6. Change server network configuration and setup munin monitoring tool

    cd /root/hetzner/host/install/chef && chef-solo -c solo.rb -j ../config/node.server-001.json

### Create and manage VM's:

- Create a new VM (It can take up to 20 minutes. I recommend to do this inside a [screen session](http://de.wikipedia.org/wiki/GNU_Screen) (see above).)

    ``thor hetzner:host:vm:create --ip=79.48.232.9 --name=vm-001 --user-pass=password``

    - Options:
    <pre>
    :name      => :required, # Name of the VM. e.g. vm-001
    :user_name => 'server',  # Linux user (Default: server)
    :user_pass => :required, # Password for the Linux user
    :ip        => :required, # VM IP, one of the available IP's from your subnet. e.g. 79.48.232.9
    :cpus      => 4,         # VM CPU cores (Default: 4)
    :ram       => 4096,      # VM RAM (Default: 4096 MB)
    :swap      => 1024,      # VM Swap (Default: 1024 MB)
    :hdd       => 20480      # VM HDD (Default: 20480 MB)
    </pre>

    - Increase IO performance (add: ``cache='writeback'``):

        ``thor hetzner:host:vm:edit --name=vm-001``

        ```xml
        <disk type='file' device='disk'>
          <driver name='qemu' type='qcow2' cache='writeback'/>
          <source file='/root/vms/vm-002/tmphJhs3T.qcow2'/>
          <target dev='hda' bus='ide'/>
        </disk>
        ```
        Don't forget to restart the VM.

- Show all VM's and the status (running, shut off)

    ``thor hetzner:host:vm:list``

- Stop the given VM

    ``thor hetzner:host:vm:stop --name=vm-001``

- Start the given VM

    ``thor hetzner:host:vm:start --name=vm-001``

- Create a backup (dump)

    ``thor hetzner:host:vm:backup --name=vm-001``

- Show all backups from the given VM

    ``thor hetzner:host:vm:backups --name=vm-001``

- Restore the VM from backup dump

    ``thor hetzner:host:vm:stop --name=vm-001``

    ``thor hetzner:host:vm:restore --file=vm_backup_20111012-1247 --name=vm-001``

Contributing and Support
------------------------
If you have any suggestions or criticism write me an e-mail [michael.voigt@spider-network.net](mailto:michael.voigt@spider-network.net)
or create an issue. If you need help just contact me. I will always support the latest Ubuntu LTS ("Long Term Support")
version, which is currently "Ubuntu 12.04 LTS".
