# my-jenkins

Installed docker and docker-compose on the server

For docker

sudo apt update

sudo apt install apt-transport-https ca-certificates curl software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"

sudo apt update

apt-cache policy docker-ce

sudo apt install docker-ce

For docker-compose

sudo curl -L "https://github.com/docker/compose/releases/download/1.24.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

docker-compose --version

docker-compose file

Permission for the mounted volumes


sudo chmod -R +777 /data


version: '2'
services:
  jenkins:
    image: 'jenkinsci/blueocean:latest'
    ports:
      - '8080:8080'
    volumes:
      - /data/jenkins:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
    privileged: true
    user: root
    restart: always
  nexus:
    image: sonatype/nexus3:latest
    ports:
      - "8081:8081"
      - "8123:8123"
    volumes:
      - /data/nexus-volume:/nexus-data
    restart: always
sample for jenkins reverse proxy
server {
    listen 80;
    server_name build-jenkins.evolvemtgs.com;

    location / {
      proxy_set_header        Host $host:$server_port;
      proxy_set_header        X-Real-IP $remote_addr;
      proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header        X-Forwarded-Proto $scheme;

      # Fix the "It appears that your reverse proxy set up is broken" error.
      proxy_pass          http://127.0.0.1:8080;
      proxy_read_timeout  90;
    }
  }
Nexus
server {
    listen 80;
    server_name nexus.evolvemtgs.com;

    location / {
      proxy_set_header        Host $host:$server_port;
      proxy_set_header        X-Real-IP $remote_addr;
      proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header        X-Forwarded-Proto $scheme;

      # Fix the "It appears that your reverse proxy set up is broken" error.
      proxy_pass          http://127.0.0.1:8081;
      proxy_read_timeout  90;
      client_max_body_size 10M;
    }
  }
Docker Registry
server {
    listen 80;
    server_name docker-registry.evolvemtgs.com;

    location / {
      proxy_set_header        Host $host:$server_port;
      proxy_set_header        X-Real-IP $remote_addr;
      proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header        X-Forwarded-Proto $scheme;

      # Fix the "It appears that your reverse proxy set up is broken" error.
      proxy_pass          http://127.0.0.1:8123;
      proxy_read_timeout  90;
      client_max_body_size 10G;
    }
  }
Also add domain mapping in /etc/hosts

10.19.187.208 build-jenkins.evolve.com
10.19.187.208 docker-registry.evolvemtgs.com
10.19.187.208 nexus.evolvemtgs.com
Restart Nginx Server
sudo /etc/init.d/nginx start
sudo /etc/init.d/nginx status
Add your user to docker
sudo usermod -aG docker $USER
Environment Variable
NUGET: NuGet URL
NUGET_KEY: NuGet Key (Admin Icon-> NuGet API Key )
DOCKER_USER: username
DOCKER_PASS: password
DOCKER_REPO: docker repository (domain)
NPM_REGISTRY_URL: https://www.npmjs.com
NPM_TOKEN: 987ab1f7-1858-4783-8c65-ef413b2e47cd

NUGET_PASS: NuGet Password
NUGET_USER: NuGet User
RESTORE: "dotnet restore --ignore-failed-sources --no-cache -s https://api.nuget.org/v3/index.json -s http://nexus.evolvemtgs.com/repository/nuget-hosted/ "
Configure the pipeline
Global Pipeline Libraries

Load implicitly: true
Allow default version to be overridden: true
Include @Library changes in job recent changes: true
