## 1. Join GCR at http://gcr/

## 2. Reserve a GPU DEV machine at http://gcr-reservations/

## 3. Setup SSH Key for local windows computer at https://dev.azure.com/msresearch/GCR/_wiki/wikis/GCR.wiki/531/Linux-Sandbox-Getting-Started?anchor=generate-ssh-key-and-upload-to-gcr

## 4. Mount directory at local workstation to share folder
```
sudo mount.cifs //gcrnfsw2-msraim/msraimscratch/$(whoami|cut -d. -f2) /msraimscratch/$(whoami|cut -d. -f2) --verbose -o cruid=$(id -u),uid=$(id -u),gid=$(id -g),nounix,serverino,mapposix,file_mode=0777,dir_mode=0777,noforceuid,noforcegid,vers=2.1,domain=FAREAST,username=chnuwa
```
## 5. Copy data to the share folder (mind the "*" and "/" at the end )
```
rysnc -avz --progress chnuwa@10.150.146.3:/home/jrmei/GCR/* /msraimscratch/chnuwa/GCR/
```
