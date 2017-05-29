Tomcat Server
========

Installs tomcat to the desired hosts.

Requirements
------------

RHEL/CentOS
Should work with Debian/Ubuntu but is untested


Role Variables
--------------
main.yml

```
# Install variables
# Make sure your versions are compatible http://tomcat.apache.org/whichversion.html
tomcat_version: '7.0.50' # 6.0.39 latest 6.x release
http_port: 8080
https_port: 8443

# Environment variables
catalina_base: /opt/apache-tomcat-{{ tomcat_version }}
catalina_home: /opt/apache-tomcat-{{ tomcat_version }}
catalina_tempdir: /opt/apache-tomcat-{{ tomcat_version }}/temp

# User management
tomcat_user: tomcat
tomcat_user_home: "{{ catalina_home }}"
tomcat_uid: '500'
manager_user: manager-gui
manager_password: managersecret
```
