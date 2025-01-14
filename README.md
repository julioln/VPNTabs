# VPN Container Tabs
Force the traffic of a Firefox container tab to pass through a VPN

On newer version of Firefox, it is possible to use [container tabs](https://support.mozilla.org/en-US/kb/containers), which isolate the cookies from the normal tabs. It is a good feature, since by isolating the cookies you can avoid tracking and some types of attacks that could steal your cookies, in addition to being able to log in to multiple accounts on the same website.

Furthermore, it is possible to assign different proxies to be used in different containers using [Container proxy](https://addons.mozilla.org/en-US/firefox/addon/container-proxy/).

Then, we can create a docker container with a VPN client and a proxy server running. By doing this, we cab assign our docker proxy to a container tab so that all the traffic of that tab goes through the VPN that is running only inside the docker container.

## Usage


### 1. Install docker
[See instructions here](https://docs.docker.com/engine/install/)

### 2. Clone the repository
```
git clone https://github.com/Nickguitar/VPNTabs
```
**Important note:** if your user doesn't have permission to run docker containers you will need to run the scripts with `sudo`
### 3. Building the docker image and the `setup.sh`
```
cd VPNTabs
```
```
./build.sh
```
*[Alternative] Building only the docker image*
```
cd VPNTabs
```
```
./container-build.sh
```
### 4. Place your VPN files in the diretory `ovpn_files`
```
cp /path/to/vpn/file ovpn_files/
```

### 5. Running with the `setup.sh`
```
./setup.sh <your_vpn_file> [portnumber]
```
*[Alternative] Instead using* `setup.sh` *You can run your custom script or use docker-compose, here is an example:*
```
docker run -d --rm \
--cap-add=NET_ADMIN \
--device /dev/net/tun \
--sysctl net.ipv6.conf.all.disable_ipv6=0 \
-p 3128:3128 \
-e OVPN_FILE=<YOUR_VPN_FILE_HERE> \
-v <PATH_OF_VPN_FILES_DIRECTORY_HERE>:/ovpn \
squid_openvpn:1.0
```
*The envoriment variable* `OVPN_FILE` *is used to know which file should openvpn use*


### If everything is ok, you should see port 3128 (or anyone you chose) listening on your machine.
```
$ netstat -tapeno | grep 3128
tcp     0    0 0.0.0.0:3128       0.0.0.0:*    LISTEN    0   5767579  -  off (0.00/0/0)
```

### 6. Add Multi Account Containers and Container Proxy to Firefox

- [Multi Account Containers](https://addons.mozilla.org/en-US/firefox/addon/multi-account-containers/)
- [Container Proxy](https://addons.mozilla.org/en-US/firefox/addon/container-proxy/) 

### 7. Set the proxy on Container Proxy
Open Container Proxy, click on "Proxy", set protocol to HTTP, server to 127.0.0.1 and use the port you chose in step 2.3 (the default port is 3128). Also, uncheck the checkbox "Do not proxy local addresses". Then, click "save".

![image](https://user-images.githubusercontent.com/3837916/136625420-925f7d61-41c1-4b41-aa41-abea137475b7.png)

Now, click on "Assign" and change the proxy of the container to the one you've just created.
![image](https://user-images.githubusercontent.com/3837916/136626051-4b05ea82-bae4-427e-875b-4b959308d6e9.png)


To use the container tab with VPN, right click the new tab button and choose the container for which you configured the proxy

![image](https://user-images.githubusercontent.com/3837916/136625934-b389fba1-db40-43a2-9066-92e1bd657555.png)


### You're done

Now every website you access using those container tabs will pass through your local proxy, which points to a docker container whose traffic pass through your VPN. =)

### Comments

- You can generate as many containers as you want, each one running a different VPN config file. In this way, it is possible to have multiple container tabs, each with a different VPN.
- Note that this doesn't have a kill switch. If your VPN goes down and you access some website within the container tab, your IP will be exposed. It's at your own risk.
- Since the VPN client is running inside a docker container, all your other network traffic isn't being tunneled through the VPN. The only connections going through the VPN are those pointing to the local proxy you've created.
