docker-sybase
=============

### How to run Sybase ASE on Docker.  
This will allow you to create a Docker Image with Sybase on it (in this case Syabse ASE 15.7)
This demo uses Redhat but most of the steps are generic.

#### 1) Install docker

See : http://docs.docker.io/installation/rhel/

Enable epel (which is where docker.io comes from)
```
sudo su -
wget http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
rpm -ivh epel-release-6-8.noarch.rpm
yum -y install docker-io
service docker start
docker version
```

#### 2) From a previous installation of Sybase ASE
tar up the entire folder and call it : sybase_version_15_7.tgz.  For these instructions the server is called AWS. Note as you are going to need to change the /opt/sybase/interfaces file you will need to using a PLACE_HOLDER in that file where the ipaddress would normally go.  It will be swapped out in step 7 below.

#### 3) Create a Sybase Image 
vi Dockerfile the below
```
# DOCKER-VERSION 0.3.4
FROM    centos:6.4
RUN yum -y install libaio
RUN yum -y install gtk2.i686
ADD sybase_version_15_7.tgz /opt
```

#### 4) Build the Image as sybase_15_7
```
docker build -t sybase_15_7 .
```

#### 5) Run the sybase_15_7 image
```
docker run -i -t -p 5000:5000 -v sybase_15_7 bash

```
#### 6) Keep the sybase_15_7 image running
in the bash prompt type to exit and keep the container running
```
control-p then control-q
```

#### 7) Apply a dump file to the database via sh script
copy the dumpfile to /tmp on the host. Also copy there the script listed in 7.1 calledrunSybase_15_7_and_install_dump.sh
```
docker run -i -t -p 5000:5000 -v /tmp/:/tmpfromhost sybase_15_7 bash -c 'cd tmpfromhost; chmod 777 *.*; ./runSybase_15_7_and_install_dump.sh;'

```

##### 7.1) Apply a dump file to the database
```
# 0.1) Install dos2unix
yum -y install dos2unix

# 1) Get the IPAddress of the container and use it to configure this instance of Sybase
IPADDRESS="$(ifconfig eth0 | sed -n '/inet /{s/.*addr://;s/ .*//;p}')"
sed -i "s/PLACE_HOLDER/$IPADDRESS/g" /opt/sybase/interfaces

# 2) Set memory for sybase and start it up
rm -f /opt/sybase/ASE-15_0/install/AWS.log
touch /opt/sybase/ASE-15_0/install/AWS.log
sysctl -w kernel.shmmax=1067108864
sysctl -w kernel.shmall=30000000000
startserver -f /opt/sybase/ASE-15_0/install/RUN_AWS

# 4) Loop until Sybase is started up
LINES=$(grep 'Console logging is disabled. This is controlled via the ' /opt/sybase/ASE-15_0/install/AWS.log | wc -l);
while [ $LINES == 0 ]
    do
    sleep 5
    LINES=$(grep 'Console logging is disabled. This is controlled via the ' /opt/sybase/ASE-15_0/install/AWS.log | wc -l);
done

echo =================Sybase is Started============================

# 5) run the db scripts to install the dump. Note this will be whatever you need to do to install a dump, create users etc.
dos2unix /tmpfromhost/load_database.sql
echo connecting to isql using $IPADDRESS
${SYBASE}/${SYBASE_OCS}/bin/isql -S$IPADDRESS:5000 -Usa -Pxxxx -i /tmpfromhost/load_database.sql

# 6) Note this is here to keep the docker container running.
while ( true )
    do
    echo "Detach with Ctrl-p Ctrl-q. Dropping to shell"
    sleep 1
    /bin/bash
done


```


#### 8) Build the database Image after the above finishes
docker commit containerId my_database_running_on_sybase

#### 9) Start the new sybase image with running the database 
docker run -i -t -p 5000:5000 -v /tmp:/tmpfromhost my_database_running_on_sybase bash -c 'cd tmpfromhost; chmod 777 *.*; ./startSybase.sh;'

##### startSybase.sh looks like this
```
#startSybase.sh
sysctl -w kernel.shmmax=1067108864
sysctl -w kernel.shmall=30000000000
startserver -f /opt/sybase/ASE-15_0/install/RUN_AWS

while ( true )
    do
    echo "Detach with Ctrl-p Ctrl-q. Dropping to shell"
    sleep 1
    /bin/bash
done

```
#### 10) Test the new container by logging onto sybase

```
IPADDRESS="$(ifconfig eth0 | sed -n '/inet /{s/.*addr://;s/ .*//;p}')"
${SYBASE}/${SYBASE_OCS}/bin/isql -S$IPADDRESS:5000 -Usa -Pxxxx
online database MY_DATABASE
go
use MY_DATABASE
go
select * from people
go
```

    
