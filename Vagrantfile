VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|

  config.vm.box = "geerlingguy/ubuntu1604"
  #config.vm.network :private_network, ip: "192.168.2.2"

  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "ansible-loops.yml"
    ansible.sudo = true

    # Uncomment to make Ansible use verbose output.
    # ansible.raw_arguments = [
    #   "-vvv"
    # ]
  end

end
