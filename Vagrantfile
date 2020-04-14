# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.

# WARNING: You will need the following plugin:
# vagrant plugin install vagrant-docker-compose
if Vagrant.plugins_enabled?
  unless Vagrant.has_plugin?('vagrant-docker-compose')
    puts 'Plugin missing.'
    system('vagrant plugin install vagrant-docker-compose')
    puts 'Dependencies installed, please try the command again.'
    exit
  end
end

Vagrant.configure(2) do |config|
  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "ubuntu/bionic64"
  config.vm.hostname = "project"

  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "forwarded_port", guest: 5000, host: 5000, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  config.vm.network "private_network", ip: "192.168.33.10"

  # Mac users can comment this next line out but
  # Windows users need to change the permission of files and directories
  # so that nosetests runs without extra arguments.
  config.vm.synced_folder ".", "/vagrant", mount_options: ["dmode=775,fmode=664"]

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Customize the amount of memory on the VM:
    vb.memory = "1024"
    vb.cpus = 2
    # Fixes some DNS issues on some networks
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
    vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
  end

  # Copy your .gitconfig file so that your git credentials are correct
  if File.exists?(File.expand_path("~/.gitconfig"))
    config.vm.provision "file", source: "~/.gitconfig", destination: "~/.gitconfig"
  end

  # Copy the ssh keys into the vm for git access
  if File.exists?(File.expand_path("~/.ssh/id_rsa"))
    config.vm.provision "file", source: "~/.ssh/id_rsa", destination: "~/.ssh/id_rsa"
  end

  if File.exists?(File.expand_path("~/.ssh/id_rsa.pub"))
    config.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "~/.ssh/id_rsa.pub"
  end

  # Copy your .vimrc file so that your vi looks like you expect
  if File.exists?(File.expand_path("~/.vimrc"))
    config.vm.provision "file", source: "~/.vimrc", destination: "~/.vimrc"
  end

  # Copy your IBM Cloud API Key if you have one
  if File.exists?(File.expand_path("~/.suppliers/apiKey.json"))
    config.vm.provision "file", source: "~/.suppliers/apiKey.json", destination: "~/.suppliers/apiKey.json"
  end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y git python3 python3-pip python3-venv
    apt-get -y autoremove

    # Create a Python3 Virtual Environment and Activate it in .profile
    sudo -H -u vagrant sh -c 'python3 -m venv ~/venv'
    sudo -H -u vagrant sh -c 'echo ". ~/venv/bin/activate" >> ~/.profile'
    sudo -H -u vagrant sh -c '. ~/venv/bin/activate && cd /vagrant && pip install -r requirements.txt'

    # Install app dependencies
    # cd /vagrant
    # pip3 install -r requirements.txt
  SHELL

  ######################################################################
  # Add PostgreSQL docker container
  ######################################################################
  # docker run -d --name postgres -p 5432:5432 -v psql_data:/var/lib/postgresql/data postgres
  config.vm.provision :docker do |d|
    d.pull_images "postgres:alpine"
    d.run "postgres:alpine",
       args: "-d --name postgres -p 5432:5432 -v psql_data:/var/lib/postgresql/data -e POSTGRES_PASSWORD=postgres"
  end

  ############################################################
  # Add Docker compose
  ############################################################
  config.vm.provision :docker_compose
  # config.vm.provision :docker_compose,
  #   yml: "/vagrant/docker-compose.yml",
  #   rebuild: true,
  #   run: "always"

  ######################################################################
  # Setup a Bluemix and Kubernetes environment
  ######################################################################
  config.vm.provision "shell", inline: <<-SHELL
    echo "\n************************************"
    echo " Installing IBM Cloud CLI..."
    echo "************************************\n"
    # Install IBM Cloud CLI as Vagrant user
    sudo -H -u vagrant sh -c 'curl -sL http://ibm.biz/idt-installer | bash'
    sudo -H -u vagrant sh -c 'ibmcloud config --usage-stats-collect false'
    sudo -H -u vagrant sh -c "ibmcloud cf install --version 6.46.1"
    sudo -H -u vagrant sh -c "echo 'source <(kubectl completion bash)' >> ~/.bashrc"
    sudo -H -u vagrant sh -c "echo alias ic=/usr/local/bin/ibmcloud >> ~/.bash_aliases"
    echo "\n"
    echo "If you have an IBM Cloud API key in ~/.suppliers/apiKey.json"
    echo "You can login with the following command:"
    echo "\n"
    echo "ibmcloud login -a https://cloud.ibm.com --apikey @~/.suppliers/apiKey.json -r us-south -o <org name> -s dev"
    echo "\n"
    echo "\n************************************"
    echo " For the Kubernetes Dashboard use:"
    echo " kubectl proxy --address='0.0.0.0'"
    echo "************************************\n"

  SHELL

end
