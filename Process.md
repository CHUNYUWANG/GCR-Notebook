## 1. Join GCR at http://gcr/

## 2. Reserve a GPU DEV machine at http://gcr-reservations/

## 3. Setup SSH Key for local windows computer at https://dev.azure.com/msresearch/GCR/_wiki/wikis/GCR.wiki/531/Linux-Sandbox-Getting-Started?anchor=generate-ssh-key-and-upload-to-gcr

## 4. Mount directory at local linux workstation to share folder (performed on GCR DEV machine)
```
sudo mount.cifs //gcrnfsw2-msraim/msraimscratch/$(whoami|cut -d. -f2) /msraimscratch/$(whoami|cut -d. -f2) --verbose -o cruid=$(id -u),uid=$(id -u),gid=$(id -g),nounix,serverino,mapposix,file_mode=0777,dir_mode=0777,noforceuid,noforcegid,vers=2.1,domain=FAREAST,username=chnuwa
```

## 5. Copy data to the share folder (performed on GCR DEV machine) (mind the "*" and "/" at the end )
```
rysnc -avz --progress chnuwa@10.150.146.3:/home/jrmei/GCR/* /msraimscratch/chnuwa/GCR/
```

## 6. Build a docker (performed on GCR DEV machine)
```
uid=$(id -u)
did=$(id -g)
alias=$(whoami|awk -F. '{ print $2 }' )
domain=$(whoami|awk -F. '{ print $1 }')


sudo docker build -t gcr-repos.redmond.corp.microsoft.com:5000/chnuwa/tensorflow:1.7-cu90-cudnn70-py27 --build-arg uid=$uid --build-arg did=$did --build-arg alias=$alias --build-arg domain=$domain -f myconfig .

docker push gcr-repos.redmond.corp.microsoft.com:5000/chnuwa/tensorflow:1.7-cu90-cudnn70-py27
```

Sample myconfig file
```
# cp config_tensorflow_1.10.1_py35 myconfig
FROM nvidia/cuda:9.0-devel-ubuntu16.04
#
ARG uid
ARG did
ARG domain
ARG alias
# Custom package installs

#*************************************
RUN apt-get update \
&& apt-get install -y --no-install-recommends \
            libcudnn7=7.0.5.15-1+cuda9.0 \
            libcudnn7-dev=7.0.5.15-1+cuda9.0 \
            sudo python3-dev python3-pip libgtk2.0-dev \
&& cd /usr/bin \
&& rm python \
&& ln -s python3 python \
&& ln -s pip3 pip \
&& pip --no-cache-dir install --upgrade pip==9.0.3 \
&& pip --no-cache-dir install --upgrade setuptools \
&& pip --no-cache-dir install tensorflow-gpu==1.10.1 scipy Pillow sacred pymongo nvidia-ml-py3 opencv-python \
&& apt-get clean \
&& rm -rf /var/lib/apt/lists/*
#*************************************
# The below will create your user in your container which you can use to run jobs
RUN echo "${domain}.domain users:x:${did}:${domain}.${alias}" >> /etc/group \
&& echo "${domain}.${alias}:x:${uid}:${did}:${alias},,,:/home/${alias}:/bin/bash" >> /etc/passwd \
&& echo "${domain}.${alias}:*:17575:0:99999:7:::" >> /etc/shadow \
&& echo "${domain}.${alias} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/${alias} \
&& echo "  " >> /etc/sudoers.d/${alias}
# Copy "myconfig" file into my container on build
COPY myconfig /root/myconfig

```

## Run demo code on GPU DEV machine first
```
docker run --rm --ipc=host --volume-driver=nfs -e CUDA_VISIBLE_DEVICES=0 -v gcrnfsw2-msraim.redmond.corp.microsoft.com/msraimscratch:/msraimscratch --user FAREAST.chnuwa gcr-repos.redmond.corp.microsoft.com:5000/chnuwa/tensorflow:1.7-cu90-cudnn70-py27 sh -c "cd /msraimscratch/wenxie/GCR && python -u cnn_mnist.py 1> out.txt 2>err.txt"
```

## Run code on HPC
```
Install the client at \\gcrgpu03\REMINST

```

