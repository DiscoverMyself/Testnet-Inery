<p align="center">
  <img height="50" height="auto" src="https://user-images.githubusercontent.com/38981255/184088981-3f7376ae-7039-4915-98f5-16c3637ccea3.PNG">
</p>

# Tutorial Become a Master Node

**Official Document :**
> [Node Lite & Master](https://docs.inery.io/docs/category/lite--master-nodes)

**Explorer :**
> [Explorer Inery](https://explorer.inery.io/ "Explorer Inary")

## 1. Update
```
sudo apt-get update && sudo apt install git && sudo apt install screen
```
```
sudo apt-get install -y make bzip2 automake libbz2-dev libssl-dev doxygen graphviz libgmp3-dev \
autotools-dev libicu-dev python2.7 python2.7-dev python3 python3-dev \
autoconf libtool curl zlib1g-dev sudo ruby libusb-1.0-0-dev \
libcurl4-gnutls-dev pkg-config patch llvm-7-dev clang-7 vim-common jq libncurses5
```
## 2. Open Port
```
ufw allow 22 && ufw allow 8888 && ufw allow 9010 && ufw enable
```
## 3. Clone repository
```
git clone https://github.com/inery-blockchain/inery-node
```
## 4. Allow file access
```
cd inery-node/inery.setup
```
```
chmod +x ine.py
```
```
./ine.py --export
```
```
cd; source .bashrc; cd -
```
## 6. Become a Master Node
Set your Configuration here
```
cd ~/inery-node/inery.setup/
```
```
cd tools
```
```
nano config.json
```
Change only on **MASTER NODE** part:

the informations below appeared on [dashboard](testnet.inery.io/dashboard):

- `IP`: your IP
- `account name`: your inery account name
- `public key`: your inery public key
- `private key`: your inery private key

**SAVE**

`Ctrl`+`X` then `Y` and `Enter`


## 7. Start blockchain protocol
```
cd
cd ~/inery-node/inery.setup/master.node/
```
```
screen -R master
```
```
./ine.py --master
```
**detached the screen (running in background)**
`Ctrl`+ `A` `D`

**if you want to open again use:**
```
screen -Rd master
```

**or, if you want to delete:**
```
screen -X -S master quit
```

<p align="center">
  <img height="auto" height="auto" src="https://user-images.githubusercontent.com/38981255/184091596-3a11bd09-7b26-4cd9-a444-a14facf332a3.PNG">
</p>

just wait till block get synced,

<p align="center">
  <img height="auto" height="auto" src="https://user-images.githubusercontent.com/38981255/184104361-73d223ce-0f70-408d-bec8-7aefea128dc6.png">
</p>

## 8. Run Node
in new tab:

```
cd
cd ~/inery-node/inery.setup/master.node/
```
```
chmod +x start.sh
```
```
./start.sh
```

<p align="center">
  <img height="auto" height="auto" src="https://user-images.githubusercontent.com/38981255/184124841-87e95c29-2a1c-4d7e-beac-31a35549869e.PNG">
</p>


if it synced, will shows `received block...` logs

**ALL SET!**
