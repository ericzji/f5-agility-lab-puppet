Module 1 – Configuring and Running Puppet BIG-IP Module 
=============================================

.. toctree::
   :maxdepth: 1
   :glob:

To start the deployment, install the Puppet master and create a proxy system able to run the Puppet agent. In addition, you must install all the dependencies, including iControl gem and Faraday gem into the Puppet Ruby environment on the proxy host (Puppet agent). 

In this lab, the puppet master and proxy have already been installed and provisioned.

ssh to Puppet Master from Jumphost Terminal

	.. code::

		ssh scs@10.1.1.101

#.  **Download F5 module**

	.. NOTE:: In this lab, F5 module has already been downloaded with the latest version. You may only perform this step to get the updated version.

	Download the F5 module from https://github.com/f5devcentral/f5-puppet. 

	.. code::

	   scs@master:/etc/puppetlabs/code/modules/f5_rest$ ls
	   CHANGELOG.md  examples  Gemfile  lib  LICENSE  manifests  metadata.json  NOTICE  pkg  Rakefile  README.md  spec

#. **Create a device.conf File**

	.. NOTE:: In this lab, you may find the configuration file is already created, and you just have to verify the configuration.

	Before you can use the F5 module, you must create a device.conf file in the Puppet configuration directory (/etc/puppetlabs/puppet) on the Puppet proxy:
	
	.. code::

	   [bigip1]
	   type f5
	   url https://admin:admin@10.1.1.246

	In the above example, admin:admin@10.1.1.246 refers to Puppet's login for the F5 device: <USERNAME>:<PASSWORD>@<IP ADDRESS OF BIGIP>.

#. **Classify Your Nodes on the Puppet Master**

	Next, you enter the configuration in the relevant class statement or node declaration in your site.pp. 
	Following is a sample Puppet manifest file (site.pp) for configuring a VLAN on the BIG-IP platform::

		scs@master:/etc/puppetlabs/code/environments/production/manifests$ pwd
		/etc/puppetlabs/code/environments/production/manifests

	.. Code::

		node bigip1 {
		f5_vlan { '/Common/test_vlan':
		      ensure                 => 'present',
		      auto_last_hop          => 'enabled',
		      cmp_hash               => 'src-ip',
		      description            => 'This is VLAN 10',
		      fail_safe              => 'enabled',
		      fail_safe_action       => 'restart-all',
		      fail_safe_timeout      => '90',
		      mtu                    => '1500',
		      sflow_polling_interval => '3000',
		      sflow_sampling_rate    => '4000',
		      source_check           => 'enabled',
		      vlan_tag               => '10',

		 }
		}


#. **Run puppet device**

	Running the *puppet device -v --user=root* command will have the device proxy node generate a certificate and apply your classifications to the F5 device.

	As shown below, all the tasks were completed successfully with no failures. 

	.. Code::

		$ sudo puppet device -v --user=root --trace
		Info: starting applying configuration to bigip1 at https://10.1.1.246:443
		Info: Retrieving pluginfacts
		Info: Retrieving plugin
		Info: Caching catalog for bigip1
		Info: Applying configuration version '1530306055'
		Notice: /Stage[main]/Main/Node[bigip1]/F5_vlan[/Common/test_vlan]/ensure: created
		Info: Node[bigip1]: Unscheduling all events on Node[bigip1]
		Notice: Applied catalog in 0.57 seconds

#. **Puppet resource to query a F5 device**

	Once you've established a basic configuration, you can explore the providers and their allowed options by running puppet resource <TYPENAME> for each type. (Note: You must have your authentification credentials in FACTER_url within your command, or puppet resource will not work.) 

	.. Code::

	$ FACTER_url=https://<USERNAME>:<PASSWORD>@<IP ADDRESS OF BIGIP> puppet resource f5_vlan

	.. Code::

		$ sudo FACTER_url=https://admin:admin@10.1.1.246 puppet resource f5_vlan
		f5_vlan { '/Common/test_vlan':
		  ensure                 => 'present',
		  auto_last_hop          => 'enabled',
		  cmp_hash               => 'src-ip',
		  dag_round_robin        => 'disabled',
		  description            => 'This is VLAN 10',
		  fail_safe              => 'enabled',
		  fail_safe_action       => 'restart-all',
		  fail_safe_timeout      => '90',
		  mtu                    => '1500',
		  sflow_polling_interval => '3000',
		  sflow_sampling_rate    => '4000',
		  source_check           => 'enabled',
		  vlan_tag               => '10',
		}

#. **Idempotency**

	All the Puppet F5 modules are idempotent, which means that tasks are executed only if the node state doesn’t match the configured or desired state. In other words, if the same manifest is run again, Puppet does not reconfigure these objects. 

	.. Code::

		$ sudo puppet device -v --user=root --trace
		Info: starting applying configuration to bigip1 at https://10.1.1.246:443
		Info: Retrieving pluginfacts
		Info: Retrieving plugin
		Info: Caching catalog for bigip1
		Info: Applying configuration version '1530558089'
		Notice: Applied catalog in 0.18 seconds


#. **Remove the configuration**

	Next, modify your site.pp to the following:

	.. Code::

		node bigip1 {
		f5_vlan { '/Common/test_vlan':
		      ensure                 => 'absent',

		 }
		}
	Re-run puppet device to delete the vlan:

	.. Code::

			$ sudo puppet device -v --user=root --trace
			Info: starting applying configuration to bigip1 at https://10.1.1.246:443
			Info: Retrieving pluginfacts
			Info: Retrieving plugin
			Info: Caching catalog for bigip1
			Info: Applying configuration version '1533924613'
			Notice: /Stage[main]/Main/Node[bigip1]/F5_vlan[/Common/test_vlan]/ensure: removed
			Info: Node[bigip1]: Unscheduling all events on Node[bigip1]
			Notice: Applied catalog in 0.75 seconds
			