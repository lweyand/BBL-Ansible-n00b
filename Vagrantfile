Vagrant.configure("2") do |config|

    # Auto install plugins
    required_plugins = %w( vagrant-vbguest )
    _retry = false
    required_plugins.each do |plugin|
        unless Vagrant.has_plugin? plugin
            system "vagrant plugin install #{plugin}"
            _retry=true
        end
    end
    
    if (_retry)
        exec "vagrant " + ARGV.join(' ')
    end

    config.vm.box = "debian/stretch64"
    config.vm.network "forwarded_port", guest: 80, host: 8080

    config.vm.provider "virtualbox" do |v|
        v.name = "Ansible Target"
        v.memory = 512
        v.cpus = 1
        v.customize [
            "modifyvm", :id,
            "--vram", "128",
            "--clipboard", "bidirectional"
        ]
    end
end

