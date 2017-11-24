# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|

  ## boxの指定
  # bento Pre-built Boxes ( https://github.com/chef/bento )
  config.vm.box = "bento/centos-7.4"

  ## vagrantプラグインが未導入の場合はインストールする
  # - vagrant-hostsupdater
  # - vagrant-vbguest
  # - vagrant-winnfsd
  #
  required_plugins = %w(vagrant-hostsupdater vagrant-vbguest vagrant-winnfsd)
  plugins_to_install = required_plugins.select { |plugin| not Vagrant.has_plugin? plugin }
  if not plugins_to_install.empty?
    puts "Installing plugins: #{plugins_to_install.join(' ')}"
    if system "vagrant plugin install #{plugins_to_install.join(' ')}"
      exec "vagrant #{ARGV.join(' ')}"
    else
      abort "Installation of one or more plugins has failed. Aborting."
    end
  end

  ## vagrant-cachier プラグイン設定
  # https://github.com/fgrehm/vagrant-cachier
  if Vagrant.has_plugin?("vagrant-cachier")
    config.cache.scope = :box
  end

  ## vagrant-winnfsd プラグイン設定
  # https://github.com/winnfsd/vagrant-winnfsd
  # ディレクトリマウント時の guest OS側の所有者IDを指定
  # デフォルトはrootでマウントされる。
  # vagrant ユーザでマウントしたい場合は `vagrant ssh`でゲストOSに入り `id -u vagrant` などで調べる。BOXにより異なる。
  # 設定変更時はホストOSのWindows側で winnfsd.exe プロセスを一度KILLして再起動すれば反映される。
  # - プロセス単体で停止: `taskkill /f /im winnfsd.exe`
  # - vagrant haltと合わせて停止 : `vagrant halt && taskkill /f /im winnfsd.exe`
  if Vagrant.has_plugin?("vagrant-winnfsd")
    config.winnfsd.uid = 500
    config.winnfsd.gid = 500
  end

  ## vagrant-vbguest プラグイン設定
  # https://github.com/dotless-de/vagrant-vbguest
  if Vagrant.has_plugin?("vagrant-vbguest")
    config.vbguest.auto_update = false
    config.vbguest.no_remote = true
  end

  ### node設定
  config.vm.define "develop.vm" do |node|

    ## vagrant-hostsupdater で ホストOSの hostsへホスト名を追加
    # https://github.com/cogitatio/vagrant-hostsupdater
    node.vm.network "private_network", ip: "192.168.56.10"
    node.vm.hostname = "vagrant.localhost"
    node.hostsupdater.aliases = ["test01.localhost", "test02.localhost"]

    # /vagrantディレクトリをNFS指定でマウント
    config.vm.synced_folder ".", "/vagrant", type: "nfs"

    # ansible_local でのセットアップ
    # https://www.vagrantup.com/docs/provisioning/ansible_local.html
    node.vm.provision "ansible_local" do |ansible|
      ansible.playbook = "ansible_local/site.yml"
    end

    # Vagrant provision 時に docker-compose buildを実行
    node.vm.provision "shell", inline: "/usr/local/bin/docker-compose -f /vagrant/docker/docker-compose.yml build"

    # Vagrant up 時に docker-compose up -d を起動
    node.vm.provision "shell",run: "always", inline: "echo wait 10.sec && sleep 10"
    node.vm.provision "shell",run: "always", inline: "/usr/local/bin/docker-compose -f /vagrant/docker/docker-compose.yml up -d 1>&2"

  end
end
