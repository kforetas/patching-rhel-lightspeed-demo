managed
=========

This role "managed" is designed to create managed servers on AWS EC2, which are registered to Red Hat Lightspeed for demo purpose. Those servers will be registered to the Satellite , and also be installed a demo application (WordPress) and required packages.

Requirements
------------

Basically, the role assumes to setup RHEL servers using Red Hat Cloud Access Gold Images on AWS EC2. 

The tested environment:
- RHEL-9.7.0_HVM-20260120-x86_64-0-Hourly2-GP3
- RHEL-9.7.0_HVM-20260513-x86_64-0-Hourly2-GP3
- WordPress 6.3.7
- MySQL 8.0

Role Variables
--------------

var/main.yml includes the following pre-set variables. You should change them adequately.
- mysql_wp_user
- mysql_wp_database
- wp_archive_url
- wp_weblog_title
- wp_user_name
- wp_admin_password
- wp_allow_weak_pass
- wp_admin_email
- wp_blog_public

Also, the following variables should be supplied when using the role. In this project, upper playbooks are supposed to set these variables.
- rhsm_username
- rhsm_passwd
- mysql_root_passwd
- mysql_wp_passwd

Dependencies
------------

The following collections need to be installed beforehand. You can install those collections by using requirements.yml located in the project top directry.
- community.crypto
- community.mysql
- redhat.rhel_system_roles
- redhat.insights

Example Playbook
----------------

    - name: Configure dev server
      hosts: dev
      become: true
      gather_facts: true
      vars_prompt:
        - name: rhsm_username
          prompt: "What is your Red Hat login name?"
          private: false
        - name: rhsm_passwd
          prompt: "What is your Red Hat login password?"
        - name: mysql_root_passwd
          prompt: "Enter your MySQL root password for WordPress"
        - name: mysql_wp_passwd
          prompt: "Enter your MySQL user password for WordPress"

      roles:
        - managed

License
-------

MIT

Author Information
------------------

Yukiya Shimizu
https://github.com/yukshimizu
