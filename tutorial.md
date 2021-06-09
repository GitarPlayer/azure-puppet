# Setup for running terraform
In your terminal, use the Azure CLI tool to setup your account permissions locally:
    az login  
  
Initialize the azure provider:  
    terraform init

Plan and create the VMs:
    terraform plan -out plan && terraform apply 

You should see a similar output:
```Apply complete! Resources: 9 added, 0 changed, 0 destroyed.

Outputs:

public_ip_address = [
"X.X.X.X",
"X.X.X.X",
]
```

Grab one ip and connect to your server via ssh (the password is Welcome$21! if you haven't changed it in main.tf):  
    ssh tux@x.x.x.x



# Install the puppet server and agent
Add the Puppetlabs repo:  
    yum install https://yum.puppet.com/puppet-release-el-8.noarch.rpm -y

Check what is available within the Puppetlabs repo:
    yum list --disablerepo=* --enablerepo=puppet available

The output should resemble the following:
```
Last metadata expiration check: 0:09:31 ago on Wed Jun  9 10:59:43 2021.
Available Packages
pdk.x86_64                                                                             2.1.0.0-1.el8                                                                puppet
puppet-bolt.x86_64                                                                     3.9.2-1.el8                                                                  puppet
puppet-release.noarch                                                                  1.0.0-15.el8                                                                 puppet
puppet7-release.noarch                                                                 7.0.0-2.el8                                                                  puppet
puppetdb.noarch                                                                        7.3.1-1.el8                                                                  puppet
puppetdb-termini.noarch                                                                7.3.1-1.el8                                                                  puppet
```

installing the server will install the agent along. 


Set host record to puppet
    echo "127.0.0.1 puppet">> /etc/hosts 

Install puppet server and check version afterwards*:
    yum install puppetserver -y; exit && sudo -i puppet --version

The output should resemble the following:
```7.7.0```

* We log out and relogin to reexecute the login script, so puppet will be in the path. 


# Setup TimeSync with puppet
Puppet needs correct time for communication with agents. 

Install chrony package if not installed. We quote the manifest in single quotes:
    puppet apply -e 'package { "chrony": ensure => installed }'

Start chronyd and enable it
    puppet apply -e 'service { "chronyd": ensure => running, enable => true }'

The output should resemble the following:
```
Notice: Compiled catalog for vm-0.ma0z3b0xmbeuzduwooz1fwcpnd.ax.internal.cloudapp.net in environment production in 0.01 seconds
Notice: /Stage[main]/Main/Service[chronyd]/ensure: ensure changed 'stopped' to 'running'
Notice: Applied catalog in 0.31 seconds
```

If we run the same cmd again, we can see that puppet is idempotent. As chronyd is already running so puppet does not need to do anything.:
```
Notice: Compiled catalog for vm-0.ma0z3b0xmbeuzduwooz1fwcpnd.ax.internal.cloudapp.net in environment production in 0.01 seconds
Notice: Applied catalog in 0.06 seconds
```

# Enable puppetserver and puppet agent
Enable both servics check for open ports (TCP 8140 should be open):  
    systemctl enable --now puppetserver puppet
    ss -ntl

Check if puppet agent can connect to local server:  
    puppet agent -t

The output should resemble the following:
```Info: Using configured environment 'production'
Info: Retrieving pluginfacts
Info: Retrieving plugin
Info: Caching catalog for vm-0.ma0z3b0xmbeuzduwooz1fwcpnd.ax.internal.cloudapp.net
Info: Applying configuration version '1623236471'
Notice: Applied catalog in 0.03 seconds
```

# See the power of puppet
We now can use a module to quickly install, configure and start an apache web server:
    puppet module install puppetlabs/apache
    puppet apply -e "include apache"

When we check for open ports now we will see port 80 has been opened:
    ss -ntl

And curling localhost will show the default apache page being served:
    curl http://localhost:80

# Manifests in puppet
A manifest in puppet is a declarative script how to configure a resource. Their file endings is .pp. They can be applied locally or on the puppet server in the correct environment, the default one is production.

# Useful puppet config commands
Print all puppet config settings:  
    puppet config print

Print the path to the puppet config:  
    puppet config print config

Print config settings for master (puppet server) and for the production environment:  
    puppet config print manifest --section master --environment production

# Manifest files
The puppet agent can apply manifest by accecpting a path as parameter. If the path is a directory, then all the .pp files within the directory are executed in alphabetic order. 

This can be done with:
    puppet apply /path/to/manifest

# Puppet environments 
We can create multiple devs by simpling creating the right directory structure:
    mkdir -p /etc/puppetlabs/code/environments/dev/manifests

And then set the environment for all the agents for example:
    puppet config set environment dev --section=agent

# Agent Run
By default the agents runs every 30 minutes to get into its Desired State. You can see the interval with:  
    puppet config print runinterval

The output should ressemble the following:
```
1800
```

# Useful BASH Aliases
Change into the manifest directory:
    alias cdpp="cd /etc/puppetlabs/code/environments/production/manifests"  

or alternatively:  
    alias cdpp="cd $(puppet config print manifest)"

Add these line to the bottom of .bashrc:  
    echo 'alias cdpp="cd $(puppet config print manifest)"' >> ~/.bashrc





