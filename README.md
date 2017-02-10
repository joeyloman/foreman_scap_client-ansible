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

Go to the Policies section in the Satellite web UI under the Hosts menu. Create a new policy  named "rhel6" by clicking the "New Compliance Policy" button. Select the "SCAP content" and the "XCCDF Profile". The schedule settings are not important because we configure this in the playbook. Select the locations and organizations. The Hostgroups are also not important because we don't use this in the playbook. Save the policy and remember this as policy_id "1".

Repeat the same steps for the "rhel7" policy and remember this as policy_id "2".

## Playbook configuration

Edit the group\_vars/all file and configure the Satellite 6 or Capsule server hostname in the foreman\_proxy_server option, for example:

    foreman_proxy_server: satellite6.example.com
    foreman_proxy_port:   9090
    ca_file:              /etc/rhsm/ca/katello-server-ca.pem
    host_certificate:     /etc/pki/consumer/cert.pem
    host_private_key:     /etc/pki/consumer/key.pem

    satellite_version:    6.2

    # cron settings
    minute: "*"
    hour: "*"
    day: "*"
    weekday: "1"
    month: "*"

    openscap_policies_rhel6:
      - policy_id: 1
        profile_id: xccdf_org.ssgproject.content_profile_stig-rhel6-server-upstream
        content_path: /usr/share/xml/scap/ssg/content/ssg-rhel6-ds.xml
        download_path: /compliance/policies/1/content

    openscap_policies_rhel7:
      - policy_id: 2
        profile_id: xccdf_org.ssgproject.content_profile_stig-rhel7-server-upstream
        content_path: /usr/share/xml/scap/ssg/content/ssg-rhel7-ds.xml
        download_path: /compliance/policies/2/content

As you can see there are also two OpenSCAP policies defined, one RHEL6 and one for RHEL7. 

The policy\_id variable should match the one in the "Creating the policies in Satellite 6" section. The number in the download\_path should also match the policy\_id.

The profile\_id should match the selected profile you selected in the "Creating the policies in Satellite 6" section.

The content\_path variable contains the file path which is used by the client to store the policy files.

## Playbooks

### update\_scap_content.yml

This playbook installs, configures and run the OpenSCAP client. It can also be scheduled in Ansible Tower to run the client at certain timestamps.

### add\_scaprun\_to_cron.yml

This playbook adds the client runs to a cron.d configuration file. The cron execution settings can be configured in the group_vars/all file. This is the same approach as the default Satellite 6/puppet module implementation.
