# <h1> Task 3 - Create Your Value Contract </h1>

## Instal/Update dependencies
```
sudo apt update -y && sudo apt upgrade -y
sudo apt install -y make bzip2 automake libbz2-dev libssl-dev doxygen graphviz libgmp3-dev \
autotools-dev libicu-dev python2.7 python2.7-dev python3 python3-dev \
autoconf libtool curl zlib1g-dev sudo ruby libusb-1.0-0-dev \
libcurl4-gnutls-dev pkg-config patch llvm-7-dev clang-7 vim-common jq libncurses5 git
```
## 1. Clone Repositories
```
git clone --recursive https://github.com/inery-blockchain/inery.cdt
```

## 2. Export Path
```
export PATH="$PATH:$HOME/inery.cdt/bin:$HOME/inery-node/inery/bin"
```

## 3. Create & write code
```
mkdir -p $HOME/inrcrud
```
```
sudo tee $HOME/inrcrud/inrcrud.cpp >/dev/null <<EOF
#include <inery/inery.hpp>
#include <inery/print.hpp>
#include <string>

using namespace inery;

using std::string;

class [[inery::contract]] inrcrud : public inery::contract {
  public:
    using inery::contract::contract;


        [[inery::action]] void create( uint64_t id, name user, string data ) {
            records recordstable( _self, id );
            auto existing = recordstable.find( id );
            check( existing == recordstable.end(), "record with that ID already exists" );
            check( data.size() <= 256, "data has more than 256 bytes" );

            recordstable.emplace( _self, [&]( auto& s ) {
               s.id         = id;
               s.owner      = user;
               s.data       = data;
            });

            print( "Hello, ", name{user} );
            print( "Created with data: ", data );
        }

         [[inery::action]] void read( uint64_t id ) {
            records recordstable( _self, id );
            auto existing = recordstable.find( id );
            check( existing != recordstable.end(), "record with that ID does not exist" );
            const auto& st = *existing;
            print("Data: ", st.data);
        }

        [[inery::action]] void update( uint64_t id, string data ) {
            records recordstable( _self, id );
            auto st = recordstable.find( id );
            check( st != recordstable.end(), "record with that ID does not exist" );


            recordstable.modify( st, get_self(), [&]( auto& s ) {
               s.data = data;
            });

            print("Data: ", data);
        }

            [[inery::action]] void destroy( uint64_t id ) {
            records recordstable( _self, id );
            auto existing = recordstable.find( id );
            check( existing != recordstable.end(), "record with that ID does not exist" );
            const auto& st = *existing;

            recordstable.erase( st );

            print("Record Destroyed: ", id);

        }

  private:
    struct [[inery::table]] record {
       uint64_t        id;
       name     owner;
       string          data;
       uint64_t primary_key()const { return id; }
    };

    typedef inery::multi_index<"records"_n, record> records;
 };
EOF
```

## 4. Compile code

```
inery-cpp $HOME/inrcrud/inrcrud.cpp -o $HOME/inrcrud/inrcrud.wasm
```

### <b> before doing any transaction, make sure your wallet unlocked </b>

```
cline wallet unlock -n <wallet_name> --pasword <wallet_password>
```

## 5. Set Contract

```
cline set contract <account_name> ./inrcrud
```

## 6. Create Contract

```
cline push action <account_name> create '[1, "<account_name>", "My first Record"]' -p <account_name> --json
```

## 7. Read Contract

```
cline push action <account_name> read [1] -p <account_name> --json
```
## 8. Update

```
cline push action <account_name> read [1] -p <account_name> --json
```

## 9. Destroy

```
cline push action <account_name> destroy [1] -p <account_name> --json
```


# Create token and make transaction using your contract 



## 1. Clone Repositories
```
git clone https://github.com/inery-blockchain/inery.cdt
```


## 2. Edit bashrc
```
nano ~/.bashrc
```
hold arrow down on your keyboard, then paste this:

```
export PATH="$PATH:/root/inery.cdt/bin"
```

**SAVE**
`Ctrl`+ `X` then `Y` and `Enter`


```
cd; source .bashrc; cd -
```


## 3. Create Project

**a. create**
```
inery-init -bare=1 --project=neriitoken
```

**b. edit `neriitoken.hpp`**
```
cd ~/neriitoken
rm -f neriitoken.hpp
```
```
sudo tee $HOME/neriitoken/neriitoken.hpp <<EOF
#pragma once

#include <inery/asset.hpp>
#include <inery/inery.hpp>

#include <string>

namespace inerysystem {
   class system_contract;
}

namespace inery {

   using std::string;

   /**
    * The `inery.token` sample system contract defines the structures and actions that allow users to create, issue, and manage tokens for INERY based blockchains. It demonstrates one way to implement a smart contract which allows for creation and management of tokens. It is possible for one to create a similar contract which suits different needs. However, it is recommended that if one only needs a token with the below listed actions, that one uses the `inery.token` contract instead of developing their own.
    * 
    * The `inery.token` contract class also implements two useful public static methods: `get_supply` and `get_balance`. The first allows one to check the total supply of a specified token, created by an account and the second allows one to check the balance of a token for a specified account (the token creator account has to be specified as well).
    * 
    * The `inery.token` contract manages the set of tokens, accounts and their corresponding balances, by using two internal multi-index structures: the `accounts` and `stats`. The `accounts` multi-index table holds, for each row, instances of `account` object and the `account` object holds information about the balance of one token. The `accounts` table is scoped to an inery account, and it keeps the rows indexed based on the token's symbol.  This means that when one queries the `accounts` multi-index table for an account name the result is all the tokens that account holds at the moment.
    * 
    * Similarly, the `stats` multi-index table, holds instances of `currency_stats` objects for each row, which contains information about current supply, maximum supply, and the creator account for a symbol token. The `stats` table is scoped to the token symbol.  Therefore, when one queries the `stats` table for a token symbol the result is one single entry/row corresponding to the queried symbol token if it was previously created, or nothing, otherwise.
    */
   class [[inery::contract("neriitoken")]] neriitoken : public contract {
      public:
         using contract::contract;

         /**
          * Allows `issuer` account to create a token in supply of `maximum_supply`. If validation is successful a new entry in statstable for token symbol scope gets created.
          *
          * @param issuer - the account that creates the token,
          * @param maximum_supply - the maximum supply set for the token created.
          *
          * @pre Token symbol has to be valid,
          * @pre Token symbol must not be already created,
          * @pre maximum_supply has to be smaller than the maximum supply allowed by the system: 1^62 - 1.
          * @pre Maximum supply must be positive;
          */
         [[inery::action]]
         void create( const name&   issuer,
                      const asset&  maximum_supply);
         /**
          *  This action issues to `to` account a `quantity` of tokens.
          *
          * @param to - the account to issue tokens to, it must be the same as the issuer,
          * @param quntity - the amount of tokens to be issued,
          * @memo - the memo string that accompanies the token issue transaction.
          */
         [[inery::action]]
         void issue( const name& to, const asset& quantity, const string& memo );

         /**
          * The opposite for create action, if all validations succeed,
          * it debits the statstable.supply amount.
          *
          * @param quantity - the quantity of tokens to retire,
          * @param memo - the memo string to accompany the transaction.
          */
         [[inery::action]]
         void retire( const asset& quantity, const string& memo );

         /**
          * Allows `from` account to transfer to `to` account the `quantity` tokens.
          * One account is debited and the other is credited with quantity tokens.
          *
          * @param from - the account to transfer from,
          * @param to - the account to be transferred to,
          * @param quantity - the quantity of tokens to be transferred,
          * @param memo - the memo string to accompany the transaction.
          */
         [[inery::action]]
         void transfer( const name&    from,
                        const name&    to,
                        const asset&   quantity,
                        const string&  memo );
         /**
          * Allows `mem_payer` to create an account `owner` with zero balance for
          * token `symbol` at the expense of `mem_payer`.
          *
          * @param owner - the account to be created,
          * @param symbol - the token to be payed with by `mem_payer`,
          * @param mem_payer - the account that supports the cost of this action.
          *
          * More information can be read [here](https://github.com/INERY/inery.contracts/issues/62)
          * and [here](https://github.com/INERY/inery.contracts/issues/61).
          */
         [[inery::action]]
         void open( const name& owner, const symbol& symbol, const name& mem_payer );

         /**
          * This action is the opposite for open, it closes the account `owner`
          * for token `symbol`.
          *
          * @param owner - the owner account to execute the close action for,
          * @param symbol - the symbol of the token to execute the close action for.
          *
          * @pre The pair of owner plus symbol has to exist otherwise no action is executed,
          * @pre If the pair of owner plus symbol exists, the balance has to be zero.
          */
         [[inery::action]]
         void close( const name& owner, const symbol& symbol );

         static asset get_supply( const name& token_contract_account, const symbol_code& sym_code )
         {
            stats statstable( token_contract_account, sym_code.raw() );
            const auto& st = statstable.get( sym_code.raw() );
            return st.supply;
         }

         static asset get_balance( const name& token_contract_account, const name& owner, const symbol_code& sym_code )
         {
            accounts accountstable( token_contract_account, owner.value );
            const auto& ac = accountstable.get( sym_code.raw() );
            return ac.balance;
         }

         using create_action = inery::action_wrapper<"create"_n, &neriitoken::create>;
         using issue_action = inery::action_wrapper<"issue"_n, &neriitoken::issue>;
         using retire_action = inery::action_wrapper<"retire"_n, &neriitoken::retire>;
         using transfer_action = inery::action_wrapper<"transfer"_n, &neriitoken::transfer>;
         using open_action = inery::action_wrapper<"open"_n, &neriitoken::open>;
         using close_action = inery::action_wrapper<"close"_n, &neriitoken::close>;
      private:
         struct [[inery::table]] account {
            asset    balance;

            uint64_t primary_key()const { return balance.symbol.code().raw(); }
         };

         struct [[inery::table]] currency_stats {
            asset    supply;
            asset    max_supply;
            name     issuer;

            uint64_t primary_key()const { return supply.symbol.code().raw(); }
         };

         typedef inery::multi_index< "accounts"_n, account > accounts;
         typedef inery::multi_index< "stat"_n, currency_stats > stats;

         void sub_balance( const name& owner, const asset& value );
         void add_balance( const name& owner, const asset& value, const name& mem_payer );
   };

}
EOF
```







**c. edit `neriitoken.cpp`**
```
cd
cd ~/neriitoken
rm -f neriitoken.cpp
```

```
sudo tee $HOME/neriitoken/bakekok <<EOF
#include "neriitoken.hpp"

namespace inery {

void neriitoken::create( const name&   issuer,
                    const asset&  maximum_supply )
{
    require_auth( get_self() );

    auto sym = maximum_supply.symbol;
    check( sym.is_valid(), "NGETIK YANG BENER MAS" );
    check( maximum_supply.is_valid(), "NGETIK YANG BENER!!!");
    check( maximum_supply.amount > 0, "MAX SUPPLY NGGA BOLEH MINES");

    stats statstable( get_self(), sym.code().raw() );
    auto existing = statstable.find( sym.code().raw() );
    check( existing == statstable.end(), "NGETIK YANG BENER!!!" );

    statstable.emplace( get_self(), [&]( auto& s ) {
       s.supply.symbol = maximum_supply.symbol;
       s.max_supply    = maximum_supply;
       s.issuer        = issuer;
    });
}


void neriitoken::issue( const name& to, const asset& quantity, const string& memo )
{
    auto sym = quantity.symbol;
    check( sym.is_valid(), "NGETIK YANG BENER!!!" );
    check( memo.size() <= 256, "JANGAN PANJANG-PANJANG MEMO NYA" );

    stats statstable( get_self(), sym.code().raw() );
    auto existing = statstable.find( sym.code().raw() );
    check( existing != statstable.end(), "TOKEN DIBIKIN AJA BELOM" );
    const auto& st = *existing;
    check( to == st.issuer, "INI BUKAN CONTRACT MU" );

    require_auth( st.issuer );
    check( quantity.is_valid(), "YANG BENER DIKIT" );
    check( quantity.amount > 0, "NGETIK YANG BENER!!!" );

    check( quantity.symbol == st.supply.symbol, "NGETIK YANG BENER!!!" );
    check( quantity.amount <= st.max_supply.amount - st.supply.amount, "OVERLOAD");

    statstable.modify( st, same_payer, [&]( auto& s ) {
       s.supply += quantity;
    });

    add_balance( st.issuer, quantity, st.issuer );
}

void neriitoken::retire( const asset& quantity, const string& memo )
{
    auto sym = quantity.symbol;
    check( sym.is_valid(), "NGETIK YANG BENER!!!" );
    check( memo.size() <= 256, "JANGAN PANJANG-PANJANG MEMO NYA" );

    stats statstable( get_self(), sym.code().raw() );
    auto existing = statstable.find( sym.code().raw() );
    check( existing != statstable.end(), "NGGA ADA TOKENNYA!" );
    const auto& st = *existing;

    require_auth( st.issuer );
    check( quantity.is_valid(), "SALAH JUMLAH" );
    check( quantity.amount > 0, "NGGA BOLEH MINES" );

    check( quantity.symbol == st.supply.symbol, "SALAH SALAH" );

    statstable.modify( st, same_payer, [&]( auto& s ) {
       s.supply -= quantity;
    });

    sub_balance( st.issuer, quantity );
}

void neriitoken::transfer( const name&    from,
                      const name&    to,
                      const asset&   quantity,
                      const string&  memo )
{
    check( from != to, "TRANSFER KE ORANG LAIN LA" );
    require_auth( from );
    check( is_account( to ), "NGIRIM KE SIAPA, NGGA ADA NAMA GITUAN");
    auto sym = quantity.symbol.code();
    stats statstable( get_self(), sym.raw() );
    const auto& st = statstable.get( sym.raw() );

    require_recipient( from );
    require_recipient( to );

    check( quantity.is_valid(), "SALAH JUMLAH" );
    check( quantity.amount > 0, "NGGA BOLEH MINES" );
    check( quantity.symbol == st.supply.symbol, "SALAH SALAH" );
    check( memo.size() <= 256, "JANGAN PANJANG-PANJANG MEMO NYA" );

    auto payer = has_auth( to ) ? to : from;

    sub_balance( from, quantity );
    add_balance( to, quantity, payer );
}

void neriitoken::sub_balance( const name& owner, const asset& value ) {
   accounts from_acnts( get_self(), owner.value );

   const auto& from = from_acnts.get( value.symbol.code().raw(), "no balance object found" );
   check( from.balance.amount >= value.amount, "overdrawn balance" );

   from_acnts.modify( from, owner, [&]( auto& a ) {
         a.balance -= value;
      });
}

void neriitoken::add_balance( const name& owner, const asset& value, const name& mem_payer )
{
   accounts to_acnts( get_self(), owner.value );
   auto to = to_acnts.find( value.symbol.code().raw() );
   if( to == to_acnts.end() ) {
      to_acnts.emplace( mem_payer, [&]( auto& a ){
        a.balance = value;
      });
   } else {
      to_acnts.modify( to, same_payer, [&]( auto& a ) {
        a.balance += value;
      });
   }
}

void neriitoken::open( const name& owner, const symbol& symbol, const name& mem_payer )
{
   require_auth( mem_payer );

   check( is_account( owner ), "owner account does not exist" );

   auto sym_code_raw = symbol.code().raw();
   stats statstable( get_self(), sym_code_raw );
   const auto& st = statstable.get( sym_code_raw, "symbol does not exist" );
   check( st.supply.symbol == symbol, "symbol precision mismatch" );

   accounts acnts( get_self(), owner.value );
   auto it = acnts.find( sym_code_raw );
   if( it == acnts.end() ) {
      acnts.emplace( mem_payer, [&]( auto& a ){
        a.balance = asset{0, symbol};
      });
   }
}

void neriitoken::close( const name& owner, const symbol& symbol )
{
   require_auth( owner );
   accounts acnts( get_self(), owner.value );
   auto it = acnts.find( symbol.code().raw() );
   check( it != acnts.end(), "Balance row already deleted or never existed. Action won't have any effect." );
   check( it->balance.amount == 0, "Cannot close because the balance is not zero." );
   acnts.erase( it );
}

} /// namespace inery
EOF
```

## 4. Build
```
inery-cpp -abigen -o neriitoken.wasm neriitoken.cpp
```

## 5. Deploy Contract to Blockchain
```
cline set contract <account_name> ./ --json
```



## 6. Create, deploy, push, and retired your Token

**a. Create Token***
```
cline push action <account_name> create '["<account_name>", "<amount> <your_token_name>" , "creating my first tokens"]' -p <account_name>
```


**b. Deploy Token**
```
cline push action <account_name> issue '["<account_name>", "<amount> <your_token_name>", "Issuing my own tokens"]' -p <account_name>
```

**c. Push/Send Token**
```
cline push action <account_name> transfer '["<account_name>", "<target_account>", "<amount> <your_token_name>", "Here you go my token for free :) "]' -p <account_name>
```

**d. Retire Token**
```
cline push action <account_name> retire '["<amount> <your_token_name>","All set!"]' -p <account_name> --json
```


### **---------- CONFIGURATION -----------**

- `<your_name_wallet>`: your wallet name who created before
- `<account_name>`: your account name on [Inery dashboard](testnet.inery.io/dashboard)
- `<amount>`: how much it (without . or ,) ex: 1000000
- `<your_token_name>`: create your Token's name, in other case, use the name you created before
- `<target_account>`: account target who wants you sending your Tokens


### Finish the task

After completed all the tasks, you can click finish task **"Create Your Value Contract"** on [Inery dashboard](https://testnet.inery.io/dashboard)

![aaainery](https://user-images.githubusercontent.com/78480857/204692435-4caa53d4-949e-44cb-8f8b-910702ff55dd.png)


