require "yaml"
settings = YAML.load_file "settings.yaml"

puts ""
puts ""
puts "\e[32m
  ____        _        _       _        
 |  _ \\  __ _| |_ __ _| | __ _| | _____ 
 | | | |/ _` | __/ _` | |/ _` | |/ / _ \\
 | |_| | (_| | || (_| | | (_| |   <  __/
 |____/ \\__,_|\\__\\__,_|_|\\__,_|_|\\_\\___|
\e[0m"
puts ""
puts "\e[32m github.com/rerichardjr\e[0m"
puts ""
puts ""

SPARK_VER = settings["software"]["spark"]
MINIO_VER = settings["software"]["minio"]
HUDI_VER = settings["software"]["hudi"]
HAPROXY_VER = settings["software"]["haproxy"]
SUPPORT_USER = settings["support_user"]
NETWORK = settings["ip"]["network"]
IP = settings["ip"]["start"]
DOMAIN = settings["ip"]["domain"]
BRIDGE = settings["bridge"]
COMPUTE_COUNT = settings["nodes"]["compute"]["count"]
COMPUTE_HOSTNAME = settings["nodes"]["compute"]["hostname"]
LB_VIP_COUNT = settings["vips"]["minio_vip"]["count"]
LB_VIP_HOSTNAME = settings["vips"]["minio_vip"]["hostname"]
LB_COUNT = settings["nodes"]["loadbalancer"]["count"]
LB_HOSTNAME = settings["nodes"]["loadbalancer"]["hostname"]
DISK_SIZE=settings["nodes"]["storage"]["disk_size"]
DISK_COUNT=settings["nodes"]["storage"]["disk_count"]
STORAGE_COUNT = settings["nodes"]["storage"]["count"]
STORAGE_HOSTNAME = settings["nodes"]["storage"]["hostname"]

# Build hostname-to-IP map for /etc/hosts
nodes = {
  "loadbalancer_vip" => { "hostname" => LB_VIP_HOSTNAME, "count" => LB_VIP_COUNT },
  "storage" => { "hostname" => STORAGE_HOSTNAME, "count" => STORAGE_COUNT },
  "loadbalancer" => { "hostname" => LB_HOSTNAME, "count" => LB_COUNT },
  "compute" => { "hostname" => COMPUTE_HOSTNAME, "count" => COMPUTE_COUNT },
}

current_ip = IP
hostname_ip_map = {}
nodes.each do |node_type, config|
  config["count"].times do |i|
    hostname = if config["hostname"] == LB_VIP_HOSTNAME
                 config["hostname"]
               else
                 "#{config['hostname']}#{i + 1}"
               end
    ip = "#{NETWORK}.#{current_ip}"
    hostname_ip_map[hostname] = ip
    current_ip += 1
  end
end

map_strings = hostname_ip_map.map { |key, value| "#{key}=#{value}" }
map_strings_bash = map_strings.join(" ")

# set LB VIP and increment
LB_VIP_IP=IP
IP+=1

DEFAULT_VM_FOLDER = `VBoxManage list systemproperties | grep "Default machine folder"`.split(":").last.strip
puts "Using VirtualBox default folder: #{DEFAULT_VM_FOLDER}"

Vagrant.configure("2") do |config|
  config.vm.provision "file", source: "scripts/lib.sh", destination: "/tmp/lib.sh"
  config.vm.box = settings["software"]["box"]
  config.vm.box_check_update = true

  config.vm.provision "shell",
    name: "Support user creation",
    env: {
      "SUPPORT_USER" => SUPPORT_USER,
    },
    path: "scripts/common.sh"

    config.vm.provision "shell",
    name: "Networking configuration",
    env: {
      "IP" => IP,
      "NETWORK" => NETWORK,
      "DOMAIN" => DOMAIN,
      "HOSTNAME_IP_MAP" => map_strings_bash,
      "STORAGE_HOSTNAME" => STORAGE_HOSTNAME,
    },
    path: "scripts/networking.sh"


  # Storage nodes
  (1..STORAGE_COUNT).each do |i|
    config.vm.define "#{STORAGE_HOSTNAME}#{i}" do |storage|
      storage.vm.hostname = "#{STORAGE_HOSTNAME}#{i}"
      storage.vm.network :public_network, ip: "#{NETWORK}.#{IP}", :bridge => BRIDGE
      IP+=1
      RANDOM_SEQ = (Time.now.to_i).to_s << "_" << (rand(10000..99999)).to_s
      storage.vm.provider "virtualbox" do |vb|
        vb.cpus = settings["nodes"]["storage"]["cpu"]
        vb.memory = settings["nodes"]["storage"]["memory"]
        vm_dir = File.join(DEFAULT_VM_FOLDER, "vagrant-datalake_#{STORAGE_HOSTNAME}#{i}_#{RANDOM_SEQ}")
        FileUtils.mkdir_p(vm_dir) unless Dir.exist?(vm_dir)
        DISK_COUNT.times do |d|
          disk_name = "disk-#{i}-#{d}.vmdk"
          disk_path = File.join(vm_dir, disk_name)
          vb.customize ["createhd", "--filename", disk_path, "--size", DISK_SIZE * 1024]
          vb.customize ["storageattach", :id, "--storagectl", "SATA Controller", "--port", d + 1, "--device", 0, "--type", "hdd", "--medium", disk_path]
        end
      end

      storage.vm.provision "shell",
        name: "Configure storage nodes",
        env: {
            "LB_VIP_HOSTNAME" => LB_VIP_HOSTNAME,
            "STORAGE_HOSTNAME" => STORAGE_HOSTNAME,
            "NODE_ID" => "#{i}",
            "STORAGE_COUNT" => STORAGE_COUNT,
            "DISK_COUNT" => DISK_COUNT,
            "DOMAIN" => DOMAIN,
            "MINIO_VER" => MINIO_VER,
        },
        path: "scripts/storage-node.sh"

    end
  end

  # Load balancing nodes
  (1..LB_COUNT).each do |i|
    config.vm.define "#{LB_HOSTNAME}#{i}" do |lb|
      lb.vm.hostname = "#{LB_HOSTNAME}#{i}"
      lb.vm.network :public_network, ip: "#{NETWORK}.#{IP}", :bridge => BRIDGE
      IP+=1
      lb.vm.provider "virtualbox" do |vb|
        vb.cpus = settings["nodes"]["loadbalancer"]["cpu"]
        vb.memory = settings["nodes"]["loadbalancer"]["memory"]
      end

      lb.vm.provision "shell",
        name: "Configure load balancing nodes",
        env: {
            "NETWORK" => NETWORK,
            "NODE_ID" => "#{i}",
            "LB_VIP_HOSTNAME" => LB_VIP_HOSTNAME,
            "LB_VIP_IP" => LB_VIP_IP,
            "STORAGE_HOSTNAME" => STORAGE_HOSTNAME,
            "DOMAIN" => DOMAIN,
            "HOSTNAME_IP_MAP" => map_strings_bash,
        },
        path: "scripts/lb-node.sh"

    end
  end

  # Compute nodes
  (1..COMPUTE_COUNT).each do |i|
    config.vm.define "#{COMPUTE_HOSTNAME}#{i}" do |compute|
      compute.vm.hostname = "#{COMPUTE_HOSTNAME}#{i}"
      compute.vm.network :public_network, ip: "#{NETWORK}.#{IP}", :bridge => BRIDGE
      IP+=1
      compute.vm.provider "virtualbox" do |vb|
        vb.cpus = settings["nodes"]["compute"]["cpu"]
        vb.memory = settings["nodes"]["compute"]["memory"]
      end

      compute.vm.provision "shell",
        name: "Configure Spark master and worker nodes",
        env: {
            "SPARK_VER" => SPARK_VER,
            "SPARK_MASTER" => "#{COMPUTE_HOSTNAME}1.#{DOMAIN}",
            "NODE_ID" => "#{i}",
        },
        path: "scripts/compute-node.sh"
    end
  end

  # Automation nodes
  (1..COMPUTE_COUNT).each do |i|
    config.vm.define "#{COMPUTE_HOSTNAME}#{i}" do |compute|
      compute.vm.hostname = "#{COMPUTE_HOSTNAME}#{i}"
      compute.vm.network :public_network, ip: "#{NETWORK}.#{IP}", :bridge => BRIDGE
      IP+=1
      compute.vm.provider "virtualbox" do |vb|
        vb.cpus = settings["nodes"]["automation"]["cpu"]
        vb.memory = settings["nodes"]["automation"]["memory"]
      end

      compute.vm.provision "shell",
        name: "Configure Apache Airflow",
        env: {
            "AIRFLOW_VER" => AIRFLOW_VERSION,
            "NODE_ID" => "#{i}",
        },
        path: "scripts/automation-node.sh"
    end
  end

end
