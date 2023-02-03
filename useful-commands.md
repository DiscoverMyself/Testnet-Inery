# CHECK LOG

```
cd ~/inery-node/inery.setup/master.node
tail -f blockchain/nodine.log
```

# RESTART NODE

## Lite Node
```
cd ~/inery-node/inery.setup/lite.node/
./stop.sh
```
run again
```
./start.sh
```

## Master Node
```
cd ~/inery-node/inery.setup/master.node/
./stop.sh
```
run again
```
./start.sh
```

# DELETE NODE

## Lite Node
```
cd
pkill nodine
```
```
screen -X -S lite quit
```
```
cd ~/inery-node/inery.setup/lite.node/
./stop.sh
```
```
cd ~/inery-node/inery.setup/lite.node/
```
```
./clean.sh
```
```
cd
rm -rf inery-node
```

## Master Node
```
cd
pkill nodine
```
```
screen -X -S master quit
```
```
cd ~/inery-node/inery.setup/master.node/
./stop.sh
```
```
cd ~/inery-node/inery.setup/master.node/
```
```
./clean.sh
```
```
cd
rm -rf inery-node
```


# OPEN LOCKED WALLET
```
cline wallet unlock --name <your_wallet_name> --password <your_wallet_password>

```
**or** 

```
cline wallet unlock -n <your_wallet_name>
```

**then input your wallet password**


# CHECK YOUR WALLET PASSWORD

```
cd
cd ~/inery-node/inery.setup/
```
```
more file.txt
```

- if you change `file.txt` file name before, you must replace the command with file name that you given



# OPEN WALLET THAT UNLOCKED
```
cline wallet open --name <your_wallet_name>
```

# CHECK TRANSACTION ON BLOCKCHAIN

```
 cline get transaction TX_ID
```

# CHECK ACCOUNT INFORMATION

```
cline get account <account_name>
```

