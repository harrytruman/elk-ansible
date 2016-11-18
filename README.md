# elk-ansible
ELK stack deployed through Ansible

These playbooks install/configure Elasticsearch, Logstash, Kibana, and Filebeat. Can be ran independently, or all at once from elk.yaml.

Pre-configuration
  1. Add Elasticsearch yum repo.

Elasticsearch
  1. Install JDK and Elasticsearch
  2. Bind to default IP address, port 9200.
  3. Set JVM min/max memory to 50% of system RAM.
  4. Increase vm.max_map_count to 262144.
  5. Start service and validate that Elasticsearch is up and available.
  
Kibana
  1. Install Kibana
  2. Bind to default IP address, port 5601.
  3. Assign Elasticsearch URL to default.ip:9200.
  4. Start service.
  
Logstash
  1. Install JDK and Logstash.
  2. Place template to set input as port 5044 and output to Elasticsearch at default.ip:9200.
  3. Start service and wait for it become available.
  
Filebeat
  1. Install Filebeat.
  2. Place template with some default log locations and output to logstash at default.ip:5044
  3. Start service.
