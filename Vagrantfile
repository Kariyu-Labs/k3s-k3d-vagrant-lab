Vagrant.configure("2") do |config|

  machines = {
    "kquetat-S" => "192.168.56.110",
    "kquetat-SW" => "192.168.56.111"
  }

  machines.each do |machine_name, machine_ip|
    config.vm.define machine_name do |machine|
      machine.vm.box = "generic/alpine318"
      machine.vm.hostname = machine_name
      machine.vm.network "private_network", ip: machine_ip

      if machine_name == "kquetat-S"
        machine.vm.synced_folder "./shared", "/vagrant/shared"
      end

      if machine_name == "kquetat-SW"
        machine.vm.synced_folder "./shared", "/vagrant/shared"
      end

      # if the machine is kquetat-S, install k3s as master
      if machine_name == "kquetat-S"
        machine.vm.provision "shell",
          inline: <<-END
            echo "Provisioning #{machine_name}..."
            apk add --no-cache curl
            curl -sfL https://get.k3s.io | sh -

            # Wait for node-token to appear
            while [ ! -f /var/lib/rancher/k3s/server/node-token ]; do
              echo "Waiting for k3s to generate node-token..."
              sleep 3
            done

            cp /var/lib/rancher/k3s/server/node-token /vagrant/shared/node-token
            echo "K3s installed on #{machine_name}."
            echo "Node-token copied to /vagrant/shared/node-token"
          END
      # if the machine is kquetat-SW, install k3s as worker
      elsif machine_name == "kquetat-SW"
        machine.vm.provision "shell",
          inline: <<-END
            echo "Provisioning #{machine_name}..."
            while [ ! -f /vagrant/shared/node-token ]; do
              echo "Waiting for node-token..."
              sleep 5
            done
            echo "Node-token found, proceeding with installation..."
            TOKEN=$(cat /vagrant/shared/node-token)
            apk add --no-cache curl
            curl -sfL https://get.k3s.io | K3S_URL=https://#{machines["kquetat-S"]}:6443 K3S_TOKEN=$TOKEN sh -
            echo "K3s installed on #{machine_name}."
          END
      end

      machine.vm.provider :virtualbox do |provider|
        provider.cpus = 1
        provider.memory = 512
      end
    end
  end

  # Configure SSH no pw
  config.ssh.insert_key = false

end
