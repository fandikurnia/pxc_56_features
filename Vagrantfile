# -*- mode: ruby -*-
# vi: set ft=ruby :

# Assumes a box from https://github.com/jayjanssen/packer-percona

# This sets up 3 nodes with a common PXC, but you need to run bootstrap.sh to connect them.

require './lib/vagrant-common.rb'

# Our puppet config
puppet_config = {
	'innodb_buffer_pool_size' 	=> '128M',
	'innodb_log_file_size' 		=> '64M',
        "experimental_repo"             => "no",
        "percona_server_version"        => "56",
		"extra_mysqld_config" => "binlog_row_image = minimal
innodb_api_enable_binlog = ON
log-bin = cluster_log
log-slave-updates
gtid_mode = ON
enforce-gtid-consistency
",
}

# Node names and ips (for local VMs)
pxc_nodes = {
	'node1' => '192.168.70.2',
	'node2' => '192.168.70.3',
	'node3' => '192.168.70.4',
}

Vagrant.configure("2") do |config|
	config.vm.box = "centos-6_4-64_percona"
	config.ssh.username = "root"

	# Create all three nodes identically except for name and ip
	pxc_nodes.each_pair { |node, ip|
		config.vm.define node do |node_config|
			node_config.vm.hostname = node
			node_config.vm.network :private_network, ip: ip

			provider_aws( node_config, "PXC #{node}", 'm1.small') { |aws, override|
				aws.block_device_mapping = [
					{
						'DeviceName' => "/dev/sdb",
						'VirtualName' => "ephemeral0"
					}
				]
				provision_puppet( override, 'pxc.pp', 
					puppet_config.merge( 'datadir_dev' => 'xvdb' )
				)
			}

			provider_virtualbox( node_config, '256' ) { |vb, override|		
				provision_puppet( override, 'pxc.pp', 
					puppet_config.merge('datadir_dev' => 'dm-2')
				)

			}

		end
	}
end

