.. The contents of this file are included in multiple topics.
.. This file should not be changed in a way that hinders its ability to appear in multiple documentation sets.


To run a resource at the start of the resource collection phase of the |chef client| run, set up a ``Chef::Resource`` object, and then call the method that runs the action.

**Update a package cache**

It is important to make sure that an operating system's package cache is up to date before installing packages, otherwise there may be references to versions that no longer exist. For example, on |debian| or |ubuntu| systems, the |apt| cache needs to be updated. Use code similar to the following:

.. code-block:: ruby

   e = execute "apt-get update" do
     action :nothing
   end
   
   e.run_action(:run)

where ``e`` is created as a ``Chef::Resource::Execute`` |ruby| object. The ``action`` attribute is set to ``:nothing`` so that the ``run_action`` method can be used to tell the |chef client| to run the specified command. The |cookbook apt| (for |debian| and |ubuntu|) and |cookbook pacman| (for |archlinux|) cookbooks can be used for this purpose. The preceding recipe can be placed at the top of a node's run list to ensure it is run before the |chef client| tries to install any packages.

**An anti-pattern**

Unfortunately, resources that are executed when the resource collection is being built cannot notify any resource that has yet to be added to the resource collection. For example:

.. code-block:: ruby

   execute "ifconfig"
   
   p = package 'vim-enhanced' do
     action :nothing
     notifies :run, "execute[ifconfig]", :immediately
   end
   p.run_action(:install)

In some cases, the better approach may be to install the package before the resource collection is built to ensure that it is available to other resources later on. Or, something like the following can be used:

.. code-block:: ruby

   p = package "foo" do
     #parameters
   end
   p.run_action(:install)
   
   if p.updated_by_last_action?
     #Call the resource that we want to "notify"  
   end 


