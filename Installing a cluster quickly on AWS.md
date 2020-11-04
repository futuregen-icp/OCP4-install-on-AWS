
# OCP4 Install on AWS

## bastion node 설정

##### 1, SSH private key 생성
```
ssh-keygen -t rsa -b 4096 -N ''
```  
##### 2, Add your SSH private key to the `ssh-agent`
```
eval "$(ssh-agent -s)"
ssh-add /root/.ssh/id_rsa
```  
##### 3, 설치 파일 준비
###### 1,OpenShift installer
```
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-install-linux.tar.gz
```
###### 2,Command line interface
```
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux.tar.gz
```
###### 3,Pull secret
###### Pull secret 파일 복사

###### 4, command 파일을 path 위치로 이동
```
mv kubectl /usr/bin/ && mv oc /usr/bin/ && mv openshift-install /usr/bin/
```


## Deploying the cluster

##### 1, Run the installation program
```
openshift-install create cluster --dir=/ocp/  --log-level=debug

? SSH Public Key /root/.ssh/id_rsa.pub
? Platform aws
? AWS Access Key ID xxxxxxxxx
? AWS Secret Access Key xxxxxxxxxxxx
? Region ap-northeast-2 (Asia Pacific (Seoul))
? Base Domain icsfuturegen.de
? Cluster Name ocp4
? Pull Secret xxxxxxxxxxxxxxxxxxxxxxxx
```

## Uninstalling a cluster on AWS
###### 설치 위치를 지정하세요!!
```
openshift-install destroy cluster --dir=/ocp --log-level=debug
```
