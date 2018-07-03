Lab – Puppet Workflow automation
-----------------------------------

.. TODO:: Needs lab description

This lab will teach you how to download the |bip| |ve| image to your system.


Task – Create your own module
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


.. Code::

	$ sudo puppet module list
	/etc/puppetlabs/code/environments/production/modules
	├── eric-workflow (v0.1.0)
	├── puppetlabs-apache (v1.6.0)
	├── puppetlabs-ciscopuppet (v1.0.0)
	├── puppetlabs-concat (v1.2.4)
	└── puppetlabs-stdlib (v4.8.0)
	/etc/puppetlabs/code/modules
	├── puppetlabs-f5 (v1.5.1)
	└── puppetlabs-f5 (v1.3.0)

	scs@master:/etc/puppetlabs/code/environments/production/modules$ sudo puppet module generate scs-bigip --skip-interview

	Notice: Generating module at /etc/puppetlabs/code/environments/production/modules/bigip...
	Notice: Populating templates...
	Finished; module generated in bigip.
	bigip/metadata.json
	bigip/Rakefile
	bigip/README.md
	bigip/tests
	bigip/tests/init.pp
	bigip/manifests
	bigip/manifests/init.pp
	bigip/Gemfile
	bigip/spec
	bigip/spec/classes
	bigip/spec/classes/init_spec.rb
	bigip/spec/spec_helper.rb


	scs@master:/etc/puppetlabs/code/environments/production/modules$ cd bigip/
	scs@master:/etc/puppetlabs/code/environments/production/modules/bigip$ ls
	Gemfile  manifests  metadata.json  Rakefile  README.md  spec  tests



Task – Create new resource (wrapper)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. Code::

	scs@master:/etc/puppetlabs/code/environments/production/modules/bigip$ cd manifests/
	scs@master:/etc/puppetlabs/code/environments/production/modules/bigip/manifests$ vi bigip_vip.pp

.. Code::

	define bigip::bigip_vip (
	$nodeip,
	$virtualip,
	){

	f5_node { "/Common/web_server_1":
	   ensure                          => 'present',
	   address                         => "$nodeip",
	   description                     => 'Web Server Node',
	   availability_requirement        => 'all',
	   health_monitors                 => ['/Common/icmp'],
	}

	f5_pool { "/Common/pool1":
	   ensure                          => 'present',
	   members                         => [
	        { name => '/Common/web_server_1', port => '80', },
	   ],
	   availability_requirement        => 'all',
	   health_monitors                 => ['/Common/http_head_f5'],
	}

	f5_virtualserver { "/Common/vs1":
	   ensure                          => 'present',
	   provider                        => 'standard',
	   default_pool                    => '/Common/pool1',
	   destination_address             => $virtualip,
	   destination_mask                => '255.255.255.255',
	   http_profile                    => '/Common/http',
	   service_port                    => '80',
	   protocol                        => 'tcp',
	   source                          => '0.0.0.0/0',
	   source_address_translation      => 'automap'
	}

	}	



Task – Puppet run to use new resource
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

site.pp

.. Code::

	node bigip1 {

	bigip::bigip_vip { "vs" :
	  nodeip   => '100.1.1.1',
	  virtualip => '1.1.1.1',
	}

	}

.. Code::

	$ sudo puppet device -v --user=root --trace
	Info: starting applying configuration to bigip1 at https://10.192.74.111:443
	Info: Retrieving pluginfacts
	Info: Retrieving plugin
	Info: Caching catalog for bigip1
	Info: Applying configuration version '1530656207'
	Notice: /Stage[main]/Main/Node[bigip1]/Bigip::Bigip_vip[vs]/F5_node[/Common/web_server_1]/ensure: created
	Notice: /Stage[main]/Main/Node[bigip1]/Bigip::Bigip_vip[vs]/F5_pool[/Common/pool1]/ensure: created
	Notice: /Stage[main]/Main/Node[bigip1]/Bigip::Bigip_vip[vs]/F5_virtualserver[/Common/vs1]/ensure: created


