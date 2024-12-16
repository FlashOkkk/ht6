# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2204"
  
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.cpus = 2

    # Додавання 4 додаткових дисків по 400MB
    (1..4).each do |i|
      vb.customize ['createhd', '--filename', "./disk#{i}.vdi", '--size', 400] # 400MB
      vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', i,
                    '--device', 0, '--type', 'hdd', '--medium', "./disk#{i}.vdi"]
    end
  end

  # Налаштування додаткових дисків, LVM і точок монтування
  config.vm.provision "shell", inline: <<-SHELL
    # Оновлення системи та встановлення необхідних пакетів
    sudo apt-get update -y
    sudo apt-get install -y lvm2 parted

    # Створення таблиці розділів GPT і первинного розділу на кожному диску
    for DISK in /dev/sd{b,c,d,e}; do
      sudo parted --script $DISK mklabel gpt
      sudo parted --script $DISK mkpart primary 0% 100%
    done

    # Створення групи томів та логічних томів
    sudo vgcreate vg_data /dev/sdb1 /dev/sdc1 /dev/sdd1 /dev/sde1
    sudo lvcreate -L 800M -n vol0 vg_data
    sudo lvcreate -L 800M -n vol1 vg_data

    # Створення файлової системи для томів
    sudo mkfs.ext4 /dev/vg_data/vol0
    sudo mkfs.ext4 /dev/vg_data/vol1

    # Створення точок монтування
    sudo mkdir -p /mnt/vol0 /mnt/vol1

    # Монтування томів
    sudo mount /dev/vg_data/vol0 /mnt/vol0
    sudo mount /dev/vg_data/vol1 /mnt/vol1

    # Налаштування /etc/fstab для автоматичного монтування
    echo '/dev/vg_data/vol0 /mnt/vol0 ext4 defaults 0 0' | sudo tee -a /etc/fstab
    echo '/dev/vg_data/vol1 /mnt/vol1 ext4 defaults 0 0' | sudo tee -a /etc/fstab

    # Налаштування прав доступу
    sudo chmod 777 /mnt/vol0
    sudo chmod 777 /mnt/vol1

    # Перевірка
    sudo mount -a
    df -h
  SHELL
end

