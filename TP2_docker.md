# Part I : Programmatic approach

🌞 **Créez une VM depuis le Azure CLI**


```
az vm create -g Efrei -n machine3 --image Ubuntu2204 --admin-username julien --ssh-key-values "C:\Users\menan\.ssh\id_rsa.pub" --size Standard_B1s --location westeurope
```

🌞 **Assurez-vous que vous pouvez vous connecter à la VM en SSH sur son IP publique.**
  - **la présence du service `walinuxagent`**
```
julien@machine3:~$ systemctl status walinuxagent
● walinuxagent.service - Azure Linux Agent
     Loaded: loaded (/lib/systemd/system/walinuxagent.service; enabled; ven>
     Active: active (running) since Tue 2025-03-18 09:13:53 UTC; 5min ago
   Main PID: 767 (python3)
      Tasks: 6 (limit: 1004)
     Memory: 40.2M
        CPU: 2.354s
     CGroup: /system.slice/walinuxagent.service
             ├─ 767 /usr/bin/python3 -u /usr/sbin/waagent -daemon
             └─1093 python3 -u bin/WALinuxAgent-2.12.0.2-py3.9.egg -run-ext
```
  - **la présence du service `cloud-init`**
```
julien@machine3:~$ systemctl status cloud-init
● cloud-init.service - Cloud-init: Network Stage
     Loaded: loaded (/lib/systemd/system/cloud-init.service; enabled; vendor preset: enabled)
     Active: active (exited) since Tue 2025-03-18 09:13:52 UTC; 20min ago
   Main PID: 515 (code=exited, status=0/SUCCESS)
        CPU: 2.689s
```
```
julien@machine3:~$ cloud-init status
status: done
```
# II. Un ptit LAN

🌞 **Créez deux VMs depuis le Azure CLI**
```
az network vnet create -g Efrei -n MyVNet --subnet-name MySubnet --address-prefix 10.0.0.0/16 --subnet-prefix 10.0.0.0/24 --location westeurope
```
```
az vm create -g Efrei -n machine4 --image Ubuntu2204 --size Standard_B1s --admin-username julien --ssh-key-values "C:\Users\menan\.ssh\id_rsa.pub" --vnet-name MyVNet --subnet MySubnet --location westeurope
```
```
julien@machine4:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 7c:ed:8d:14:b1:00 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.4/24 metric 100 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::7eed:8dff:fe14:b100/64 scope link
       valid_lft forever preferred_lft forever
```
```
az vm create -g Efrei -n machine5 --image Ubuntu2204 --size Standard_B1s --admin-username julien --ssh-key-values "C:\Users\menan\.ssh\id_rsa.pub" --vnet-name MyVNet --subnet MySubnet --location westeurope
```
```
julien@machine5:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 7c:1e:52:75:8a:a5 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.5/24 metric 100 brd 10.0.0.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::7e1e:52ff:fe75:8aa5/64 scope link
       valid_lft forever preferred_lft forever
```
```
julien@machine4:~$ ping 10.0.0.5
PING 10.0.0.5 (10.0.0.5) 56(84) bytes of data.
64 bytes from 10.0.0.5: icmp_seq=1 ttl=64 time=2.27 ms
64 bytes from 10.0.0.5: icmp_seq=2 ttl=64 time=2.69 ms
64 bytes from 10.0.0.5: icmp_seq=3 ttl=64 time=1.92 ms
```

# Part II : cloud-init
🌞 **Tester `cloud-init`**
```
az vm create -g Efrei -n VMFINAL2 --image Ubuntu2204 --size Standard_B1s --admin-username julien --ssh-key-values "C:\Users\menan\.ssh\id_rsa.pub" --custom-data "C:\Users\menan\Documents\CoursEfrei\Cloud\cloud-init.txt" --location westeurope
```
## 3. Write your own
🌞 **Utilisez `cloud-init` pour préconfigurer la VM :**
```
#cloud-config
users:
  - default
  - name: user
    asswd: $6$je4cCvxfL/5PgBbB$xFl/qQDh6b1fSs7rvxlnkRw5e9ujt08IPXh.Aq96XPJi8x.7IcPPufCEFkCFUZg4mSUVS3y.hbxJGV19ZRzMq1
    sudo: sudo
    groups: sudo, docker
    shell: /bin/bash
    ssh_authorized_keys:
      - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQD7P4dGjPS8cfuZcz+3DAYAYvbTq8qT8/+WwMG085/gDsgMsDb50KsDNwkegjPpnbj59eRYLWeZHNThiSiKyWx51R4sRQBJARguTf1+mB+MUgdJ4VkQPY1Iji2AD09XyxSgtqHenROT2YFkEpWylNyoLRE3gfS3f98UeNIAy6gq2h+G/gyW5SJsciEzrhAh85UwR7MPDdEWU2MHb3HGceOfTZSrQvcMesIm6WGVv1jpyst53gj8ZEI50J0u3Ky6gdkzh5m2GwmZoCStfY2b1h/RJl++3QGoccIcYa/gTTOmtGBGv7pBeZYEROu3jyK1QppiAfothmRSWaoOY0GBPU9PxpLUk3N+Wrgx3wMX/ZinBoGsbv9XBF6DkIKfW7URx0leRD5weQNiasH2b7JqBr92HC6MXmN1wQs4B5VPSI2AXK0XA7gaPCitGaixZ7YqBzWrMOHVbQtn/dLUKaxp3egF0PQQWla/cfAWuxY3WNrgAiLnixNCnOQk8/KTeY67GXmefjyEiseKJVj5LvVm2VY/rXkpe6QmpVYfMerEY7/z4zrLJO71EQLOPXmdgKGdXNzzER+/t5/iGTJsWYGkq3PPyy8luX0ybCgcYL71nwp0sbBzA37sBUchRev+bau7pAaeKISr0lJ+QADGqToZyN1dx2lVEUBsv6CfEKVabRRFJw== 

package_update: true
package_upgrade: true

groups:
  - docker

write_files:
  - path: /usr/share/keyrings/docker.asc
    owner: root:root
    permissions: '0644'
    content: |
      -----BEGIN PGP PUBLIC KEY BLOCK-----

      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
      YT90qFF93M3v01BbxP+EIY2/9tiIPbrd
      =0YYh
      -----END PGP PUBLIC KEY BLOCK-----

apt:
  sources:
    docker.list:
      source: deb [arch=amd64 signed-by=/usr/share/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $RELEASE stable

packages:
  - docker-ce
  - docker-ce-cli
  - containerd.io
  - docker-compose-plugin

runcmd:
  - docker pull alpine:latest
```
```
user@VMFINAL2:~$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
alpine       latest    aded1e1a5b37   4 weeks ago   7.83MB
user@VMFINAL2:~$
```