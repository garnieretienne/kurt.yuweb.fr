I need to define some good pratices before to renew my puppet architecture and redefine my modules.

Some documentations:

* http://projects.puppetlabs.com/projects/1/wiki/Puppet_Best_Practice
* http://plathrop.tertiusfamily.net/puppet/best-practice-draft.html
* http://docs.puppetlabs.com/guides/best_practices.html

Using Puppet
============

I use puppet to deploy, monitor and centrally configure services on my hosts. With it, i can easilly configure a host to be ready until i deploy an application on it. Puppet is not in charge of deploying applications or virtualhosts, it only manage service stack.

Puppet architecture
===================

I use three repositories managed by Git to administrate my network:

* a modules repository: it contain my services modules
* a manifest repository: it contain my host declarations
* a fileserver repository: it contain all my configurations files, generic and by-host configurations
