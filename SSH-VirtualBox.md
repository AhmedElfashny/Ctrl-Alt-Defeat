# SSH to Virtual Machine From Your Host
## Description
After setting up the VirtualBox, I encountered difficulty SSH-ing into it. The network configuration was initially set to NAT instead of Bridge, resulting in the virtual machine being placed in a separate network. To establish connectivity, I had to reconfigure VirtualBox to be on the same network as my host machine. Note that all the command are being used for Debian Distro

### Actions
Set the VirtualBox Network to use Bridge [ Which will use same network as the Host ]  
In VirtualBox Setting --> Network:  
- **Attached to**: Bridge Adapter
- **Name**: [Your Newtork Device]  

I have encountered an <span style="color:red;">issue</span>:  
The virtual machine is connected to the network,   
but **there's no internet connection**.

In the Virtual Machine, Install the Following Package  
to be able to idetify the Virtual Machine IP and Gateway
```bash
sudo apt update
sudo apt install network-manager

sudo systemctl start NetworkManager
sudo systemctl enable NetworkManager

sudo systemctl status NetworkManager
```  
From the Status you will be able to Identify the device  
Here you go the output of the command   
In My case the device is wlo1  

```
● NetworkManager.service - Network Manager
     Loaded: loaded (/lib/systemd/system/NetworkManager.service; enabled; preset: enabled)
     Active: active (running) since Thu 2023-10-19 14:50:50 EEST; 6h ago
       Docs: man:NetworkManager(8)
   Main PID: 885 (NetworkManager)
      Tasks: 4 (limit: 14128)
     Memory: 11.3M
        CPU: 6.333s
     CGroup: /system.slice/NetworkManager.service
             └─885 /usr/sbin/NetworkManager --no-daemon

Oct 19 17:47:15 debian NetworkManager[885]: <info>  [1697726835.6571] dhcp4 (wlo1): state changed no lease
Oct 19 17:47:15 debian NetworkManager[885]: <info>  [1697726835.6572] dhcp4 (wlo1): activation: beginning transaction (timeout in 45 seconds)
Oct 19 17:47:15 debian NetworkManager[885]: <info>  [1697726835.6573] device (<span style="color:red;">**wlo1**</span>): ip:dhcp6: restarting
Oct 19 17:47:15 debian NetworkManager[885]: <info>  [1697726835.6573] dhcp6 (wlo1): canceled DHCP transaction
Oct 19 17:47:15 debian NetworkManager[885]: <info>  [1697726835.6574] dhcp6 (wlo1): state changed no lease
Oct 19 17:47:15 debian NetworkManager[885]: <info>  [1697726835.6574] dhcp6 (wlo1): activation: beginning transaction (timeout in 45 seconds)
Oct 19 17:47:15 debian NetworkManager[885]: <info>  [1697726835.6575] device (p2p-dev-wlo1): supplicant management interface state: 4way_handshake ->>
Oct 19 17:47:15 debian NetworkManager[885]: <info>  [1697726835.7564] dhcp4 (wlo1): state changed new lease, address=192.168.1.14
Oct 19 17:47:16 debian NetworkManager[885]: <info>  [1697726836.6765] dhcp6 (wlo1): state changed new lease
Oct 19 20:07:11 debian NetworkManager[885]: <info>  [1697735231.0436] agent-manager: agent[6f6f4357bd04ff3a,:1.74/org.gnome.Shell.NetworkAgent/1000]:>
```  
You need to obtain the following:  

The IP:
```bash  
sudo ip addr show <your-interface-name>
```
The Gateway
```bash  
sudo ip route 
```
By identifying the gateway,I realized the need for modification.  
The default setting was configured to use NAT, So I used  
the following commands.
```bash
sudo ip route del default
sudo ip route add default via 192.168.1.1 dev <your-interface-name>

```
To establish a connection from your host machine to the virtual machine,  
you must install the OpenSSH server by executing the following command:
```bash
sudo apt-get install openssh-server
```

Once the OpenSSH server is installed,   
you can SSH into the virtual machine from the host  
using the following command:
```bash
ssh -p <your_port> username@<VirtualMachine-IP>
```
Note: The default port for SSH is 22,  so if you are using the standard port,  
you can connect without specifying the port: 
```bash
ssh username@<VirtualMachine-IP>
```
This allows you to securely access and manage your virtual machine from  
the host machine using SSH.