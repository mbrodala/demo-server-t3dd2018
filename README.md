Build a demo server, to server the sample project https://github.com/SomeBdyElse/typo3_jenkins_sample.

# Requires

* docker
* docker-compose
* htpasswd


# Setup demo server
```bash
# Use a random prefix
DEMO_SERVER_PREFIX=alpha

# Get an access token from digitalocean.com
export DIGITALOCEAN_ACCESS_TOKEN=2192e925c0e435251d1358504773d1ac2db0910891318d498995e5a37e8bbb5e
export DEMO_SERVER_DOMAIN=electro-snail.biz
export DEMO_SERVER_HOSTNAME=${DEMO_SERVER_PREFIX}.${DEMO_SERVER_DOMAIN}

# Configure the new machine
export DIGITALOCEAN_REGION=fra1
export DIGITALOCEAN_SIZE=s-2vcpu-4gb
export DIGITALOCEAN_MONITORING=true

# Define password for the load balancer dashboard
LB_DASHBOARD_BASIC_AUTH_USERNAME=admin
LB_DASHBOARD_BASIC_AUTH_PASSWORD=admin

# Create a new docker node at DO
docker-machine create \
  --driver digitalocean \
  --tls-san $DEMO_SERVER_HOSTNAME \
  $DEMO_SERVER_HOSTNAME
  
export swarm_master_ip=`docker-machine ip $DEMO_SERVER_HOSTNAME

# Make the new node a docker swarm master
docker-machine ssh $DEMO_SERVER_HOSTNAME docker swarm init --advertise-addr=${swarm_master_ip}

# Update DNS
./update_dns.sh $DEMO_SERVER_PREFIX $DEMO_SERVER_DOMAIN $swarm_master_ip $DIGITALOCEAN_ACCESS_TOKEN
./update_dns.sh \*.${DEMO_SERVER_PREFIX} $DEMO_SERVER_DOMAIN $swarm_master_ip $DIGITALOCEAN_ACCESS_TOKEN

# Setup local docker env to talk to new swarm master 
eval $(docker-machine env $DEMO_SERVER_HOSTNAME)


# Deploy load balancer
# export traefik_basic_auth_string=$(htpasswd -nb "$LB_DASHBOARD_BASIC_AUTH_USERNAME" "$LB_DASHBOARD_BASIC_AUTH_PASSWORD")
export traefik_basic_auth_string='admin:$apr1$7137RgbO$86qZPCbEt4BEYWXfDBDQK1'
docker stack deploy traefik --compose-file docker-compose.yml
```

Now, you should have the demo server running. 
You can visit the dashboard at `http://traefik.${DEMO_SERVER_HOSTNAME}`

Setup a sample app. by following `nginx/README.md`. Run the commands within that directory. 

Follow the instructions to setup a docker registry and a CI server
* `registry/README.md` 
* `jenkins/README.md`

Jenkins should now deploy your app to the demo server.

This demo is not ready for production. 
