Vagrant.configure("2") do |config|
    config.vm.box = "bento/rockylinux-8"
    config.vm.hostname = 'vagr-openCHAMI'
    # Disabling shared folders and smb mounts because there's no way to mitigate 
    # the netskope meddling before Vagrant tries to install packages for shared mounts
    config.vm.synced_folder '.','/vagrant', disabled: true
    config.vm.provider "qemu" do |qe|
        qe.arch = "x86_64"
        qe.machine = "q35"
        qe.cpu = "max"
        qe.smp = "4"
        qe.memory = "8G"
        qe.net_device = "virtio-net-pci"
        qe.ssh_port = "50023"
    end
    # config.vm.network "forwarded_port", guest: 8000, host: 8000, host_ip: "127.0.0.1"
    # config.vm.network "private_network", ip: "172.16.0.2"
    config.vm.provision "shell", inline: <<-SHELL #priviledged by default
        # Add SSL certificates from Netskope to Rocky's TLS certs so we can actually download stuff
        echo "Setting certs..."
        cd /etc/pki/ca-trust/extracted/pem/
        chmod 777 ./tls-ca-bundle.pem
        sudo openssl s_client -showcerts -servername mirrors.rockylinux.org -connect mirrors.rockylinux.org:443 2>/dev/null </dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' >> ./tls-ca-bundle.pem
        chmod 600 ./tls-ca-bundle.pem
        echo "Certs set"
        echo "Downloading openCHAMI and installing associated packages..."
        curl --cacert /etc/pki/tls/certs/ca-bundle.trust.crt https://github.com/OpenCHAMI/deployment-recipes/archive/refs/heads/main.zip -OJL
        dnf install -y unzip jq yum-utils
        yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
        yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
        systemctl start docker
        echo "Done."
        echo "Unzipping openCHAMI..."
        unzip deployment-recipes-main.zip
        cd deployment-recipes-main/quickstart/
        ./generate-configs.sh chamiTest

        # dnf update -y

    SHELL
end
