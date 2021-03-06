# My version of zabbix aws monitoring

Why is it unique?

Zabbix agent automatically connects to cloudwatch and discovers AWS services (EC, EC2, RDS, ASG, DynamoDB) and starts monitoring "out of the box". It's not perfect, templates require some tuning and adjustments.
![](https://github.com/aliaskov/dockerized-zabbix/raw/master/main_dash.png)

Services:
1. DB
2. Zabbix server
3. Zabbix web interface
4. Zabbix agent with scripts, based on original zabbix/zabbix-agent alpine image

Docker-compose file use 4 images:
1. mysql:5.7
2. zabbix/zabbix-server-mysql:latest
3. zabbix/zabbix-web-nginx-mysql:latest
4. aliaskov/aws-zabbix-agent:latest

Ports:
1. 3306/tcp for DB conections
2. 10051/tcp Zabbix checks
3. 8080/tcp For Web Interface

Mounts:
1. DB data files mysql_data/ on host are mounted on container's /var/lib/mysql
2. Custom py scripts  from agent_scripts/ are mounted on container's  /etc/zabbix/zabbix_scripts
3. Config files for scripts  from agent_conf/ are mounted on container's  /etc/zabbix/zabbix_agentd.d

AWS permissions (CloudFormation template creates and assigns role with listed policies to EC2 instance, where this stack runs):
1. Cloudwatch readonly
2. EC2 readonly
3. RDS readonly (if RDS monitoring is needed)
4. DynamoDB readonly (if DynamoDB monitoring is needed)
5. AmazonElastiCacheReadOnlyAccess  (if AmazonElastiCache monitoring is needed)

Usage

0. Try using CloudFormation template to launch EC2, attach EBS and create Zabbix role:
AWS - CloudFormation - Create Stack - Upload a template to Amazon S3 - aws-cfn-ec2-zabbix-template.json
![](https://github.com/aliaskov/dockerized-zabbix/raw/master/CFN.png)

1. CloudFormation template adds userdata in order to pull docker compose file to start stack.
Check /var/log/user-data.log, fix possible issues and start stack manually if needed:
 docker-compose -f zabbix-docker-compose.yml up -d

2. Import xml files (from templates dir) to zabbix web frontend, using web interface : Configuration - Templates - Import


Don't forget to delete all existing templates, and disconnect zabbix server from default linux PASSIVE template.

![](https://github.com/aliaskov/dockerized-zabbix/raw/master/templates.png)

Enable zabbix server host and wait for aws services discovery
![](https://github.com/aliaskov/dockerized-zabbix/raw/master/hosts.png)
zabbix - is automatically discovered host, which is a docker container with zabbix agent.
Zabbix server - is a default entry. Make sure that AWS Services template is applied on it

3. Create auto-registration actions to make "auto discovery" work:
![](https://github.com/aliaskov/dockerized-zabbix/raw/master/actions.png)

Alternative way of importing templates:
Modify DB/data.sql which contains default items,triggers, etc. and it is a part of zabbix/zabbix-server-mysql container (/usr/share/doc/zabbix-server-mysql/data.sql) and build your own zabbix-server-mysql image.
Almost empty DBdump is in DB/MyzabbixDump.sql

One more way or importing templates:
https://www.zabbix.com/documentation/3.2/manual/api
