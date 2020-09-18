# Overview
CI/CD setup.

## Host Setup

```bash
# Install docker and required tools.
apt-get update -y
apt-get install -y git docker docker-compose net-tools apache2-utils
```

```bash
# Change host ssh for easier gitea pass through.
sudo ufw allow 2288/tcp
sudo vim /etc/ssh/sshd_config
sudo systemctl restart ssh
```

```bash
# The git user will be used to handle the ssh connections and storage.
# Not using uid 1000 so needing some custom settings.
sudo addgroup --gid 1001 hosting
sudo adduser --ingroup hosting -u 1001 git
sudo adduser --system --ingroup hosting --ingroup docker git-docker
sudo mkdir -p /home/git/.ssh
sudo chown -R git:hosting /home/git/.ssh
```

```bash
su git
htpasswd -nb admin <pass>
git clone https://git.unexpectedeof.xyz/n0mn0m/miq.git traefik
mkdir ./traefik2/letsencrypt
mkdir ./traefik2/gitea
mkdir ./traefik2/postgres
mkdir ./traefik2/teamcity-postgres
mkdir -p ./traefik2/teamcity/data/lib/jdbc
mkdir -p ./traefik2/teamcity/data/config
mkdir -p ./traefik2/teamcity/logs
curl -L https://jdbc.postgresql.org/download/postgresql-42.2.16.jar -o postgresql-42.2.16.jar
mv postgresql-42.2.16.jar ./traefik2/teamcity/data/lib/jdbc
```

```bash
cd traefik
docker-compose up
```

```bash
vim giteassh.sh

#!/bin/sh
ssh -p 22 -o StrictHostKeyChecking=no git@127.0.0.1 "SSH_ORIGINAL_COMMAND=\"$SSH_ORIGINAL_COMMAND\" $0 $@"

chmod +x giteassh.sh

ssh-keygen -t rsa -b 4096 -C "Gitea Host Key"
ln -s ./gitea/git/.ssh/authorized_keys /home/git/.ssh/authorized_keys
echo "no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty $(cat /home/git/.ssh/id_rsa.pub)" >> ./gitea/git/.ssh/authorized_keys
```


```bash
docker-compose exec minio sh
wget https://dl.min.io/client/mc/release/linux-amd64/mc
chmod +x mc
./mc alias set minio http://localhost:9000 <access> <key>
./mc policy set download minio/static
wget https://github.com/cloudacademy/static-website-example/archive/master.zip
unzip master
./mc cp -r static-website-example-master/ minio/static
```

TeamCity Agent

```bash
sudo apt-get update
sudo apt-get install wget
sudo wget https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb
sudo dpkg -i packages-microsoft-prod.deb
sudo apt-get update; \
  sudo apt-get install -y apt-transport-https && \
  sudo apt-get update && \
  sudo apt-get install -y dotnet-sdk-3.1
sudo wget https://dl.min.io/client/mc/release/linux-amd64/mc
sudo chmod +x mc
sudo mv mc /usr/bin/
mc alias set minio http://192.168.1.51 BKIKJAA5BMMU2RHO6IBB V7f1CwQqAcwo80UEIJEjc5gVQUSSx5ohQ9GSrr12 --api S3v4
```

## References

- https://docs.traefik.io/user-guides/docker-compose/acme-dns/
- https://help.ui.com/hc/en-us/articles/217367937-EdgeRouter-Port-Forwarding
- https://community.ui.com/questions/ER-X-1-9-1-1-cant-forward-ports-Is-port-forwarding-broken-in-1-9/8514db59-0065-420c-a121-94ba456b56af#answer/254be129-5cbe-4fa3-9ded-90657f72bb79
- https://blog.thesparktree.com/traefik-advanced-config
- https://zah.me/blog/2020/06/14/build-a-pi-cluster-for-local-development-part-3/?ref=laravelnews
