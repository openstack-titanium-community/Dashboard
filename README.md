Dashboard
=========

Openstack horizon dashboard cookbook follows installation and configuration of cookbook OpenStack compute utilising chef-client and chef-server in a 3 node cluster. All of the OpenStack cookbooks can be found from https://github.com/stackforge. Also, list of questions and bugs regarding these cookbooks can be found from the respective site https://launchpad.net/openstack-chef. Below is a list of other useful web sites.

Cookbook OpenStack Dashboard
https://github.com/stackforge/cookbook-openstack-dashboard

Horizon Guidelines
http://docs.openstack.org/developer/horizon/

Requirements
Note Chef 0.10.0 or higher is required (for Chef environment use). Four nodes are required with a Linux environment, three nodes will be used to deploy horizon in a 3 node cluster while 1 node will have all dependant openstack services installed and configured.


###Dependencies

The following cookbooks are dependencies for cookbook OpenStack dashboard:

* apache2
* MySQL
* openstack-common

At a minimum cookbook keystone is required to have horizon up and running. Other installed OpenStack services installed will provide a wider visibility from a GUI perspective. These services should be installed prior to installeding horizon. In this guideline all OpenStack services are installed on 1 node.

###MySQL

Database port 3306 is required to be opened for openstack services to communicate with the database.
On the Ubuntu environment this can be done by two ways detailed below:
```bash
sudo ufw disable (disables the firewall)
sudo ufw allow 3306 (open up the specific port)
```
Second configuration includes changing the MySQL bind ip address this allows remotes connection to the database.

1. Open file /etc/mysql/my.cnf
2. Edit bind-address with IP address
3. Restart MySQL /etc/init.d/mysqld restart

###Create Database Horizon
The following steps create a database table ‘horizon’ and provide database privileges.

1.	Log in to MySQL and create a database called 'horizon'.
```bash
mysql –h host –u user –p
CREATE DATABASE horizon;
```

2. Create	a encrypted password for horizon database and grant privileges.
```html
select password('horizon');
+-------------------------------------------+
| password('horizon')                       |
+-------------------------------------------+
| *0BE3B501084D35F4C66DD3AC4569EAE5EA738212 |
+-------------------------------------------+
GRANT ALL PRIVILEGES ON *.* TO 'horizon'@'%' IDENTIFIED BY PASSWORD '*0BE3B501084D35F4C66DD3AC4569EAE5EA738212' 
WITH GRANT OPTION;
FLUSH PRIVILEGES;
```
3. Verify  MySQL connection
```bash
telnet 10.125.0.11 3306
mysql -u horizon -h 10.125.0.11 –p
```

##OpenStack Dashboard Configuration
use git to get clone of cookbook dashboard compute
```bash
git clone http://github.com/stackforge/cookbook-openstack-dashboard.git
```
Edit file default.rb located at /attributes/default.rb and add configuration to set this deployment in a developer mode, this will allow username and passwords to be default i.e. username horizon and password horizon.
```bash
default["openstack"]["developer_mode"] = true
```
Include the MySQL database IP address
```bash
default["openstack"]["db"]["dashboard"]["host"] = "10.49.117.250"
```

###Horizon Cluster Deployment
Upload the amended cookbook openstack-horizon to the chef server.
```bash
sudo knife cookbook import openstack-dashboard
```
Create a role for which can be tied to multiple nodes. The role in turn will include a run list or recipes to apply.
```bash
knife role create horizon_base
```
Amend the created role to include a run list of OpenStack dashboard recipes.
```bash
knife role edit horizon_base
```
```bash
{
  "name": "horizon_base",
  "description": "base configuration for openstack horizon",
  "json_class": "Chef::Role",
  "default_attributes": {
  },
  "override_attributes": {
  },
  "chef_type": "role",
"run_list": [
    "recipe[apache2]",
    "recipe[mysql::client]",
    "recipe[openstack-common]",
    "recipe[openstack-dashboard::server]"
  ],
  "env_run_lists": {
  }
}
```
Deploy our cluster with the 3 nodes with chef using the role created.
```bash
knife bootstrap 10.49.117.241 -x username -P password -r 'role[horizon_base]' --sudo
knife bootstrap 10.49.117.242 -x username -P password -r 'role[horizon_base]' --sudo
knife bootstrap 10.49.117.243 -x username -P password -r 'role[horizon_base]' –sudo
```
###Verification
A successful message will be displayed on completion of the bootstrap. The new nodes will be displayed in the chef server UI with all the recipes applied. The dasboard configuration is located under /etc/openstack_dashboard/local_settings.py. To test openstack horizon has been configured and installed correctly open your web browser and go to the IP address of each node.
