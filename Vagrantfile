# -*- mode: ruby -*-
# vim: set ft=ruby :

# Указываем зеркало для скачивания образов
ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'

Vagrant.configure("2") do |config|

  # ============================================
  # 1. inetRouter (AlmaLinux 9)
  # ============================================
  config.vm.define "inetRouter" do |inet|
    inet.vm.box = "almalinux/9"
    inet.vm.hostname = "inetRouter"
    inet.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end

    inet.vm.network "private_network", ip: "192.168.255.1", adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"
    inet.vm.network "private_network", ip: "192.168.51.10", adapter: 3, netmask: "255.255.255.0"

    inet.vm.provision "shell",
      run: "always",
      inline: <<-SHELL
        sudo dnf install -y iptables-services
        sudo systemctl enable iptables
        sudo sysctl -w net.ipv4.ip_forward=1
        grep -q "net.ipv4.ip_forward" /etc/sysctl.conf || echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
        sudo iptables -t nat -A POSTROUTING ! -d 192.168.0.0/16 -o eth0 -j MASQUERADE
        sudo iptables -A FORWARD -j ACCEPT
        sudo iptables-save | sudo tee /etc/sysconfig/iptables
        sudo nmcli connection modify "System eth1" +ipv4.routes "192.168.0.0/16 192.168.255.2"
        sudo nmcli con reload
        sudo nmcli con up 'System eth1'
      SHELL
  end

  # ============================================
  # 2. centralRouter
  # ============================================
  config.vm.define "centralRouter" do |central|
    central.vm.box = "almalinux/9"
    central.vm.hostname = "centralRouter"
    central.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end
    central.vm.network "private_network", ip: "192.168.255.2", adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "router-net"
    central.vm.network "private_network", ip: "192.168.52.10", adapter: 3, netmask: "255.255.255.0"
    central.vm.network "private_network", ip: "192.168.0.1", adapter: 4, netmask: "255.255.255.240", virtualbox__intnet: "directors-net"
    central.vm.network "private_network", ip: "192.168.255.9", adapter: 5, netmask: "255.255.255.252", virtualbox__intnet: "office1Router-net"
    central.vm.network "private_network", ip: "192.168.255.5", adapter: 6, netmask: "255.255.255.252", virtualbox__intnet: "office2Router-net"
    central.vm.network "private_network", ip: "192.168.0.33", adapter: 7, netmask: "255.255.255.240", virtualbox__intnet: "hardware2-net"
    # на схеме указано две сети Office hardware и не указана сеть wifi. Вторую сеть Office hardware заменил на сеть wifi.
    central.vm.network "private_network", ip: "192.168.0.65", adapter: 8, netmask: "255.255.255.192", virtualbox__intnet: "wifi-net"

    central.vm.provision "shell",
    run: "always",
    inline: <<-SHELL
      nmcli con modify 'eth0' ipv4.never-default yes
      nmcli con reload
      nmcli con up 'eth0'
      sudo sysctl -w net.ipv4.ip_forward=1
      grep -q "net.ipv4.ip_forward" /etc/sysctl.conf || echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
      sudo nmcli connection modify "System eth4" +ipv4.routes "192.168.2.0/24 192.168.255.10"
      sudo nmcli connection modify "System eth5" +ipv4.routes "192.168.1.0/24 192.168.255.6"
      sudo nmcli connection modify "System eth1" +ipv4.routes "0.0.0.0/0 192.168.255.1"
      sudo nmcli con reload
      sudo nmcli con up 'System eth1'
      sudo nmcli con up 'System eth4'
      sudo nmcli con up 'System eth5'
    SHELL
  end

  # ============================================
  # 3. centralServer
  # ============================================
  config.vm.define "centralServer" do |srv|
    srv.vm.box = "almalinux/9"
    srv.vm.hostname = "centralServer"
    srv.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end
    srv.vm.network "private_network", ip: "192.168.0.2", adapter: 2, netmask: "255.255.255.240", virtualbox__intnet: "directors-net"
    srv.vm.network "private_network", ip: "192.168.53.10", adapter: 3, netmask: "255.255.255.0"

    srv.vm.provision "shell",
    run: "always",
    inline: <<-SHELL
      nmcli con modify 'eth0' ipv4.never-default yes
      nmcli con reload
      nmcli con up 'eth0'
      sudo nmcli connection modify "System eth1" +ipv4.routes "0.0.0.0/0 192.168.0.1"
      sudo nmcli con reload
      sudo nmcli con up 'System eth1'
      SHELL
  end

  # ============================================
  # 4. office1Router
  # ============================================
  config.vm.define "office1Router" do |office1|
    office1.vm.box = "almalinux/9"
    office1.vm.hostname = "office1Router"
    office1.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end
    office1.vm.network "private_network", ip: "192.168.255.10", adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office1Router-net"
    office1.vm.network "private_network", ip: "192.168.54.10", adapter: 3, netmask: "255.255.255.0"
    office1.vm.network "private_network", ip: "192.168.2.1", adapter: 4, netmask: "255.255.255.192", virtualbox__intnet: "dev4-net"
    # начале методички указана маска /26 для Test servers, а на схеме уже указана маска /28. Принял решение использовать маску /28.
    office1.vm.network "private_network", ip: "192.168.2.65", adapter: 5, netmask: "255.255.255.240", virtualbox__intnet: "test4-net"
    office1.vm.network "private_network", ip: "192.168.2.129", adapter: 6, netmask: "255.255.255.192", virtualbox__intnet: "managers-net"
    # на схеме указан интерфейс 192.168.2.192/26, а это адрес сети. Исправил на 192.168.2.193/26, как первый доступный айпи в этой сети.
    office1.vm.network "private_network", ip: "192.168.2.193", adapter: 7, netmask: "255.255.255.192", virtualbox__intnet: "hardware4-net"

    office1.vm.provision "shell",
    run: "always",
    inline: <<-SHELL
      nmcli con modify 'eth0' ipv4.never-default yes
      nmcli con reload
      nmcli con up 'eth0'
      sudo sysctl -w net.ipv4.ip_forward=1
      grep -q "net.ipv4.ip_forward" /etc/sysctl.conf || echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
      sudo nmcli connection modify "System eth1" +ipv4.routes "0.0.0.0/0 192.168.255.9"
      sudo nmcli con reload
      sudo nmcli con up 'System eth1'
    SHELL
  end

  # ============================================
  # 5. office1Server
  # ============================================
  config.vm.define "office1Server" do |srv|
    srv.vm.box = "almalinux/9"
    srv.vm.hostname = "office1Server"
    srv.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end
    srv.vm.network "private_network", ip: "192.168.2.130", adapter: 2, netmask: "255.255.255.192", virtualbox__intnet: "managers-net"
    srv.vm.network "private_network", ip: "192.168.55.10", adapter: 3, netmask: "255.255.255.0"

    srv.vm.provision "shell",
    run: "always",
    inline: <<-SHELL
      nmcli con modify 'eth0' ipv4.never-default yes
      nmcli con reload
      nmcli con up 'eth0'
      sudo nmcli connection modify "System eth1" +ipv4.routes "0.0.0.0/0 192.168.2.129"
      sudo nmcli con reload
      sudo nmcli con up 'System eth1'
    SHELL
  end

  # ============================================
  # 6. office2Router
  # ============================================
  config.vm.define "office2Router" do |office2|
    office2.vm.box = "almalinux/9"
    office2.vm.hostname = "office2Router"
    office2.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end
    office2.vm.network "private_network", ip: "192.168.255.6", adapter: 2, netmask: "255.255.255.252", virtualbox__intnet: "office2Router-net"
    office2.vm.network "private_network", ip: "192.168.56.10", adapter: 3, netmask: "255.255.255.0"
    office2.vm.network "private_network", ip: "192.168.1.1", adapter: 4, netmask: "255.255.255.128", virtualbox__intnet: "dev6-net"
    office2.vm.network "private_network", ip: "192.168.1.129", adapter: 5, netmask: "255.255.255.192", virtualbox__intnet: "test6-net"
    office2.vm.network "private_network", ip: "192.168.1.193", adapter: 6, netmask: "255.255.255.192", virtualbox__intnet: "hardware6-net"

    office2.vm.provision "shell",
    run: "always",
    inline: <<-SHELL
      nmcli con modify 'eth0' ipv4.never-default yes
      nmcli con reload
      nmcli con up 'eth0'
      sudo sysctl -w net.ipv4.ip_forward=1
      grep -q "net.ipv4.ip_forward" /etc/sysctl.conf || echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
      sudo nmcli connection modify "System eth1" +ipv4.routes "0.0.0.0/0 192.168.255.5"
      sudo nmcli con reload
      sudo nmcli con up 'System eth1'
    SHELL
  end

  # ============================================
  # 7. office2Server
  # ============================================
  config.vm.define "office2Server" do |srv|
    srv.vm.box = "almalinux/9"
    srv.vm.hostname = "office2Server"
    srv.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
    end
    srv.vm.network "private_network", ip: "192.168.1.2", adapter: 2, netmask: "255.255.255.128", virtualbox__intnet: "dev6-net"
    srv.vm.network "private_network", ip: "192.168.57.10", adapter: 3, netmask: "255.255.255.0"

    srv.vm.provision "shell",
    run: "always",
    inline: <<-SHELL
      nmcli con modify 'eth0' ipv4.never-default yes
      nmcli con reload
      nmcli con up 'eth0'
      sudo nmcli connection modify "System eth1" +ipv4.routes "0.0.0.0/0 192.168.1.1"
      sudo nmcli con reload
      sudo nmcli con up 'System eth1'
    SHELL
  end

end
