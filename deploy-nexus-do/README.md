# Deploy Nexus on Digital Ocean


Translates the bash commands to an Ansible playbook. The source code for provisioning the DO droplet can be found [here](https://github.com/flaviassantos/terraform-projects/tree/master/simple-digital-ocean-droplet).

```bash
apt update
apt install openjdk-8-jre-headless
apt install net-tools

cd /opt
wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz
tar -zxvf latest-unix.tar.gz

adduser nexus
chown -R nexus:nexus nexus-3.28.1-01
chown -R nexus:nexus sonatype-work

vim nexus-3.28.1-01/bin/nexus.rc
run_as_user="nexus"

su - nexus
/opt/nexus-3.28.1-01/bin/nexus start

ps aux | grep nexus
netstat -lnpt
```

### Modules used

- [ansible.builtin.apt module](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/apt_module.html)