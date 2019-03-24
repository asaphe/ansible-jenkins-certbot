# Jenkins Role

Install Jenkins on a Ubuntu server.  

* OS: Ubuntu 16.04+  

**AWS ROUTE-53 & CERTBOT**

The assumption is that AWS credentials exists in the environment (IAM Instance role or any boto auth method)

## Role Vars

| VAR | Description |
|:---:|:-----------:|
|jenkins_dns_record|allows overriding of the DNS record used|
|jenkins_selfsigned|True/False if Certbot should generate a self-signed certificate for jenkins|
|certbot_cloudflare|True/False use cloudflare provider|
|certbot_route53|True/False use route53 provider|
|certbot_renew|True/False renew existing certificates|
|aws_key_id|AWS Credentials override|
|aws_secret_key|AWS Credentials override|
|letsencrypt_live_base_path|lets encrypt `live` directory location|
|plugins|a list of jenkins plugins to install (use the ID of each plugin)|
|cloudflare|a dict specifiying credentials for cloudflare|
|jenkins_pkg_name|jenkins package name including the version (if desired)|
|jenkins_pkg_state|jenkins package state (Ansible apt package - state)|
|java_pkg_name|java package name including the version (if desired)|
|java_pkg_state|java package state (Ansible apt package - state)|

>_pkg_name/state as strings to ease the usage of those variables from the cli

* Get a sorted list of plugins from Jenkins

```groovy
#!groovy
Jenkins.instance.pluginManager.plugins.each{
  plugin -> println ("${plugin.getShortName()}:${plugin.getVersion()}") }
```

## Role Tags

```text
jenkins.java
jenkins.install
jenkins.configure
jenkins.plugins
jenkins.nginx
jenkins.nginx.installation
jenkins.nginx.sites
jenkins.nginx.sites.certbot
jenkins.nginx.sites.standalone
```

## Plugins

[credentials-plugin](https://github.com/jenkinsci/credentials-plugin/blob/master/README.md)  
[plain-credentials-plugin](https://github.com/jenkinsci/plain-credentials-plugin/blob/master/README.md)  
[kubernetes-plugin](https://github.com/jenkinsci/kubernetes-plugin/blob/master/README.md)  
[kubernetes-credentials-plugin](https://github.com/jenkinsci/kubernetes-credentials-plugin/blob/master/README.md)  
[kubernetes-cli-plugin](https://github.com/jenkinsci/kubernetes-cli-plugin/blob/master/README.md)  
[kubernetes-cd-plugin](https://github.com/jenkinsci/kubernetes-cd-plugin/blob/dev/README.md)  
[kubernetes-pipeline-plugin](https://github.com/jenkinsci/kubernetes-pipeline-plugin/blob/master/readme.md)  
[github-branch-source-plugin](https://github.com/jenkinsci/github-branch-source-plugin)  

### Extra

[github-branch-source](https://go.cloudbees.com/docs/plugins/github-branch-source/)  
[jenkins-configuration](https://github.com/edx/jenkins-configuration)  

### JAVA_ARGS

1. Disable Jenkins setup Wizard UI  
    >Disable the wizard after a clean installation  
2. Set Java DNS TTL to 60 seconds  
    >The Java virtual machine (JVM) caches DNS name lookups. When the JVM resolves a hostname to an IP address, it caches the IP address for a specified period of time, known as the time-to-live (TTL).  
3. Disable Multicasting  
    >DNS Multi-cast logic is rarely used in Jenkins, but it always tries to start DNS Multicast on startup. It slows down the startup and usually leads to errors since Jenkins instances have no such permissions on default setups.  
4. Java Xmx (maximum Java heap size)  
    >-Xmx option changes the maximum Heap Space for the VM. java -Xmx1024m means that the VM can allocate a maximum of 1024 MB  
    >This value must a multiple of 1024 greater than 2MB. Append the letter k or K to indicate kilobytes, or m or M to indicate megabytes.  

[jvm-ttl](http://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/java-dg-jvm-ttl.html)  
[java-heap](https://support.cloudbees.com/hc/en-us/articles/204859670-Java-Heap-settings-best-practice)  
[java-permsize](https://support.cloudbees.com/hc/en-us/articles/204264000-Why-do-I-receive-java-lang-OutOfMemoryError-PermGen-space)  
[java-net](http://docs.oracle.com/javase/7/docs/technotes/guides/net/properties.html)  
[java-cache-ttl](https://stackoverflow.com/questions/29579589/whats-the-recommended-way-to-set-networkaddress-cache-ttl-in-elastic-beanstalk)  
[java_args](https://support.cloudbees.com/hc/en-us/articles/209715698-How-to-add-Java-arguments-to-Jenkins-)  
[aws-java-dev-guide](https://docs.aws.amazon.com/sdk-for-java/v2/developer-guide/welcome.html)  
[aws-dns-ttl](https://aws.amazon.com/articles/4035)  
[dns-multicast](https://issues.jenkins-ci.org/browse/JENKINS-50816)  
[jenkins-linux-bestpractice](https://support.cloudbees.com/hc/en-us/articles/115000486312-CJP-Performance-Best-Practices-for-Linux)

### Usage Examples

Install Jenkins using the default configuration (Self-signed certificate)

```shell
ansible-playbook jenkins-playbook.yml -i inventories/dev -u ansible --ssh-common-args "-i ~/.ssh/ansible-dev -o IPQoS=throughput -o ProxyCommand=ssh -W %h:%p -q -i ~/.ssh/id_rsa ${USER}@${BASTION}" -l 'tilix_dev_jenkins' -e env='dev'
```

Install Jenkins using a specific version with a customised DNS record, ensuring certbot with route53 is used

```shell
ansible-playbook jenkins-playbook.yml -i inventories/dev -u ansible --ssh-common-args "-i ~/.ssh/ansible-dev -o IPQoS=throughput -o ProxyCommand=ssh -W %h:%p -q -i ~/.ssh/id_rsa ${USER}@${BASTION}" -l 'jenkins_dev' -e env='dev' -e jenkins_dns_record='jenkins-dev.example.io' -e certbot_route53='True' -e jenkins_pkg_name='jenkins=2.150.3'
```

Install Jenkins plugins (Only)

```shell
ansible-playbook jenkins-playbook.yml -i inventories/dev -u ansible --ssh-common-args "-i ~/.ssh/ansible-dev -o IPQoS=throughput -o ProxyCommand=ssh -W %h:%p -q -i ~/.ssh/id_rsa ${USER}@${BASTION}" -e env='dev' --tags 'jenkins.plugins'
```
