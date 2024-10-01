Vagrant.configure("2") do |config|
    config.vm.define "sms", before: :all, primary: true do |sms|
        sms.vm.box = "bento/rockylinux-9.3"
        sms.vm.hostname = "vagr-oC-sms"
        # Disabling shared folders and smb mounts because there's no way to mitigate 
        # the netskope meddling before Vagrant tries to install packages for shared mounts
        sms.vm.synced_folder '.','/vagrant', disabled: true
        sms.vm.provider "virtualbox" do |v|
            v.memory = 8192
            v.cpus = 4
        end
        sms.vm.network "private_network", ip: "172.16.70.100", virtualbox__intnet: true #for cluster connection
        sms.vm.provision "shell", inline: <<-SHELL #priviledged by default
            # dnf update -y
            # Add SSL certificates from Netskope to Rocky's TLS certs so we can actually download stuff
            # echo "Setting certs..."
            # cd /etc/pki/ca-trust/extracted/pem/
            # chmod 777 ./tls-ca-bundle.pem
            # sudo openssl s_client -showcerts -servername mirrors.rockylinux.org -connect mirrors.rockylinux.org:443 2>/dev/null </dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' >> ./tls-ca-bundle.pem
            # chmod 600 ./tls-ca-bundle.pem
            # echo "Certs set"

            echo "Downloading openCHAMI and installing associated packages..."
            su vagrant
            cd /home/vagrant/
            curl --cacert /etc/pki/tls/certs/ca-bundle.trust.crt https://github.com/OpenCHAMI/deployment-recipes/archive/refs/heads/main.zip -OJL
            dnf install -y unzip jq yum-utils
            yum-config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
            yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
            systemctl start docker
            echo "Done."

            echo "Unzipping openCHAMI..."
            unzip deployment-recipes-main.zip
            echo "Done."
            
            #Derived from the openCHAMI quickstart guide
            echo "Configuring and starting openCHAMI..."
            cd deployment-recipes-main/quickstart/
            ./generate-configs.sh chamiTest
            docker compose -f base.yml -f postgres.yml -f jwt-security.yml -f haproxy-api-gateway.yml -f  openchami-svcs.yml -f autocert.yml up -d
            echo "Done."

            echo "Obtaining rootca and tokens..."
            source bash_functions.sh
            get_ca_cert > cacert.pem
            ACCESS_TOKEN=$(gen_access_token)
            echo '127.0.0.1  vagr-oC-sms.openchami.cluster' | sudo tee -a /etc/hosts
            curl --cacert cacert.pem -H "Authorization: Bearer $ACCESS_TOKEN" https://vagr-oC-sms.openchami.cluster:8443/hsm/v2/State/Components
            echo "Creating a dedicated token for dnsmasq..."
            echo "DNSMASQ_ACCESS_TOKEN=$(gen_access_token)" >> .env
            docker compose -f base.yml -f postgres.yml -f jwt-security.yml -f haproxy-api-gateway.yml -f openchami-svcs.yml -f autocert.yml -f dnsmasq.yml up -d
            echo "Done..."
            
            echo "****************************************"
            echo "You should now be able to ssh into the box and explore openCHAMI" 
        SHELL
    end
    
    config.vm.define "node1", after: :all, autostart: false do |node1|
        node1.vm.box = "clink15/pxe"
        node1.vm.hostname = "vagr-oC-n1"
        # Disabling shared folders and smb mounts because there's no way to mitigate 
        # the netskope meddling before Vagrant tries to install packages for shared mounts
        node1.vm.synced_folder '.','/vagrant', disabled: true
        node1.vm.provider "virtualbox" do |v|
            v.memory = 4096
            v.cpus = 2
        end
        node1.vm.network "private_network", ip: "172.16.70.101", virtualbox__intnet: true #for cluster connection

    end
end