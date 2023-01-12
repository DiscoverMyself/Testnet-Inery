</p>

<p align="center">
  <img height="50" height="auto" src="https://user-images.githubusercontent.com/38981255/184088981-3f7376ae-7039-4915-98f5-16c3637ccea3.PNG">
</p>


# Create new wallet
```
cd
cd ~/inery-node/inery.setup/
```
```
cline wallet create -n <your_name_wallet> -f file.txt
```

- `<your_name_wallet>`: create name for your wallet
- `file.txt` : name your password wallet file (optional, you can change or nah)


# Import config
```
cline wallet import --private-key <private_key> -n <your_name_wallet>

```
- `<private_key>`: your private key on [Inery dashboard](testnet.inery.io/dashboard)
- `<your_name_wallet>`: your wallet name who created before


# Register wallet as block producer
```
cline system regproducer <account_name> <public_key> 0.0.0.0:9010
```

- `<account_name>`: your account name on [Inery dashboard](testnet.inery.io/dashboard)
- `<public_key>`: your Inery wallet public key on [Inery dashboard](testnet.inery.io/dashboard)


# Approve wallet as block producer
```
cline system makeprod approve <account_name> <account_name>

```
- `<account_name>`: your account name on [Inery dashboard](testnet.inery.io/dashboard)



# usefull commands

## unlock wallet
```
cline wallet unlock -n <your_name_wallet>
```

- `<your_name_wallet>`: your wallet name who created before

## find wallet password
```
cd
cd ~/inery-node/inery.setup/
```
```
more file.txt
```

- if you change `file.txt` file name before, you must replace the command with file name that you given
