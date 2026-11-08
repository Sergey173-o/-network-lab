# -*- mode: ruby -*-
# Создаёт семь ВМ и подключает их к изолированным сетям VirtualBox.
require "ipaddr"
require "yaml"

machines = YAML.safe_load(File.read(File.join(__dir__, "topology.yml")))

Vagrant.configure("2") do |config|
  config.vm.boot_timeout = 900
  # Точный образ, на котором загрузились три роутера предыдущей лаборатории.
  config.vm.box = "generic/ubuntu2004"
  config.vm.box_version = "4.3.12"
  config.vm.box_check_update = false
  config.vm.synced_folder ".", "/vagrant", disabled: true

  machines.each_with_index do |(name, machine), machine_index|
    config.vm.define name do |vm|
      vm.vm.hostname = name.downcase

      # Один и тот же MAC используется VirtualBox и конфигурацией гостевой ОС.
      nat_mac = format("02:4e:4c:01:%02x:01", machine_index + 1)
      vm.vm.provider "virtualbox" do |vb|
        vb.name = "network-lab-console-#{name}"
        vb.memory = 768
        vb.cpus = 1
        vb.customize ["modifyvm", :id, "--macaddress1", nat_mac.delete(":")]
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"] if name == "inetRouter"
      end

      interfaces = machine.fetch("links").each_with_index.map do |(segment, link), index|
        adapter = index + 2
        mac = format("02:4e:4c:01:%02x:%02x", machine_index + 1, adapter)
        ip, prefix = link.fetch("addresses").first.split("/")
        netmask = IPAddr.new("255.255.255.255").mask(Integer(prefix)).to_s
        vm.vm.network "private_network", ip: ip, netmask: netmask, adapter: adapter,
                      auto_config: false, mac: mac.delete(":"),
                      virtualbox__intnet: "network-lab-console-#{segment}"
        link.merge("segment" => segment, "mac" => mac)
      end

      vm.vm.provision "ansible" do |ansible|
        ansible.playbook = "ansible/provision.yml"
        ansible.extra_vars = {
          lab_links: interfaces,
          lab_nat_mac: nat_mac,
          lab_router: machine.fetch("router", false),
          lab_edge: name == "inetRouter"
        }
      end
    end
  end
end

# Версия 3 | 2026-11-07 | Тайм-аут SSH 900 секунд; сохранён проверенный образ.
