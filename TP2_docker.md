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
az vm create -g Efrei -n VMFINAL --image Ubuntu2204 --size Standard_B1s --custom-data "C:\Users\menan\Documents\CoursEfrei\Cloud\cloud-init.txt" --location westeurope
```
## 3. Write your own
🌞 **Utilisez `cloud-init` pour préconfigurer la VM :**
```
az vm create -g Efrei -n VMFINAL2 --image Ubuntu2204 --size Standard_B1s --custom-data "C:\Users\menan\Documents\CoursEfrei\Cloud\cloud-init.txt" --location westeurope
```
```yml
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

# Part III : Terraform
## 2. Copy paste

## main.tf
```
provider "azurerm" {
  features {}
  subscription_id = "nop_nop"
}
resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}
resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}
resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.2.0/24"]
}
resource "azurerm_public_ip" "pip" {
  name                = "${var.prefix}-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}
resource "azurerm_network_interface" "main" {
  name                = "${var.prefix}-nic1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip.id
  }
}
resource "azurerm_network_interface" "internal" {
  name                = "${var.prefix}-nic2"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}
resource "azurerm_network_security_group" "ssh" {
  name                = "ssh"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  security_rule {
    access                     = "Allow"
    direction                  = "Inbound"
    name                       = "ssh"
    priority                   = 100
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = azurerm_network_interface.main.private_ip_address
  }
}
resource "azurerm_network_interface_security_group_association" "main" {
  network_interface_id      = azurerm_network_interface.main.id
  network_security_group_id = azurerm_network_security_group.ssh.id
}
resource "azurerm_linux_virtual_machine" "main" {
  name                            = "${var.prefix}-vm"
  resource_group_name             = azurerm_resource_group.main.name
  location                        = azurerm_resource_group.main.location
  size                            = "Standard_B1s"
  admin_username                  = "Me"
  network_interface_ids = [
    azurerm_network_interface.main.id,
    azurerm_network_interface.internal.id,
  ]
  admin_ssh_key {
    username   = "Me"
    public_key = file("C:\\Users\\menan\\.ssh\\id_rsa.pub")
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}
```
## variables.tf
```
variable "prefix" {
  description = "da prefix"
  default = "tp2magueule"
}
variable "location" {
  description = "da location"
  default = "West Europe"
}
```

🌞 **Constater le déploiement**
`
  ```
  az>> az vm list -o table
  Name            ResourceGroup          Location    Zones
  --------------  ---------------------  ----------  -------
  tp2magueule-vm  TP2MAGUEULE-RESOURCES  westeurope
  ```
## 3. Do it yourself

🌞 **Créer un *plan Terraform* avec les contraintes suivantes**

- `node1`
  - Ubuntu 22.04
  - 1 IP Publique
  - 1 IP Privée
- `node2`
  - Ubuntu 22.04
  - 1 IP Privée
- les IPs privées doivent permettre aux deux machines de se `ping`

main.tf
```
provider "azurerm" {
  features {}
  subscription_id = "nop"
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_public_ip" "node1_pip" {
  name                = "${var.prefix}-node1-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}

resource "azurerm_network_interface" "node1" {
  name                = "${var.prefix}-node1-nic"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location

  ip_configuration {
    name                          = "public"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.node1_pip.id
  }
}

resource "azurerm_network_interface" "node2" {
  name                = "${var.prefix}-node2-nic"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location

  ip_configuration {
    name                          = "private"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
  }
}

resource "azurerm_network_security_group" "internal" {
  name                = "${var.prefix}-internal-nsg"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    name                       = "AllowPing"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Icmp"
    source_port_range          = "*"
    source_address_prefix      = "10.0.1.0/24"
    destination_port_range     = "*"
    destination_address_prefix = "10.0.1.0/24"
  }
  
  security_rule {
    name                       = "AllowSSH"
    priority                   = 110
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = "*"
  }
}

resource "azurerm_network_interface_security_group_association" "node1" {
  network_interface_id      = azurerm_network_interface.node1.id
  network_security_group_id = azurerm_network_security_group.internal.id
}

resource "azurerm_network_interface_security_group_association" "node2" {
  network_interface_id      = azurerm_network_interface.node2.id
  network_security_group_id = azurerm_network_security_group.internal.id
}

resource "azurerm_linux_virtual_machine" "node1" {
  name                = "${var.prefix}-node1"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_B1s"
  admin_username      = "julien"
  network_interface_ids = [
    azurerm_network_interface.node1.id
  ]
  admin_ssh_key {
    username   = "julien"
    public_key = file("C:\\Users\\menan\\.ssh\\id_rsa.pub")
  }
  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}

resource "azurerm_linux_virtual_machine" "node2" {
  name                = "${var.prefix}-node2"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_B1s"
  admin_username      = "julien"
  network_interface_ids = [
    azurerm_network_interface.node2.id
  ]

  disable_password_authentication = false
  admin_password = "Node22@"

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }
  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}
```

variables.tf
```
variable "prefix" {
  description = "Project prefix"
  default     = "partie3B"
}

variable "location" {
  description = "Azure region"
  default     = "West Europe"
}
```
Preuve
```
PS C:\Users\menan> ssh -J julien@104.45.41.151 julien@10.0.1.4
The authenticity of host '10.0.1.4 (<no hostip for proxy command>)' can't be established.
ED25519 key fingerprint is SHA256:SXLs/oBCBVVWznRc92JwaU195QWD32TBFWavxBSE9Ys.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.0.1.4' (ED25519) to the list of known hosts.
julien@10.0.1.4's password:
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1021-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sun Mar 23 10:28:20 UTC 2025

  System load:  0.04              Processes:             114
  Usage of /:   5.4% of 28.89GB   Users logged in:       0
  Memory usage: 30%               IPv4 address for eth0: 10.0.1.4
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
New release '24.04.2 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Sun Mar 23 10:23:54 2025 from 10.0.1.5
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

julien@partie3B-node2:~$
```
## 4. cloud-iniiiiiiiiiiiiit

### A. Un premier tf + cloud-init

🌞 **Intégrer la gestion de `cloud-init`**

`variables.tf`
```
variable "prefix" {
  description = "Project prefix"
  default     = "partie3C"
}

variable "location" {
  description = "Azure region"
  default     = "West Europe"
}
```
`main.tf`
```
provider "azurerm" {
  features {}
  subscription_id = "074aa5b4-e4ab-42f9-807d-4ee508b26e98"
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_public_ip" "pip" {
  name                = "${var.prefix}-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}

resource "azurerm_network_interface" "vm_nic" {
  name                = "${var.prefix}-vm-nic"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location

  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip.id
  }
}

resource "azurerm_network_security_group" "ssh" {
  name                = "ssh"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    access                     = "Allow"
    direction                  = "Inbound"
    name                       = "ssh"
    priority                   = 100
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = "*"
  }
}

resource "azurerm_network_interface_security_group_association" "vm" {
  network_interface_id      = azurerm_network_interface.vm_nic.id
  network_security_group_id = azurerm_network_security_group.ssh.id

  depends_on = [azurerm_linux_virtual_machine.vm]
}

resource "azurerm_linux_virtual_machine" "vm" {
  name                = "${var.prefix}-vm"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_B1s"
  admin_username      = "julien"

  network_interface_ids = [
    azurerm_network_interface.vm_nic.id
  ]

  admin_ssh_key {
    username   = "julien"
    public_key = file("C:\\Users\\menan\\.ssh\\id_rsa.pub")
  }

  custom_data = base64encode(file("C:\\Users\\menan\\Documents\\CoursEfrei\\Cloud\\terraform3\\cloud-init.txt"))

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}

```
`cloud-init.txt`
```
#cloud-config
users:
  - default
  - name: user
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
  - sudo docker pull alpine:latest
```

🌞 **Proof !**
```
PS C:\Users\menan\Documents\CoursEfrei\Cloud\terraform3> ssh user@52.148.202.135
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.8.0-1021-azure x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Sun Mar 23 11:05:45 UTC 2025

  System load:  0.71              Processes:             120
  Usage of /:   8.1% of 28.89GB   Users logged in:       0
  Memory usage: 36%               IPv4 address for eth0: 10.0.1.4
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

New release '24.04.2 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Sun Mar 23 11:03:59 2025 from 91.169.101.209
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

user@partie3C-vm:~$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
alpine       latest    aded1e1a5b37   5 weeks ago   7.83MB
user@partie3C-vm:~$
```

### B. Go further

`main.tf`
```
provider "azurerm" {
  features {}
  subscription_id = "074aa5b4-e4ab-42f9-807d-4ee508b26e98"
}

resource "azurerm_resource_group" "main" {
  name     = "${var.prefix}-resources"
  location = var.location
}

resource "azurerm_virtual_network" "main" {
  name                = "${var.prefix}-network"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "internal" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
}

resource "azurerm_public_ip" "pip" {
  name                = "${var.prefix}-pip"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  allocation_method   = "Static"
}

resource "azurerm_network_interface" "vm_nic" {
  name                = "${var.prefix}-vm-nic"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location

  ip_configuration {
    name                          = "primary"
    subnet_id                     = azurerm_subnet.internal.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.pip.id
  }
}

resource "azurerm_network_security_group" "ssh" {
  name                = "ssh"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  security_rule {
    access                     = "Allow"
    direction                  = "Inbound"
    name                       = "ssh"
    priority                   = 100
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = "*"
  }

  security_rule {
    access                     = "Allow"
    direction                  = "Inbound"
    name                       = "AllowWikiJS"
    priority                   = 110
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "10101"
    destination_address_prefix = "*"
  }
}

resource "azurerm_network_interface_security_group_association" "pow" {
  network_interface_id      = azurerm_network_interface.vm_nic.id
  network_security_group_id = azurerm_network_security_group.ssh.id
}

resource "azurerm_linux_virtual_machine" "wikijs" {
  name                = "${var.prefix}-vm"
  resource_group_name = azurerm_resource_group.main.name
  location            = azurerm_resource_group.main.location
  size                = "Standard_B1s"
  admin_username      = "julien"

  network_interface_ids = [
    azurerm_network_interface.vm_nic.id
  ]

  admin_ssh_key {
    username   = "julien"
    public_key = file("C:\\Users\\menan\\.ssh\\id_rsa.pub")
  }

  custom_data = base64encode(file("C:\\Users\\menan\\Documents\\CoursEfrei\\Cloud\\terraform4\\cloud-init.txt"))

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts"
    version   = "latest"
  }

  os_disk {
    storage_account_type = "Standard_LRS"
    caching              = "ReadWrite"
  }
}

```
`cloud-init.txt`
```
#cloud-config
users:
  - default
  - name: user
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
  - path: /opt/wikijs/docker-compose.yml
    owner: root:root
    permissions: '0644'
    content: |
      version: "3.8"

      services:
        db:
          image: mysql:5.7
          restart: always
          ports:
            - '3306:3306'
          volumes:
            - "./db/mysql_files:/var/lib/mysql"
          environment:
            MYSQL_ROOT_PASSWORD: mysql
            MYSQL_DATABASE: wiki
            MYSQL_USER: wiki
            MYSQL_PASSWORD: mysql

        wiki:
          image: requarks/wiki
          restart: always
          depends_on:
            - db
          ports:
            - "10101:3000"
          environment:
            DB_TYPE: mysql
            DB_HOST: db
            DB_PORT: 3306
            DB_USER: wiki
            DB_PASS: mysql
            DB_NAME: wiki

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
  - docker compose -f /opt/wikijs/docker-compose.yml up -d
```
variable.tf
```
variable "prefix" {
  description = "Project prefix"
  default     = "partie3D"
}

variable "location" {
  description = "Azure region"
  default     = "West Europe"
}
```  
🌞 **Proof !**
```
menan@Menmen MINGW64 ~
$ curl http://52.232.41.218:10101 | tail -n2
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  3017  100  3017    0     0  20774      0 --:--:-- --:--:-- --:--:-- 20806
</script><link type="text/css" rel="stylesheet" href="/_assets/css/app.0e32c5b5f7b7df293dc5.css"><script type="text/javascript" src="/_assets/js/runtime.js?1738531300"></script><script type="text/javascript" src="/_assets/js/app.js?1738531300"></script></head><body><div id="root"><page locale="en" path="home" title="j'ai fini" description="" :tags="[]" created-at="2025-03-23T11:51:52.054Z" updated-at="2025-03-23T11:51:52.054Z" author-name="Administrator" :author-id="1" editor="ckeditor" :is-published="true" toc="W10=" :page-id="1" sidebar="W3siaSI6InNkaS0xIiwiayI6ImxpbmsiLCJsIjoiSG9tZSIsImMiOiJtZGktaG9tZSIsInkiOiJob21lIiwidCI6Ii8ifV0=" nav-mode="MIXED" comments-enabled effective-permissions="eyJjb21tZW50cyI6eyJyZWFkIjp0cnVlLCJ3cml0ZSI6ZmFsc2UsIm1hbmFnZSI6ZmFsc2V9LCJoaXN0b3J5Ijp7InJlYWQiOmZhbHNlfSwic291cmNlIjp7InJlYWQiOmZhbHNlfSwicGFnZXMiOnsicmVhZCI6dHJ1ZSwid3JpdGUiOmZhbHNlLCJtYW5hZ2UiOmZhbHNlLCJkZWxldGUiOmZhbHNlLCJzY3JpcHQiOmZhbHNlLCJzdHlsZSI6ZmFsc2V9LCJzeXN0ZW0iOnsibWFuYWdlIjpmYWxzZX19" edit-shortcuts="eyJlZGl0RmFiIjp0cnVlLCJlZGl0TWVudUJhciI6ZmFsc2UsImVkaXRNZW51QnRuIjp0cnVlLCJlZGl0TWVudUV4dGVybmFsQnRuIjp0cnVlLCJlZGl0TWVudUV4dGVybmFsTmFtZSI6IkdpdEh1YiIsImVkaXRNZW51RXh0ZXJuYWxJY29uIjoibWRpLWdpdGh1YiIsImVkaXRNZW51RXh0ZXJuYWxVcmwiOiJodHRwczovL2dpdGh1Yi5jb20vb3JnL3JlcG8vYmxvYi9tYWluL3tmaWxlbmFtZX0ifQ==" filename="home.html"><template slot="contents"><div><p>tp fini&nbsp;</p>
</div></template><template slot="comments"><div><comments></comments></div></template></page></div></body></html>

```

