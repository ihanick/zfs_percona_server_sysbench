# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  #config.vm.box = "ubuntu/bionic64"
  config.vm.box = "ubuntu/xenial64"
  config.ssh.forward_agent = true
  config.hostmanager.enabled = true
  config.vm.hostname = 'main.vm'

  config.vm.provider :virtualbox do |vb|
  # Boot with GUI mode
    #vb.gui = true
    vb.memory = 4096

    extra_disks = [
      {:path => "./vagrant/sdb.vdi", :size => 2048*1024},
      {:path => "./vagrant/sdc.vdi", :size => 2048*1024},
      {:path => "./vagrant/sdd.vdi", :size => 2048*1024},
      {:path => "./vagrant/sde.vdi", :size => 2048*1024},
      {:path => "./vagrant/sdf.vdi", :size => 2048*1024},
      {:path => "./vagrant/sdg.vdi", :size => 2048*1024},
      {:path => "./vagrant/sdh.vdi", :size => 2048*1024},
      {:path => "./vagrant/sdi.vdi", :size => 2048*1024},
      {:path => "./vagrant/sdj.vdi", :size => 2048*1024},
      {:path => "./vagrant/sdk.vdi", :size => 2048*1024},
    ]

    # Adds block devices to build test zpools
    extra_disks.each_with_index do |disk, index|
      unless File.exist?(disk[:path])
        vb.customize ["createhd", "--filename", disk[:path], "--size", disk[:size]]
        vb.customize ["storageattach", :id, "--storagectl", "SCSI", "--port", index+2, "--device", 0, "--type", "hdd", "--medium", disk[:path]]
      end
    end
  end

  # Run provisioning shell script
  config.vm.provision :shell, privileged: true, inline: <<-SHELL
apt-get update
apt-get install -y zfsutils-linux wget parted
for dsk in /dev/sd[c-k] ; do parted $dsk mklabel gpt ; done
zpool create -m none -o ashift=12 local raidz /dev/sd[c-k]
zfs set atime=off local
zfs set dedup=off local
zfs create local/mysqldata
zfs create local/innodblogs
mkdir /srv/mysqldata /srv/innodblogs
zfs set mountpoint=/srv/mysqldata local/mysqldata
zfs set mountpoint=/srv/innodblogs local/innodblogs
cd /root
echo "percona-server-server-5.7 percona-server-server-5.7/root-pass password secret" | debconf-set-selections
echo "percona-server-server-5.7 percona-server-server-5.7/re-root-pass password secret" | debconf-set-selections
echo "percona-server-server-5.7 percona-server-server-5.7/re-pass password secret" | debconf-set-selections
echo "percona-server-server-5.7 percona-server-server-5.7/root-pass-mismatch string" | debconf-set-selections
echo "percona-server-server-5.7 percona-server-server-5.7/remove-data-dir boolean true" | debconf-set-selections
echo "percona-server-server-5.7 percona-server-server-5.7/data-dir string " | debconf-set-selections
wget https://repo.percona.com/apt/percona-release_0.1-4.$(lsb_release -sc)_all.deb
dpkg -i percona-release_0.1-4.$(lsb_release -sc)_all.deb
apt-get update
apt-get install -y percona-server-server-5.7 sysbench-tpcc
sleep 5
while pgrep -f initialize ; do sleep 1 ; done
mysqladmin -psecret -w ping
systemctl stop mysql.service
mkdir /srv/mysqldata/mysql /srv/innodblogs/innodblogs
chown -R mysql:mysql -R /srv/mysqldata/mysql /srv/innodblogs/innodblogs
mv /var/lib/mysql/ib_logfile* /srv/innodblogs/innodblogs/
mv /var/lib/mysql/* 
echo "innodb_log_group_home_dir=/srv/innodblogs/innodblogs" >> /etc/mysql/percona-server.conf.d/mysqld.cnf
echo "datadir=/srv/mysqldata/mysql" >> /etc/mysql/percona-server.conf.d/mysqld.cnf
echo "innodb_buffer_pool_size=2G" >> /etc/mysql/percona-server.conf.d/mysqld.cnf
echo "innodb_log_file_size=1G" >> /etc/mysql/percona-server.conf.d/mysqld.cnf
echo "innodb_flush_log_at_trx_commit=2" >> /etc/mysql/percona-server.conf.d/mysqld.cnf
systemctl start mysql.service
mysqladmin -psecret -w ping
mysqladmin -psecret create sbtest
# sysbench --threads=2 --time=60 --db-driver=mysql --mysql-user=root --mysql-password=secret oltp_insert --tables=2 --table_size=100000 prepare
  SHELL

end
