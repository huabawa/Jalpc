
Before following the steps below, please go to EC2 by typing in EC2 in the AWS services search bar in your AWS console.

1. Select Launch Instance
2. Select the Free Tier Eligible Ubuntu Server 16.04 LTS (HVM), SSD Volume Type
3. Keep the Choose an Instance Type settings. Click Next.
4. Click next until you reach Step 6: Configure Security Group.
5. Click Add Rule. Choose Custom TCP. Enter 3000 in Port Range. Select Anywhere for Source.
6. Make sure your instance is running in Instances.
7. Open Terminal.
8. Select your instance in Instances. Click on Actions. Under Actions, click on Connect.
9. Go to your key pair directory in Terminal.
10. In Terminal, type and enter: chmod 400 keypair.pem
11. In Terminal, type and enter: ssh -i "keypair.pem" ubuntu@ec0-22-22-11-11.compute-1.amazonaws.com
12. When have successfully connected to the ubuntu instance, you will get something like this, ubuntu@ec2-00-000-00-000:

Type the following into Terminal, one line at a time.


sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
sudo apt-add-repository 'deb https://apt.dockerproject.org/repo ubuntu-xenial main'
sudo apt-get update

apt-cache policy docker-engine
sudo apt-get install -y docker-engine
sudo systemctl status docker

sudo groupadd docker
sudo usermod -aG docker $USER

git clone https://github.com/catarse/catarse.git

docker-compose -v
sudo curl -o /usr/local/bin/docker-compose -L "https://github.com/docker/compose/releases/download/1.11.2/docker-compose-$(uname -s)-$(uname -m)"


docker-compose run --rm web bash

rake db:create
rake db:migrate

bundle install
exit

sudo apt-get install -y nodejs
sudo apt install npm
sudo npm install -g bower
sudo apt-get install python-software-properties
curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
sudo apt-get install nodejs

bower init

bower install

docker-compose run --rm web rake

docker-compose run --rm web rake db:seed

docker-compose up -d

docker-compose logs

docker-compose ps

You should get the following if everything is running properly.

catarse_db_1       /docker-           Up                 0.0.0.0:5432->54 
                   entrypoint.sh                         32/tcp           
                   postgres                                               
catarse_jobs_1     bundle exec        Exit 137                            
                   sidekiq -c 1 -                                         
                   ...                                                    
catarse_redis_1    docker-            Up                 0.0.0.0:6379->63 
                   entrypoint.sh                         79/tcp           
                   redis ...                                              
catarse_test_1     bundle exec        Up                                  
                   sidekiq -c 1 -                                         
                   ...                                                    
catarse_web_1      rails server -b    Up                 0.0.0.0:3000->30 
                   0.0.0.0 -p ...                        00/tcp   

Type Public DNS (IPv4):3000 into your browser. Example: http://ec2-00-00-00-00.compute-1.amazonaws.com:3000/

Voila!! The Catarse website opens up.



