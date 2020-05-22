# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"

MACHINES = {
  :zfs => {
        :box_name => "centos/7",
        :box_version => "1804.02",
        :ip_addr => '192.168.11.101',
    :disks => {
        :sata20 => {
            :dfile => home + '/VirtualBox VMs/sata20.vdi',
            :size => 10240,
            :port => 1
        },
        :sata21 => {
            :dfile => home + '/VirtualBox VMs/sata21.vdi',
            :size => 10240, # Megabytes
            :port => 2
        }
    }
  },
}

Vagrant.configure("2") do |config|

    config.vm.box_version = "1804.02"
    MACHINES.each do |boxname, boxconfig|
      config.vbguest.no_install = true
        config.vm.define boxname do |box|
            box.vm.box = boxconfig[:box_name]
            box.vm.host_name = boxname.to_s

            #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset

            box.vm.network "private_network", ip: boxconfig[:ip_addr]

            box.vm.provider :virtualbox do |vb|
                    vb.customize ["modifyvm", :id, "--memory", "4096"]
                    needsController = false
            boxconfig[:disks].each do |dname, dconf|
                unless File.exist?(dconf[:dfile])
                  vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                                  needsController =  true
                            end

            end
                    if needsController == true
                       vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                       boxconfig[:disks].each do |dname, dconf|
                           vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                       end
                    end
            end

        box.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh
            cp ~vagrant/.ssh/auth* ~root/.ssh
            echo -e "\n\n-----------------------------------------------"
            echo -e "Определение алгоритма с наилучшим сжатием\n\n"
            echo -e "\nУстановим необходимые программы...(займёт некоторое время)"
            yum install -y -q http://download.zfsonlinux.org/epel/zfs-release.el7_5.noarch.rpm > /dev/null 2>&1
            yum-config-manager --disable zfs > /dev/null 2>&1
            yum-config-manager --enable zfs-kmod > /dev/null 2>&1
            yum install zfs wget -y > /dev/null 2>&1
            /sbin/modprobe zfs
            echo -e "\nСоздадим mirror pool из дисков sdb и sdc"
            zpool create mypool mirror /dev/sdb /dev/sdc
            echo "Создадим файловые системы с разным сжатием"
            zfs create -o compression=off mypool/dir_off
            zfs create -o compression=lzjb mypool/dir_lzjb
            zfs create -o compression=gzip mypool/dir_gzip
            zfs create -o compression=zle mypool/dir_zle
            echo -e "\nСкачаем для теста ядро linux и распакуем его в разные файловые системы (займет много времени на медленных дисках)"
            echo "Скачиваем..."
            wget -q https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.6.14.tar.xz
            echo "Распаковываем в директорию без сжатия..."
            tar -xf linux-5.6.14.tar.xz -C /mypool/dir_off
            echo "Распаковываем в директорию со сжатием gzip.."
            tar -xf linux-5.6.14.tar.xz -C /mypool/dir_gzip
            echo "Распаковываем в директорию со сжатием lzjb.."
            tar -xf linux-5.6.14.tar.xz -C /mypool/dir_lzjb
            echo "Распаковываем в директорию со сжатием zle.."
            tar -xf linux-5.6.14.tar.xz -C /mypool/dir_zle
            echo -e "\nСравним используемый объем после распаковки:"
            zfs list
            sleep 5
            echo -e "\n\n-----------------------------------------------"
            echo -e "Определяем настройки pool’a\n\n"
            echo "Скачиваем архив с google drive..."
            wget -q --load-cookies /tmp/cookies.txt "https://drive.google.com/uc?export=download&confirm=$(wget --quiet --save-cookies /tmp/cookies.txt --keep-session-cookies --no-check-certificate 'https://drive.google.com/uc?export=download&id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg' -O- | sed -rn 's$.*confirm=([0-9A-Za-z_]+).*$\1\n$p')&id=1KRBNW33QWqbvbVHa3hLJivOAt60yukkg" -O zfs_task1.tar.gz && rm -rf /tmp/cookies.txt
            tar xf zfs_task1.tar.gz
            echo -e "\nПробуем импортировать пул..."
            zpool import -d /home/vagrant/zpoolexport/
            echo -e "\n\nИз-за разных версий ZFS какие то опции недоступны..."
            echo -e "\nСмонтируем в режиме read-only..."
            zpool import otus -o readonly=on -d /home/vagrant/zpoolexport/
            zpool list otus
            echo -e "\nРазмер хранилища: `zpool list -H -o size otus`"
            echo "Тип пула: `zpool list -v -H -P otus|head -n2|tail -n1|awk '{print $1}'`"
            echo "Recordsize: `zfs get -H -o value recordsize otus`"
            echo "Compression: `zfs get -H -o value compression otus`"
            echo "Checksum: `zfs get -H -o value checksum otus`"
            sleep 5
            echo -e "\n\n-----------------------------------------------"
            echo -e "Найти сообщение от преподавателей\n\n"
            echo "Скачиваем файл.."
            wget -q --load-cookies /tmp/cookies.txt "https://drive.google.com/uc?export=download&confirm=$(wget --quiet --save-cookies /tmp/cookies.txt --keep-session-cookies --no-check-certificate 'https://drive.google.com/uc?export=download&id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG' -O- | sed -rn 's$.*confirm=([0-9A-Za-z_]+).*$\1\n$p')&id=1gH8gCL9y7Nd5Ti3IRmplZPF1XjzxeRAG" -O otus_task2.file && rm -rf /tmp/cookies.txt
            echo -e "Восстановим полученный файл.. "
            cat otus_task2.file | sudo zfs recv -F mypool/task3
            echo "\nВыведем сообщение из файла:\n"
            cat `find /mypool/task3/ -name "secret_message"`
            echo -e "\n!!!====================================================================!!!"
            echo "!!!                       Все задания выполнены                        !!!"
            echo "!!!                                                                    !!!"
            echo "!!!                         Спасибо за проверку!                       !!!"
            echo "!!!====================================================================!!!"


          SHELL

        end
    end
  end
