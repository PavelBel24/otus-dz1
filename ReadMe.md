# Домашнее задание "Vagrant-стенд для обновления ядра и создания образа системы"

## 1. Vagrant-стенд для обновления ядра и обновление ядра.

На основе vagrant box generic/centos8s cоздана VM с параметрами указанными в методических рекомендациях.
  
  С помощью параметра provision были выполнены команды:
   - по получению информации об установленном ядре;
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


После перезагрузки VM в терминале подключаемся к VM с использованием команды **vagrant ssh**. 

Затем запраштваем информацию об ядре командой **uname -r**. Результат вывода команды см.ниже
```
admuser@vm1:~/otus-dz1$ vagrant ssh
[vagrant@kernel-update ~]$ uname -r
6.6.11-1.el8.elrepo.x86_64

```
Полный листинг Vagrantfile см. [здесь](Vagrantfile)

Выдержки из протокола выполнения команд из **Vagrantfile**.

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
...
    kernel-update: Removing insecure key from the guest if it's present...
    kernel-update: Key inserted! Disconnecting and reconnecting using new SSH key...
==> kernel-update: Machine booted and ready!
==> kernel-update: Checking for guest additions in VM...
    kernel-update: The guest additions on this VM do not match the installed version of
...
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

----

## 2. Создание образа системы.

Основные шаги создания образа ОС виртуальной машины:
1. Скачивание ISO-образ дистрибутива и сверяем его контрольную сумму.
2. Создание ВМ. Установка ОС ВМ при помощи Packer.
3. Настройка ВМ при помощи скриптов настройки.
4. Снятия с установленной ВМ образа при помощи Packer.

Сценарий описан в файле centos.json

Вспомогательные файлы:

в каталоге **http** расположен файл **vagrant.ks**, вкотором описаны параметры для процесса автоматической установки ОС.

в каталоге **script** расположены файлы

**stage-1-kernel-update.sh** - отвечающий за обновление ядра из репозитория elrepo-kernel

**stage-2-clean.sh**  - отвечающий за удалением временных файлов и ненужных пакетов, добавлен SSH ключь для пользователя vagrant 

Файлы получены из репозитория [https://github.com/Nickmob/vagrant_kernel_update.git](https://github.com/Nickmob/vagrant_kernel_update.git)

Результат выполненния команды - файл **centos-8-kernel-6-x86_64-Minimal.box**.

```BASH
admuser@vm1:~/otus-dz1/packer$ packer build centos.json
virtualbox-iso.centos-8stream: output will be in this color.

==> virtualbox-iso.centos-8stream: Cannot find "Default Guest Additions ISO" in vboxmanage output (or it is empty)
==> virtualbox-iso.centos-8stream: Retrieving Guest additions checksums
==> virtualbox-iso.centos-8stream: Trying https://download.virtualbox.org/virtualbox/7.0.10/SHA256SUMS
==> virtualbox-iso.centos-8stream: Trying https://download.virtualbox.org/virtualbox/7.0.10/SHA256SUMS
==> virtualbox-iso.centos-8stream: https://download.virtualbox.org/virtualbox/7.0.10/SHA256SUMS => /home/admuser/.cache/packer/4ef3e0da8edac785d8f4c608e54f8b7b25836495
...
==> virtualbox-iso.centos-8stream: Exporting virtual machine...
    virtualbox-iso.centos-8stream: Executing: export packer-centos-vm --output builds/packer-centos-vm.ovf --manifest --vsys 0 --description CentOS 8 Stream with kernel 6.x --version 8
==> virtualbox-iso.centos-8stream: Cleaning up floppy disk...
==> virtualbox-iso.centos-8stream: Deregistering and deleting VM...
==> virtualbox-iso.centos-8stream: Running post-processor: vagrant
==> virtualbox-iso.centos-8stream (vagrant): Creating a dummy Vagrant box to ensure the host system can create one correctly
==> virtualbox-iso.centos-8stream (vagrant): Creating Vagrant box for 'virtualbox' provider
    virtualbox-iso.centos-8stream (vagrant): Copying from artifact: builds/packer-centos-vm-disk001.vmdk
    virtualbox-iso.centos-8stream (vagrant): Copying from artifact: builds/packer-centos-vm.mf
    virtualbox-iso.centos-8stream (vagrant): Copying from artifact: builds/packer-centos-vm.ovf
    virtualbox-iso.centos-8stream (vagrant): Renaming the OVF to box.ovf...
    virtualbox-iso.centos-8stream (vagrant): Compressing: Vagrantfile
    virtualbox-iso.centos-8stream (vagrant): Compressing: box.ovf
    virtualbox-iso.centos-8stream (vagrant): Compressing: metadata.json
    virtualbox-iso.centos-8stream (vagrant): Compressing: packer-centos-vm-disk001.vmdk
    virtualbox-iso.centos-8stream (vagrant): Compressing: packer-centos-vm.mf
Build 'virtualbox-iso.centos-8stream' finished after 1 hour 35 minutes.

==> Wait completed after 1 hour 35 minutes

==> Builds finished. The artifacts of successful builds are
```

Для проверки  импортируем полученный vagrant box в Vagrant: 

```
dmuser@vm1:~/otus-dz1/packer$ vagrant box list
generic/centos8s (virtualbox, 4.3.4)
admuser@vm1:~/otus-dz1/packer$ vagrant box add --name centos8-kernel6 centos-8-kernel-6-x86_64-Minimal.box
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'centos8-kernel6' (v0) for provider: 
    box: Unpacking necessary files from: file:///home/admuser/otus-dz1/packer/centos-8-kernel-6-x86_64-Minimal.box
==> box: Successfully added box 'centos8-kernel6' (v0) for 'virtualbox'!

```
Загрузка на Vagrant Cloud

На данный момент загрузка завершается с ошибкой

```
vagrant cloud publish --release vagrant init PavelBel24/centos8-kernel6 centos-8-kernel-6-x86_64-Minimal.box
You are about to publish a box on Vagrant Cloud with the following options:
vagrant/:   (vinit) for provider 'PavelBel24/centos8-kernel6'
Automatic Release:     true
Do you wish to continue? [y/N]Y
Saving box information...
Failed to create box vagrant/
Vagrant Cloud request failed - Resource not found!

```
