@vagrant_version = '2'
@private_ips = (101..254).map{|v| '192.168.10.'+v.to_s}
@public_ips = (2..100).map{|v| '192.168.10.'+v.to_s}
@machines = {
  chefserver: {
    name: 'chefserver',
    public_ip: @public_ips.pop,
    private_ip: @private_ips.pop,
    ports: [
      [443, 4433],
      [80, 8003]
    ]
  },
  chefworkstation: {
    name: 'chefworkstation',
    public_ip: @public_ips.pop,
    private_ip: @private_ips.pop,
    ports: []
  },
  nodes: [
    {
      name: 'node1',
      public_ip: @public_ips.pop,
      private_ip: @private_ips.pop,
      ports: [
        [80, 8002],
        [443, 4432]
      ]
    },
    {
      name: 'node2',
      public_ip: @public_ips.pop,
      private_ip: @private_ips.pop,
      ports: [
        [80, 8001],
        [443, 4431]
      ]
    },
    {
      name: 'node3',
      public_ip: @public_ips.pop,
      private_ip: @private_ips.pop,
      ports: [
        [80, 8000],
        [443, 4430]
      ]
    }
  ]
}

# ssh -i ../vagrant_keys/id_rsa debian@192.168.10.98
# Update a Cookbook
# knife cookbook upload hello_chef_server
# Add a node and a recipe
# knife bootstrap 192.168.10.98 --ssh-user debian --sudo --identity-file ../vagrant_keys/id_rsa --node-name node1 --run-list 'recipe[hello_chef_server]'
# user password: 78yu[]OP
# Update a node (by executing it's recipe)
# knife ssh 192.168.10.98 'sudo chef-client' --manual-list --ssh-user debian --identity-file ../vagrant_keys/id_rsa







def ready_message(machine_name, ports, private_ip, public_ip, config, addMess='')
  config.vm.post_up_message = <<-MESSAGE.strip
    #{machine_name} is now live...
    Public IP: #{public_ip}
    Private IP: #{private_ip}
    Machine ports : #{ports.map{|p| p.first}.join(',')}
    Host ports : #{ports.map{|p| p.last}.join(',')}
    #{addMess}
  MESSAGE
end


def create_debian_user(config)
  # creates ssh keys if it doesn't exists
  @public_key_path = './vagrant_keys/id_rsa'
  unless File.exist?(@public_key_path)
    `ssh-keygen -t rsa -N "" -f #{@public_key_path}`
  end
  @public_key = IO.read("#{@public_key_path}.pub")
  # Create user debian with password 78yu[]OP
  config.vm.provision "shell", inline:<<-CREATE_USER
      apt-get update -y
      apt-get install -y htop git curl sudo emacs
      # Create user debian
      useradd debian --create-home -p lNqzZLROMo/AE
      # Add debian to group sudo
      usermod -a -G sudo debian
      # Add the public ssh key to debian
      mkdir /home/debian/.ssh && chmod 700 /home/debian/.ssh
      touch /home/debian/.ssh/authorized_keys && chmod 600 /home/debian/.ssh/authorized_keys
      echo '#{@public_key}' >> /home/debian/.ssh/authorized_keys
      chown -R debian:debian /home/debian/.ssh
      # echo 'PermitEmptyPasswords no' >> /etc/ssh/sshd_config
      # echo 'PasswordAuthentication no' >> /etc/ssh/sshd_config
      systemctl restart ssh || service ssh restart
  CREATE_USER
  @public_key=nil # no need to keep it in memory
end

















def create_worksation(machine_name, ports, private_ip, public_ip, config)
  config.vm.box = 'ubuntu/trusty64'
  config.vm.communicator = 'ssh' # 'winrm' for windows
  config.vm.hostname = machine_name
  config.vm.network 'public_network', ip: public_ip
  config.vm.network 'private_network', ip: private_ip, virtualbox__intnet: true
  ports.each do |p|
    config.vm.network "forwarded_port", guest: p.first, host: p.last
  end
  config.vm.provider 'virtualbox' do |vb|
    vb.gui = false
    vb.name = machine_name
    vb.memory = 512
    vb.cpus = 2
    vb.customize ['modifyvm', :id, '--cpuexecutioncap', '20']
    vb.customize ['modifyvm', :id, '--natdnsproxy1', 'on']
  end
  config.vm.synced_folder '.', '/vagrant',
                          owner: 'vagrant',
                          group: 'vagrant'#,
                          # type: 'rsync',
                          # rsync__exclude: ['.git/', 'chef-server-core_12.2.0-1_amd64.deb'],
                          # rsync__verbose: true,
                          # rsync__auto: true
  config.vm.provision "shell", inline:<<-CHEFDK
    apt-get update -y
    apt-get install -y htop git curl sudo
    wget -nv https://opscode-omnibus-packages.s3.amazonaws.com/ubuntu/12.04/x86_64/chefdk_0.8.1-1_amd64.deb
    dpkg -i chefdk_0.8.1-1_amd64.deb
    apt-get install -f
  CHEFDK
  ready_message(machine_name, ports, private_ip, public_ip, config)
end

















def create_chefserver(machine_name, ports, private_ip, public_ip, config)
  create_debian_user(config)
  config.vm.box = 'ubuntu/trusty64'
  config.vm.communicator = 'ssh' # 'winrm' for windows
  config.vm.hostname = public_ip.gsub(/\./, '-')
  config.vm.network 'public_network', ip: public_ip
  config.vm.network 'private_network', ip: private_ip, virtualbox__intnet: true
  ports.each do |p|
    config.vm.network "forwarded_port", guest: p.first, host: p.last
  end
  config.vm.provider 'virtualbox' do |vb|
    vb.gui = false
    vb.name = machine_name
    vb.memory = 4096
    vb.cpus = 2
    vb.customize ['modifyvm', :id, '--cpuexecutioncap', '50']
    vb.customize ['modifyvm', :id, '--natdnsproxy1', 'on']
  end
  config.vm.synced_folder '.', '/vagrant', disabled: true
  unless File.exist?('./chef-server-core_12.2.0-1_amd64.deb')
    `wget -nv https://web-dl.packagecloud.io/chef/stable/packages/ubuntu/trusty/chef-server-core_12.2.0-1_amd64.deb`
  end
  config.vm.provision "file", source: "./chef-server-core_12.2.0-1_amd64.deb", destination: "chef-server-core_12.2.0-1_amd64.deb"
  config.vm.provision "shell", inline:<<-CHEFSERVER
    apt-get update -y
    locale-gen UTF-8
    debconf-set-selections <<< "postfix postfix/mailname string cherfserver.com"
    debconf-set-selections <<< "postfix postfix/main_mailer_type string 'Internet Site'"
    apt-get install -y htop git curl emacs postfix libfreetype6 libpng3 ntp sudo
    # wget -nv https://web-dl.packagecloud.io/chef/stable/packages/ubuntu/trusty/chef-server-core_12.2.0-1_amd64.deb
    dpkg -i chef-server-core_12.2.0-1_amd64.deb
    apt-get install -f
    mkdir /etc/chef-server/
    echo "#{<<-F.gsub(/\n/, '\n')
      server_name = \\"#{public_ip}\\"
      api_fqdn server_name
      bookshelf['vip'] = server_name
      nginx['url'] = \\"https://\#{server_name}\\"
      nginx['server_name'] = server_name
      nginx['ssl_certificate'] = \\"/var/opt/opscode/nginx/ca/\#{server_name}.crt\\"
      nginx['ssl_certificate_key'] = \\"/var/opt/opscode/nginx/ca/\#{server_name}.key\\"
      F
    }" > /etc/opscode/chef-server.rb
    chef-server-ctl reconfigure
    echo 'Installing management console.........'
    chef-server-ctl install opscode-manage
    chef-server-ctl reconfigure
    opscode-manage-ctl reconfigure
    echo 'Installing reporting feature.........'
    chef-server-ctl install opscode-reporting
    chef-server-ctl reconfigure
    opscode-reporting-ctl reconfigure
    echo 'Creating the admin account .........'
    #chef-server-ctl user-create username name lastname email             password --filename ADMIN_USER_NAME.pem
    chef-server-ctl  user-create admin    it   admin    admin@example.com admin    --filename admin.pem
    echo 'The admin account and organization have been created.'
  CHEFSERVER
  ready_message(machine_name, ports, private_ip, public_ip, config, <<-TEXT
    Chef server now accessible at https://127.0.0.1:#{p.last.to_s}/
    Admin username => admin
    Admin password => admin
  TEXT
  )
end




















def create_machine(machine_name, ports, private_ip, public_ip, config)
  create_debian_user(config)
  # config.vm.provision "chef_client" do |chef|
  #   chef.node_name = machine_name
  # end
  config.vm.communicator = 'ssh' # 'winrm' for windows
  config.vm.hostname = public_ip.gsub(/\./, '-')
  config.vm.network 'public_network', ip: public_ip
  config.vm.network 'private_network', ip: private_ip, virtualbox__intnet: true
  ports.each do |p|
    config.vm.network "forwarded_port", guest: p.first, host: p.last
  end
  config.vm.provider 'virtualbox' do |vb|
    vb.gui = false
    vb.name = machine_name
    vb.memory = 1024
    vb.cpus = 2
    vb.customize ['modifyvm', :id, '--cpuexecutioncap', '30']
    vb.customize ['modifyvm', :id, '--natdnsproxy1', 'on']
  end
  config.vm.synced_folder '.', '/vagrant', disabled: true
  ready_message(machine_name, ports, private_ip, public_ip, config)
end




















Vagrant.configure(@vagrant_version) do |operator|
  operator.vm.box = 'debian/jessie64'
  operator.vm.box_check_update = true
  operator.vm.box_version = '>= 0' # latest
  # operator.vm.provision "chef_client" do |chef|
  #   chef.chef_server_url = "https://api.chef.io/organizations/yourorganisation"
  #   chef.client_key = "chef-repo/.chef/admin.pem"
  #   chef.validation_key_path = "chef-repo/.chef/admin-validator.pem"
  #   chef.validation_client_name= "admin-validator"
  #   chef.delete_node = true
  #   chef.delete_client = true
  # end

  station = @machines[:chefworkstation]
  operator.vm.define station[:name] do |config|
    create_worksation(station[:name], station[:ports], station[:private_ip], station[:public_ip], config)
  end

  @machines[:nodes].each do |machine|
    operator.vm.define machine[:name] do |config|
      create_machine(machine[:name], machine[:ports], machine[:private_ip], machine[:public_ip], config)
    end
  end

  server = @machines[:chefserver]
  operator.vm.define server[:name] do |config|
    create_chefserver(server[:name], server[:ports], server[:private_ip], server[:public_ip], config)
  end

end
