# -*- mode: ruby -*-
# vi: set ft=ruby :
REQUIRED_PLUGINS = [
  "vagrant-proxyconf",
  "vagrant-vbguest",
  "vagrant-triggers",
  "vagrant-omnibus"]

REQUIRED_PLUGINS.each do |plugin|
  unless Vagrant.has_plugin?(plugin)
    raise "This vagrantfile uses the " + plugin + " plugin which is missing"
  end
end

Vagrant.configure(2) do |config|
  #---------------------------------
  # Select base box
  #---------------------------------
  config.vm.box = "ubuntu/trusty64"

  #---------------------------------
  # Customise base box
  #---------------------------------
  config.vm.provider "virtualbox" do |vb|
    vb.gui = true
    vb.memory = "4096"
    vb.cpus = 2
    vb.customize ["modifyvm", :id, "--vram", "256"]
    vb.customize ["setextradata", :id, "GUI/MaxGuestResolution", "any"]
    vb.customize ["setextradata", :id, "CustomVideoMode1", "1920x1080x32"]
    # Specific to a Windows host
    # vb.customize ["modifyvm", :id, "--audio", "dsound", "--audiocontroller", "hda"] # choices: hda sb16 ac97
  end

  # Configure the guest's proxy environment variables to point to CNTLM on the host
  # if Vagrant.has_plugin?("vagrant-proxyconf") 
  #   config.proxy.http     = "http://10.0.2.2:3128"
  #   config.proxy.https    = "https://10.0.2.2:3128"
  #   config.proxy.no_proxy = "localhost,127.0.0.1"
  # end

  # Install VirtualBox guest additions 
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
    config.vbguest.no_remote = true
  end

  if Vagrant.has_plugin?("vagrant-omnibus")
  	# config.omnibus.chef_version = :latest
  	config.omnibus.chef_version = "12.3.0"
  end

  # Refer to bug #5199. Clearing sync folders help chef shared folders work better.
  if Vagrant.has_plugin?("vagrant-triggers")
    config.trigger.before [:up, :reload], stdout: true do 
      `rm .vagrant/machines/default/virtualbox/synced_folders`
    end  
    config.trigger.after [:halt], stdout: true do 
      `rm .vagrant/machines/default/virtualbox/synced_folders`
    end
  end
  #---------------------------------
  # Provision customised box
  #---------------------------------
  config.vm.provision "chef_solo" do |chef|
    chef.cookbooks_path = "cookbooks"
    chef.add_recipe "eclipse"
    chef.json = {
      "java" => {
        "install_flavor" => "openjdk",
        "jdk_version" => "7",
        "openjdk_packages" => ["openjdk-7-jdk", "openjdk-7-jre-headless"],
        "set_etc_environment" => true,
        "accept_license_agreement" => true
      },
      "eclipse" => {
        "version" => "mars",
        "release_code" => "R",
        "arch" => "x86_64",
        "suite" => "cpp",
        "os" => "linux-gtk",
        "plugins" => [
          # {"http://download.eclipse.org/releases/luna" => "org.eclipse.cdt.feature.group"},
          # {"http://download.eclipse.org/releases/luna" => "org.eclipse.cdt.managedbuilder.llvm.feature.group"},
          # {"http://download.eclipse.org/releases/luna" => "org.eclipse.cdt.build.crossgcc.feature.group"},
          # {"http://download.eclipse.org/releases/luna" => "org.eclipse.cdt.debug.gdbjtag.feature.group"},
          # {"http://download.eclipse.org/releases/luna" => "org.eclipse.cdt.debug.ui.memory.feature.group"},
          
          # {"http://download.eclipse.org/releases/luna" => "org.eclipse.jdt.feature.group"},
          # {"http://download.eclipse.org/releases/luna" => "org.eclipse.m2e.feature.feature.group"},

          # {"http://download.eclipse.org/releases/luna" => "org.eclipse.wst.xml_ui.feature.feature.group"},
          
          # {"http://download.eclipse.org/releases/luna" => "org.eclipse.egit.feature.group"},
          # {"http://download.eclipse.org/releases/luna" => "org.eclipse.team.svn.feature.group"},
        ]
      }
    }
  end
  config.vm.provision "shell", inline: $install_gui
  config.vm.provision "shell", inline: $install_armhf_build_tools
  # config.vm.provision "shell", inline: $install_build_essentials
  config.vm.provision "shell", inline: $add_eclipse_to_launcher
end

#---------------------------------
# Shell Scripts
#---------------------------------
$install_gui = <<SCRIPT
echo "Updating apt sources"
if ! grep -q '# Mirror sources' /etc/apt/sources.list; then
  sudo sed -i -e '1ideb mirror://mirrors.ubuntu.com/mirrors.txt trusty main restricted universe multiverse' /etc/apt/sources.list
  sudo sed -i -e '1ideb mirror://mirrors.ubuntu.com/mirrors.txt trusty-updates main restricted universe multiverse' /etc/apt/sources.list
  sudo sed -i -e '1ideb mirror://mirrors.ubuntu.com/mirrors.txt trusty-backports main restricted universe multiverse' /etc/apt/sources.list
  sudo sed -i -e '1ideb mirror://mirrors.ubuntu.com/mirrors.txt trusty-security main restricted universe multiverse' /etc/apt/sources.list
  sudo sed -i -e '1i# Mirror sources' /etc/apt/sources.list
  sudo apt-get update
fi
echo "Setting debconf-set-selections"
sudo debconf-set-selections <<EOF
  gdm     shared/default-x-display-manager      select    lightdm
  lightdm shared/default-x-display-manager      select    lightdm
EOF
echo "Installing lightdm"
sudo DEBIAN_FRONTEND="noninteractive" apt-get install -y lightdm
echo "Installing ubuntu-gnome-desktop"
sudo DEBIAN_FRONTEND="noninteractive" apt-get install -y ubuntu-gnome-desktop=0.32
echo "Reconfiguring to lightdm"
sudo DEBIAN_FRONTEND="noninteractive" dpkg-reconfigure lightdm
SCRIPT

$install_armhf_build_tools = <<SCRIPT
  export DEBIAN_FRONTEND="noninteractive"
  echo "INSTALLING ARMHF BUILD TOOLS"
  apt-get -q -y install libc6-armhf-cross
  apt-get -q -y install libc6-dev-armhf-cross
  apt-get -q -y install binutils-arm-linux-gnueabihf
  apt-get -q -y install linux-libc-dev-armhf-cross
  apt-get -q -y install libstdc++6-armhf-cross
  apt-get -q -y install gcc-4.7-arm-linux-gnueabihf
  apt-get -q -y install g++-4.7-arm-linux-gnueabihf
SCRIPT

$install_build_essentials = <<SCRIPT
  export DEBIAN_FRONTEND="noninteractive"
  echo "INSTALLING BUILD ESSENTIALS"
  apt-get install -q -y build-essential
SCRIPT

$add_eclipse_to_launcher = <<SCRIPT
  if [ ! -f /usr/share/applications/eclipse-mars.desktop ]; then
    cat > /usr/share/applications/eclipse-mars.desktop <<EOL
      [Desktop Entry]
      Type=Application
      Encoding=UTF-8
      Name=Eclipse Mars
      Comment=IDE for C/C++/java development
      Icon=/usr/local/eclipse-mars/icon.xpm
      Exec=/usr/local/eclipse-mars/eclipse
      Terminal=false
      Categories=ide;
EOL
  fi
SCRIPT