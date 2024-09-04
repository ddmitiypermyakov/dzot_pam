MACHINES = {
  :"pam" => {
              :box_name => "ubuntu/jammy64",
              :cpus => 2,
              :memory => 1024,
			  #:net => [
           #["192.168.8.2", 2, "255.255.255.0", "mynet"],
              :ip => "192.168.57.10",
			  
            }
}
ENV['VAGRANT_SERVER_URL'] = 'https://vagrant.elab.pro'
Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    config.vm.synced_folder ".", "/vagrant", disabled: true
    #config.vm.network "private_network", ip: boxconfig[:ip]
	
    config.vm.define boxname do |box|
      box.vm.box = boxconfig[:box_name]
      box.vm.box_version = boxconfig[:box_version]
      box.vm.host_name = boxname.to_s
	    # boxconfig[:net].each do |ipconf|
        #box.vm.network("private_network", ip: ipconf[0], adapter: ipconf[1], netmask: ipconf[2], virtualbox__intnet: ipconf[3])

      #end
	  box.vm.network("private_network", ip: boxconfig[:ip], adapter: 2,netmask: "255.255.255.0", virtualbox__intnet: "mynet")

      box.vm.provider "virtualbox" do |v|
        v.memory = boxconfig[:memory]
        v.cpus = boxconfig[:cpus]
      end
      box.vm.provision "shell", inline: <<-SHELL
          sed -i 's/^PasswordAuthentication.*$/PasswordAuthentication yes/' /etc/ssh/sshd_config
          systemctl restart sshd.service
  	  SHELL
    end
  end
end

