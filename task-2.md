# Inery Testnet Task 2 - Make your own currency and transfer to someone

## 1. Unlock wallet
```
cline wallet unlock -n <your_name_wallet>
```

## 2. Generate `abi.wasm` and `token.wasm` file

**1. generate**
```
cline get code inery.token -c token.wasm -a token.abi --wasm
```

**2. Set `wasm` to your account**
```
cline set code -j <account_name> token.wasm
```

**3. Set `abi` to your account**
```
cline set abi <account_name> token.abi
```

## 3. Create, deploy and push your Token

**1. Create Token***
```
cline push action inery.token create '["<account_name>", "<amount> <your_token_name>" , "creating my first tokens"]' -p <account_name>
```


**2. Deploy Token**
```
cline push action inery.token issue '["<account_name>", "<amount> <your_token_name>", "Issuing my own tokens"]' -p <account_name>
```

**3. Push/Send Token**
```
cline push action inery.token transfer '["<account_name>", "<target_account>", "<amount> <your_token_name>", "Here you go my token for free :) "]' -p <account_name>
```

IMPORTANT:
you must send your token to `inery` as target account, and send to 10 another users too (you can check thei account name on explorer https://explorer.inery.io)

---------- CONFIGURATION -----------

- `<your_name_wallet>`: your wallet name who created before
- `<account_name>`: your account name on [Inery dashboard](testnet.inery.io/dashboard)
- `<amount>`: how much it (without . or ,) ex: 1000000
- `<your_token_name>`: create your Token's name, in other case, use the name you created before
- `<target_account>`: account target who wants you sending your Tokens


### Finish the task

After completed all the tasks, you can click finish task "Make your own currency and transfer to someone" on [Inery dashboard](https://testnet.inery.io/dashboard)

![aaainery](https://user-images.githubusercontent.com/78480857/204692435-4caa53d4-949e-44cb-8f8b-910702ff55dd.png)
