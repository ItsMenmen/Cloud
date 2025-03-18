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

- assurez-vous qu'elles ont une IP privée (avec `ip a`)
- elles peuvent se `ping` en utilisant cette IP privée
- deux VMs dans un LAN quoi !

> *N'hésitez pas à vous rendre sur la WebUI de Azure pour voir vos VMs créées.*

---

c