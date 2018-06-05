Module – Installing a BIG-IP F5 Module
=============================================

.. TODO:: Needs module description

In this module you will learn how to install a |f5| |bip| |ve| into your
environment.

.. toctree::
   :maxdepth: 1
   :glob:

#. One
#. Two
#. Three
#. Four

#. Download F5 module
.. code::
   scs@master:/etc/puppetlabs/code/modules/f5_rest$ ls
   CHANGELOG.md  examples  Gemfile  lib  LICENSE  manifests  metadata.json  NOTICE  pkg  Rakefile  README.md  spec

#. Create a device.conf File
Before you can use the F5 module, you must create a device.conf file in the Puppet configuration directory (either /etc/puppet or /etc/puppetlabs/puppet) on the Puppet proxy:
.. code::

   [bigip1]
   type f5
   url https://admin:admin@10.192.74.111

In the above example, admin:admin@10.192.74.111 refers to Puppet's login for the F5 device: <USERNAME>:<PASSWORD>@<IP ADDRESS OF BIGIP>.

#. Classify Your Nodes on the Puppet Master
Next, you enter the configuration in the relevant class statement or node declaration in your site.pp, <devicecertname>.pp node manifest, or some profiles::<profile_name> manifest file. Following is a sample Puppet manifest file (site.pp) for configuring an HTTP application on the BIG-IP platform:  
.. Code::
   node bigip1 {
	f5_node { '/Common/web_server_1':
	   ensure                          => 'present',
	   address                         => '10.1.20.11',
	   description                     => 'Web Server Node 1',
	   availability_requirement        => 'all',
	   health_monitors                 => ['/Common/icmp'],
	}->

	f5_node { '/Common/web_server_2':
	   ensure                          => 'present',
	   address                         => '10.1.20.12',
	   description                     => 'Web Server Node 2',
	   availability_requirement        => 'all',
	   health_monitors                 => ['/Common/icmp'],
	}->
   }


#. Run puppet device
Before running puppet device (command for Puppet Network Device ), there are no virtual servers, pools, or nodes configured on the BIG-IP device. Running the puppet device -v --user=root command will have the device proxy node generate a certificate and apply your classifications to the F5 device.

As shown below, all the tasks were completed successfully with no failures. 

.. Code::
   $ sudo puppet device -v --user=root –trace
   Info: starting applying configuration to bigip1 at https://10.192.74.111:443
   Info: Retrieving pluginfacts
   Info: Retrieving plugin
   Info: Caching catalog for bigip1
   Info: Applying configuration version '1498175426'
   Notice: /Stage[main]/Main/Node[bigip1]/F5_node[/Common/web_server_1]/ensure: created
   Notice: /Stage[main]/Main/Node[bigip1]/F5_node[/Common/web_server_2]/ensure: created
   Notice: /Stage[main]/Main/Node[bigip1]/F5_node[/Common/web_server_3]/ensure: created
   Notice: /Stage[main]/Main/Node[bigip1]/F5_pool[/Common/web_pool]/ensure: created
   Notice: /Stage[main]/Main/Node[bigip1]/F5_virtualserver[/Common/http_vs]/ensure: created
   Info: Node[bigip1]: Unscheduling all events on Node[bigip1]
   Notice: Applied catalog in 0.50 seconds


