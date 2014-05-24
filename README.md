docker-sybase
=============

### How to run Sybase ASE on Docker.  
This will allow you to create an Docker Image with Syabse on it (in this case Syabse ASE 15.7)
This demo uses Redhat but most of the steps are generic.

#### 1) Install docker

See : http://docs.docker.io/installation/rhel/

Enable epel (which is where docker.io comes from)
```sudo su -
wget http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
rpm -ivh epel-release-6-8.noarch.rpm
yum -y install docker-io
service docker start
docker version```

#### 2) From a previous installation of Sybase ASE
tar up the entire folder and call it : sybase_version_15_7.tgz

#### 3) Create a Sybase Image
# DOCKER-VERSION 0.3.4
```FROM    centos:6.4
RUN yum -y install libaio
RUN yum -y install gtk2.i686
ADD sybase_version_15_7.tgz /opt```


