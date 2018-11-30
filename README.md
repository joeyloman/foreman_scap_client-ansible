# Foreman\_scap_client Ansible

This playbook is a port of the foreman\_scap_client puppet module which is included in the Satellite 6 tools repository.

## Enable OpenSCAP in Satellite 6
To make this work in Satellite 6 you must enable the foreman openscap plugin on the Satellite 6 server by executing the following command:

    satellite-installer --enable-foreman-plugin-openscap
    
Now you should see a Compliance section in the Satellite web UI under the Hosts menu.

### Install the client on the Satellite 6 Server and Capsules
Install the foreman\_scap_client Puppet module on the Satellite 6 server and every Satellite 6 Capsule server:

    yum install puppet-foreman_scap_client

### Import the Puppet Classes
Go to the Classes section in the Satellite web UI under the Configure menu. Click on the "Import from \<satellite host>" button and import the "foreman\_scap_client" class "production" environment line.  

### Import the default SCAP content

If you want to use the default SCAP content you can add this by executing the following command on the Satellite 6 server:
 
    foreman-rake foreman_openscap:bulk_upload:default

### Creating the policies in Satellite 6

Go to the Policies section in the Satellite web UI under the Hosts menu. Create a new policy  named "rhel7" by clicking the "New Compliance Policy" button. Select the "SCAP content" and the "XCCDF Profile". The schedule settings are not important because we configure this in the playbook. Select the locations and organizations. The Hostgroups are also not important because we don't use this in the playbook. Save the policy.

### Creating the OpenSCAP viewer role and openscap service user

To query the Satellite API and view only the OpenSCAP policies and contents, create the following role:

    hammer role create --name "OpenSCAP viewer"
    hammer filter create --role "OpenSCAP viewer" --permissions "view_policies"
    hammer filter create --role "OpenSCAP viewer" --permissions "view_scap_contents"

And the service user:

    hammer user create --login "openscap" --auth-source-id 1 --password "VerySecretPassword" \
      --mail "openscap@localhost" --timezone "UTC" --locations "DC1" --organizations "Example"

    hammer user add-role --login "openscap" --role "OpenSCAP viewer"

## Playbook configuration

Edit the group\_vars/all file and configure the variables, for example:

    foreman_server:       satellite6.example.com
    foreman_proxy_server: satellite6-capsule.example.com
    foreman_proxy_port:   9090
    foreman_username:     openscap
    foreman_password:     VerySecretPassword
    ca_file:              /etc/rhsm/ca/katello-server-ca.pem
    host_certificate:     /etc/pki/consumer/cert.pem
    host_private_key:     /etc/pki/consumer/key.pem
    policy_name:          rhel7

    satellite_version:    6.4

    # cron settings
    minute: "*"
    hour: "*"
    day: "*"
    weekday: "1"
    month: "*"

NOTE: If you want to configure different OpenSCAP policies on different hosts, just override the "policy_name" variable in the host_vars for example.

## Playbooks

### update\_scap_content.yml

This playbook installs, configures and run the OpenSCAP client. It can also be scheduled in Ansible Tower to run the client at certain timestamps.

### add\_scaprun\_to_cron.yml

This playbook adds the client runs to a cron.d configuration file. The cron execution settings can be configured in the group_vars/all file or can be overridden in the host_vars. This is the same approach as the default Satellite 6/puppet module implementation.
