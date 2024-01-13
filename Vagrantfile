MACHINES = {
  # Указываем имя ВМ "kernel update"
  :"kernel-update" => {
              #Какой vm box будем использовать
              :box_name => "generic/centos8s",
              #Указываем box_version
              :box_version => "4.3.4",
              #Указываем количество ядер ВМ
              :cpus => 2,
              #Указываем количество ОЗУ в мегабайтах
              :memory => 1024,
            }
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    # Отключаем проброс общей папки в ВМ
    config.vm.synced_folder ".", "/vagrant", disabled: true
    # Применяем конфигурацию ВМ
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.box_version = boxconfig[:box_version]
      box.vm.host_name = boxname.to_s
        box.vm.provider "virtualbox" do |v|
          v.memory = boxconfig[:memory]
          v.cpus = boxconfig[:cpus]
        end
    end
  end
  config.vm.provision "shell", inline: <<-SHELL
     uname -r
     sudo yum install -y https://www.elrepo.org/elrepo-release-8.el8.elrepo.noarch.rpm
     sudo yum --enablerepo elrepo-kernel install kernel-ml -y
     sudo grub2-mkconfig -o /boot/grub2/grub.cfg
     sudo grub2-set-default 0
     sudo reboot now
  SHELL
end
#ADD

