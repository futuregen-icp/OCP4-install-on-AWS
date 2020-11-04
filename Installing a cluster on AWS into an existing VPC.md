

# OCP4 Install on AWS 


## 환경 요구 사항
- region 확인
- 기존 subnet 확인 ID 및 CIDR
- route53에서 Public Zones 확보(base Domain)
- 기존 Public  Subnet에서 nat GW 설정 필수, private 인트넷 필수, 막약 인트넷 허용하지 않으면 proxy 설정하는 방법 있음
- machineNetwork에서 모든 subnet 인트넷 포함해야 함

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

##### 1, install-config.yaml 파일 생성
```
openshift-install create install-config --dir=/ocp/

? SSH Public Key /root/.ssh/id_rsa
? Platform aws
? AWS Access Key ID xxxxxxxxx
? AWS Secret Access Key xxxxxxxxxxxx
? Region ap-northeast-2 (Asia Pacific (Seoul))
? Base Domain icsfuturegen.de
? Cluster Name ocp4
? Pull Secret xxxxxxxxxxxxxxxxxxxxxxxx
```

##### 2, install-config.yaml 편집 - 현재 VPC 정보 맞게 세팅
```
apiVersion: v1
baseDomain: icsfuturegen.de
controlPlane:
  hyperthreading: Enabled
  name: master
  platform:
    aws:
      zones:
      - ap-northeast-2a
      - ap-northeast-2b
      rootVolume:
        iops: 4000
        size: 500
        type: io1
      type: m5.xlarge
  replicas: 3
compute:
- hyperthreading: Enabled
  name: worker
  platform:
    aws:
      rootVolume:
        iops: 2000
        size: 500
        type: io1
      type: c5.4xlarge
      zones:
      - ap-northeast-2a
      - ap-northeast-2b
  replicas: 3
metadata:
  name: qingsong
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 10.0.0.0/22
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  aws:
    region: ap-northeast-2
    subnets:
    - subnet-0f68f738415095fbf
    - subnet-03c340fb9bb72a460
    - subnet-0207afdbf5c333a47
    - subnet-013e4e06285455f6f
publish: External
pullSecret: '{"auths":{"cloud.openshift.com":{"auth":"b3BlbnNoaWZ0LXJlbGVhc2UtZGV2K29jbV9hY2Nlc3NfZjdhM2MzZWExNTlkNGJjNGE0MTk5YTc1MGEwZTIxNDg6OTcwT1c1Q0RHUUxWTTc3WVc4WFVGT0w4QzlFU0lYN0RYQUY3NzNLNDZNSE5QU0I3QkRSMFJJSlMzQ0ZHMThDUw==","email":"ebaysupport@futuregen.co.kr"},"quay.io":{"auth":"b3BlbnNoaWZ0LXJlbGVhc2UtZGV2K29jbV9hY2Nlc3NfZjdhM2MzZWExNTlkNGJjNGE0MTk5YTc1MGEwZTIxNDg6OTcwT1c1Q0RHUUxWTTc3WVc4WFVGT0w4QzlFU0lYN0RYQUY3NzNLNDZNSE5QU0I3QkRSMFJJSlMzQ0ZHMThDUw==","email":"ebaysupport@futuregen.co.kr"},"registry.connect.redhat.com":{"auth":"fHVoYy1wb29sLTM5Y2JjYzgzLTMyNGQtNDkzNy05NGM0LTYyYmZkMWE1ZDA5MzpleUpoYkdjaU9pSlNVelV4TWlKOS5leUp6ZFdJaU9pSXdOV1JtT0RreU5tTmlOR0UwWlRCallXSTJZemczTVdNek1EWTFaRFJrWVNKOS5vTkkxMkVfMkJ6dWJuYUhPQTJxMFoxX0xOdXZPbXIwdDh4OGFZbnExck1wWkFEeUZjRUVndndDWENuWWoxSi1lTzVPeDBNdU1xeXZ1TlRsQmxxSWpFWEk4MjZUeUFMU3hxWFFDUENyQjRuTm5SWkx5MlMtZjFyck9Wcm9MNmxYYWFfMkM3NVd6M0Voa2twVmh6NHhJVlJDZjNYaF9ndy1OdF9IWFVUSzBpUmJrbWg0MGR2T09GLU1vTEptdHhQdHk1V0sxNWlfekQwMi1vWTNnU09ZSVpWMDFJaDlNb2ZUcTBfYS10d2pMYXhrLUZac1A2MktsWDF1aHljOHA0MjBCNGNhQktvWDhYbXhzLW15S09pMHF5clhfQVdoOGpFeUZlRGRpLWxoVy12Y1VhOUVfS2JIelhlUUV0MWxwcWw4ME9CUDNOVmZBZ1dwWGhDUGhPeU1sVEkyUjBoT2w3aUJ2aWtIUWRUb2FSSWZkbGswSW9HLUJoTG93Nm5aZTM2YVNaZ2ZWOTViSWRUOHBqNzJvdlBhMmtuN3ljb0VFOUZhTFpLTHc5eDF3ck9oaFJNS1NFX1hVT09VVzBoSFNnc052TG0xS3NhS0xfbGFlbUxQajBvNlQzbG5uQko3TmVoRHphdEZ6a3pTTjludnVTa09md1ctU1ptLW1lZ1RhX2kxTzhyN2loVC1wUnN0eFJGSlhYYlhJa0tfRGtRZjNibW5zdnptRUxqV1liMnFBd2QwV2ZZVG1IU3haTzE2Wk9PbTVJeVVxbDRieVh6OVZtN1Jpb2ZKMllNRFFPV2NVZWpjeGFRTXpRZU1VVEtVc0dCYkdIMGZhcTFvanlBbDR2djlJeHBUTEVZYkhKRTAxYlBaX0hzT2N1ZVlIR2s1eG05cHM4S216RzBkaEpCbw==","email":"ebaysupport@futuregen.co.kr"},"registry.redhat.io":{"auth":"fHVoYy1wb29sLTM5Y2JjYzgzLTMyNGQtNDkzNy05NGM0LTYyYmZkMWE1ZDA5MzpleUpoYkdjaU9pSlNVelV4TWlKOS5leUp6ZFdJaU9pSXdOV1JtT0RreU5tTmlOR0UwWlRCallXSTJZemczTVdNek1EWTFaRFJrWVNKOS5vTkkxMkVfMkJ6dWJuYUhPQTJxMFoxX0xOdXZPbXIwdDh4OGFZbnExck1wWkFEeUZjRUVndndDWENuWWoxSi1lTzVPeDBNdU1xeXZ1TlRsQmxxSWpFWEk4MjZUeUFMU3hxWFFDUENyQjRuTm5SWkx5MlMtZjFyck9Wcm9MNmxYYWFfMkM3NVd6M0Voa2twVmh6NHhJVlJDZjNYaF9ndy1OdF9IWFVUSzBpUmJrbWg0MGR2T09GLU1vTEptdHhQdHk1V0sxNWlfekQwMi1vWTNnU09ZSVpWMDFJaDlNb2ZUcTBfYS10d2pMYXhrLUZac1A2MktsWDF1aHljOHA0MjBCNGNhQktvWDhYbXhzLW15S09pMHF5clhfQVdoOGpFeUZlRGRpLWxoVy12Y1VhOUVfS2JIelhlUUV0MWxwcWw4ME9CUDNOVmZBZ1dwWGhDUGhPeU1sVEkyUjBoT2w3aUJ2aWtIUWRUb2FSSWZkbGswSW9HLUJoTG93Nm5aZTM2YVNaZ2ZWOTViSWRUOHBqNzJvdlBhMmtuN3ljb0VFOUZhTFpLTHc5eDF3ck9oaFJNS1NFX1hVT09VVzBoSFNnc052TG0xS3NhS0xfbGFlbUxQajBvNlQzbG5uQko3TmVoRHphdEZ6a3pTTjludnVTa09md1ctU1ptLW1lZ1RhX2kxTzhyN2loVC1wUnN0eFJGSlhYYlhJa0tfRGtRZjNibW5zdnptRUxqV1liMnFBd2QwV2ZZVG1IU3haTzE2Wk9PbTVJeVVxbDRieVh6OVZtN1Jpb2ZKMllNRFFPV2NVZWpjeGFRTXpRZU1VVEtVc0dCYkdIMGZhcTFvanlBbDR2djlJeHBUTEVZYkhKRTAxYlBaX0hzT2N1ZVlIR2s1eG05cHM4S216RzBkaEpCbw==","email":"ebaysupport@futuregen.co.kr"}}}'
sshKey: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCZoNOWEyYlQEDfPaTh13zo5WrewVFxkj6Zwf2kXJHv50ezEn606hdnzD77iilvKWMedfXWsaUBMszQ5imL9v5/qIrPCMC2HqGzS8hxVSfkczAYb8kLDfrIM6/3VRDlyIi4wQPdind3vqK5oju7TNN+7+9ZstgrgtXx0UoaQyqMTsuYaCr8RM5Zwgis3LVkSXhn0mbE0zEWTKxEH4eqsWgL+gRu7vzizbIeBAUsLQtqTm6iNVu+ZggAkD0c0WfBMg3qzXbwG3ryhHxfknidJxCaPixQV2PayzlWCttGQMTm6bR6PZHO9nS8UazHJu0p0s9EQla8opV5O7F5QA38uttAo+w3vClDKqNVrnSxaTEvxjDpuMZ0qc+qT4TXC0KMlFfoB0OaqNRw4pS6JshHmJnjaIoRJb2Fr2w1I5CMKP9Oh3cXIZAMAebf89VWKzX7zjOd6GDuKXnWEkqbq9SoNOHISFYcVgcUMul/SMph5ot7LnsnhrPdNbsHhYCXgrN8+ZoA0pq4SFtqDpmXdyFJgChHpu2BtW+9HPjGcW/F6H0KOwmKlyz0jhgOOx2flduSfDQLkSyKNTZZ0oSclD88jR2CP10UCUQkjhYxKPecq9T1i+w2kOY8F54YNhC0HFFMg8M2trUORlsp9R3Rfg4CFlgykjEIey1vhAAxcnPbfQG85w== root@ip-10-0-0-187.ap-northeast-2.compute.internal'
```
##### 3, install-config.yaml 이용하여 클러스터 배포
```
openshift-install create cluster --dir=/ocp/ --log-level=debug
```
## Uninstalling a cluster on AWS
###### 설치 위치를 지정하세요!!
```
openshift-install destroy cluster --dir=/ocp --log-level=debug
```
