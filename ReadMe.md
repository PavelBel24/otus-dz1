# Домашнее задание "Vagrant-стенд для обновления ядра и создания образа системы"

## 1. Vagrant-стенд для обновления ядра и обновление ядра.

На основе vagrant box generic/centos8s cоздана VM с параметрами указанными в методичке.
  
  С помощью параметра provision были выполнены команды 
   - по подключению репозитория [elrepo-kernel](https://www.elrepo.org)
   - установки последнего ядра из репозитория elrepo-kernel
   - обновилена конфигурация загрузчика
   - выбрана загрузка с конфигурации с новым ядра по-умолчанию.

``` Vagrantfile
...
config.vm.provision "shell", inline: <<-SHELL
     uname -r
     sudo yum install -y https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm
     sudo yum --enablerepo elrepo-kernel install kernel-ml -y
     sudo grub2-mkconfig -o /boot/grub2/grub.cfg
     sudo grub2-set-default 0
     sudo reboot now
  SHELL
```

Полный листинг Vagrantfile см. [здесь](Vagrantfile)

Выдержки из протокола выполнения команд

```
admuser@vm1:~/otus-dz1$ vagrant up
Bringing machine 'kernel-update' up with 'virtualbox' provider...
==> kernel-update: Importing base box 'generic/centos8s'...
==> kernel-update: Matching MAC address for NAT networking...
==> kernel-update: Checking if box 'generic/centos8s' version '4.3.4' is up to date...
==> kernel-update: Setting the name of the VM: otus-dz1_kernel-update_1705148590991_9542
==> kernel-update: Clearing any previously set network interfaces...
==> kernel-update: Preparing network interfaces based on configuration...
    kernel-update: Adapter 1: nat
==> kernel-update: Forwarding ports...
    kernel-update: 22 (guest) => 2222 (host) (adapter 1)
==> kernel-update: Running 'pre-boot' VM customizations...
==> kernel-update: Booting VM...
==> kernel-update: Waiting for machine to boot. This may take a few minutes...
    kernel-update: SSH address: 127.0.0.1:2222
    kernel-update: SSH username: vagrant
    kernel-update: SSH auth method: private key
    kernel-update: 
    kernel-update: Vagrant insecure key detected. Vagrant will automatically replace
    kernel-update: this with a newly generated keypair for better security.
    kernel-update: 
    kernel-update: Inserting generated public key within guest...
    kernel-update: Removing insecure key from the guest if it's present...
    kernel-update: Key inserted! Disconnecting and reconnecting using new SSH key...
==> kernel-update: Machine booted and ready!
==> kernel-update: Checking for guest additions in VM...
    kernel-update: The guest additions on this VM do not match the installed version of
    kernel-update: VirtualBox! In most cases this is fine, but in rare cases it can
    kernel-update: prevent things such as shared folders from working properly. If you see
    kernel-update: shared folder errors, please make sure the guest additions within the
    kernel-update: virtual machine match the version of VirtualBox you have installed on
    kernel-update: your host and reload your VM.
    kernel-update: 
    kernel-update: Guest Additions Version: 6.1.30
    kernel-update: VirtualBox Version: 7.0
==> kernel-update: Setting hostname...
==> kernel-update: Running provisioner: shell...
    kernel-update: Running: inline script
    kernel-update: 4.18.0-516.el8.x86_64
    kernel-update: CentOS Stream 8 - AppStream                     881 kB/s |  35 MB     00:40
    kernel-update: CentOS Stream 8 - BaseOS                        1.4 MB/s |  58 MB     00:40
    kernel-update: CentOS Stream 8 - Extras                        5.3 kB/s |  18 kB     00:03
    kernel-update: CentOS Stream 8 - Extras common packages        940  B/s | 6.9 kB     00:07
    kernel-update: Extra Packages for Enterprise Linux 8 - x86_64  944 kB/s |  16 MB     00:17

...

kernel-update: 
    kernel-update:   Running scriptlet: kernel-ml-6.6.11-1.el8.elrepo.x86_64                   3/3
    kernel-update:   Verifying        : kernel-ml-6.6.11-1.el8.elrepo.x86_64                   1/3
    kernel-update:   Verifying        : kernel-ml-core-6.6.11-1.el8.elrepo.x86_64              2/3
    kernel-update:   Verifying        : kernel-ml-modules-6.6.11-1.el8.elrepo.x86_64           3/3
    kernel-update: 
    kernel-update: Installed:
    kernel-update:   kernel-ml-6.6.11-1.el8.elrepo.x86_64
    kernel-update:   kernel-ml-core-6.6.11-1.el8.elrepo.x86_64
    kernel-update:   kernel-ml-modules-6.6.11-1.el8.elrepo.x86_64
    kernel-update: 
    kernel-update: Complete!
    kernel-update: Generating grub configuration file ...
    kernel-update: done
admuser@vm1:~/otus-dz1$ vagrant ssh
[vagrant@kernel-update ~]$ uname -r
6.6.11-1.el8.elrepo.x86_64
```
Полный протокол выполлнения команд находиться в файле [protokol.log](protokol.log) 
