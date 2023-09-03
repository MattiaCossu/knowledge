# Responder

The "Responder" is a tool that can be used to initiate poisoning attacks on network protocols such as LLMNR (Link-Local Multicast Name Resolution), NBT-NS (NetBIOS Name Service) and MDNS (Multicast DNS). This tool is designed to respond to specific NBT-NS queries based on the name suffix (as shown in http://support.microsoft.com/kb/163409).

By default, Responder will only respond to File Server Service queries, which is used for the SMB (Server Message Block) protocol. The concept behind this default configuration is to target responses and operate more stealthily on the network. This also helps to avoid interfering with the legitimate behavior of the NBT-NS protocol. However, you can configure the "-r" option via the command line if you want to respond to Workstation Service requests with a specific name suffix.

In other words, Responder is a tool used to fraudulently respond to certain service requests on the network, trying to be selective and discreet to avoid arousing suspicion and causing unwanted interference on legitimate network communications.

## Download 
```bash
git clone https://github.com/lgandx/Responder
```

## Identifying our Setup 
To ensure Responder does not conict with other tools such as Ubuntu's built-in DNS server,
we must make sure to properly congure its network conguration. Responder allows us to
specify which network interface and address to use, which we will dene during this step.

Using the ip command in combination with grep , we can lter-out all interfaces which are
not of interest.

```bash
ip a | grep -B 2 192.168.20.109
```

## Starting Ressponder 
We now have all the needed information to spin up Responder. To do so, run the
Responder.py le from within the downloaded repository with elevated privileges.

```python
sudo python3 ./Responder/Responder.py -I ens192 -i 192.168.20.109 -wPvF
```
