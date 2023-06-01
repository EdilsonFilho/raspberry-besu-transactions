# Using a Raspberry Pi 4 to Interact with a Blockchain Node

This repository is a quick tutorial on how we can use a raspberry to interact with a blockchain node that is on another machine. This interaction will be sending tokens from the wallet whose account is on the raspberry to another wallet.


### Raspberry Pi
The Raspberry Pi will serve as a physical layer, allowing us to interact with the Hyperledger Besu network using the created account. To begin, you'll need to download and install the desired Raspberry Pi OS (previously known as Raspbian). In our case, we'll use the Lite version since we're setting up a headless system without a monitor, mouse, or keyboard, and a graphical environment isn't necessary. Once you have the OS image, follow these steps:
Flash the SD card with the OS image. On Ubuntu, you can use the dd tool or any other suitable tool for this purpose. Enable SSH by creating an empty file named ssh in the boot partition of the SD card. If you plan to use a wireless connection, configure the Wi-Fi on the Raspberry Pi. In most cases, you'll need to modify the /etc/wpa_supplicant/wpa_supplicant.conf file on the rootfs partition to include the necessary network information. With these steps completed, you'll be ready to proceed with the Raspberry Pi setup for interacting with the Hyperledger Besu network.

- OS Raspberry pi OS Lite 32 bits version.
- 8Gb Ram
- Cable internet connection

#### Blockchain

We use Hyperledger Besu [1] as blockchain. We uploaded 3 validator nodes, 1 RPC node on a notebook. After uploading the blockchain network, we use the following command.

```sh
curl -X POST --data '{"jsonrpc":"2.0","method":"admin_nodeInfo","params":[],"id":1}' 127.0.0.1:8545
```
This command brings us information about the network. If everything is correct, information like chain_id, enode, listen addres and other data will appear.


## Preparation
- Check your Python version, use 3. Make sure you have pip, as I used raspberry OS Lite I installed it.
- Create the file: Requirements.txt
```sh
cytoolz==0.10.1
python-dotenv==0.14.0
web3==5.11.1
```
```sh
pip install requirements.txt
```
Check that the dependencies have been installed using:
```sh
pip list
```
## Development enviroment

1- Create the file: .env
```sh
sudo nano .env
```
```sh
WEB3_PROVIDER_URI=http://XX.XXX.XX.XXX:8545
SIGNER_LOCAL_PRIVATE_KEY=XXX
SIGNER_LOCAL_ADDRESS=0x173F1569C2Cb4626Cc6f073E94b6183848efd153
```

Notice: Where WEB3_PROVIDER_URI is the address of the HTTP RPC API (we're using port 8545), SIGNER_LOCAL_PRIVATE_KEY is the private key of our “Raspberry” account (which should be taken from MetaMask) and SIGNER_LOCAL_ADDRESS is the public address of the account. To extract the private key from MetaMask, first, select the desired account, and access “account details”, then click “export private key”. After confirming with our MetaMask password, we’ll get the key - just copy it and paste in place of “XXX” in the above `.env` file. 
Remember to substitute public address and WEB3 provider as well!

2- Application

Create the application:
```sh
sudo nano required-imports.py
```
Run
```sh
python3 required-imports.py 
```
```sh
import os
from dotenv import load_dotenv
from eth_account import Account
from web3.auto import w3



class EnvironmentManager:
   print("in environmnent manager class..  ")
   def __init__(self):
       load_dotenv()
  
   def getenv_or_raise(self, key):
       value = os.getenv(key)
       if value is None:
           raise Exception(f'{key} variable not found in the env')
       return value

class AccountManager:
   print("in account manager class ...")
   def __init__(self, private_key, address):
       self.private_key = private_key
       self.address = address

   def sign_transaction(self, transaction):
       signed = Account.sign_transaction(transaction_dict=transaction, private_key=self.private_key)
       return signed.rawTransaction
   
   def get_account_balance(self, web3):
       return web3.eth.getBalance(self.address)
  
   def get_transaction_count(self, web3):
       return web3.eth.getTransactionCount(self.address)
  
   def send_ether(self, web3, amount_to_send, receiver_address):
       transaction = {
           'nonce': self.get_transaction_count(web3),
           'gasPrice': web3.toWei(100, 'gwei'),
           'gas': 100000,
           'to': receiver_address,
           'value': amount_to_send,
           'data': b''
       }
       signed_txn = self.sign_transaction(transaction)
       hash_tx = web3.eth.sendRawTransaction(signed_txn)
       web3.eth.waitForTransactionReceipt(hash_tx.hex())
  
       return hash_tx.hex()
 
 env_manager = EnvironmentManager()
  
if not w3.isConnected():
   raise Exception(f'web3 connection failed')
  
  
env_manager = EnvironmentManager()
  
if not w3.isConnected():
   raise Exception(f'web3 connection failed')
  
  
address = env_manager.getenv_or_raise('SIGNER_LOCAL_ADDRESS')
print('Address signer is: ',address)
private_key = env_manager.getenv_or_raise('SIGNER_LOCAL_PRIVATE_KEY')
print('Private key is:',private_key)
account_manager = AccountManager(private_key, address)
print('about account: ',account_manager)
  
print('Balance before the transaction:', account_manager.get_account_balance(w3))
  
destination_address = '0x627306090abaB3A6e1400e9345bC60c78a8BEf57'
transaction_hash = account_manager.send_ether(w3, int(1e16), destination_address)
  
print('Hash of the transaction:', transaction_hash)
print('Balance after the transaction:', account_manager.get_account_balance(w3))
```


My genisis.json file:
```sh
{
  "genesis": {
    "config": {
      "chainId": 1337,
      "berlinBlock": 0,
      "ibft2": {
        "blockperiodseconds": 2,
        "epochlength": 30000,
        "requesttimeoutseconds": 4
      }
    },
    "nonce": "0x0",
    "timestamp": "0x58ee40ba",
    "gasLimit": "0x47b760",
    "difficulty": "0x1",
    "mixHash": "0x63746963616c2062797a616e74696e65206661756c7420746f6c6572616e6365",
    "coinbase": "0x0000000000000000000000000000000000000000",
    "alloc": {
      "fe3b557e8fb62b89f4916b721be55ceb828dbd73": {
        "privateKey": "8f2a55949038a9610f50fb23b5883af3b4ecb3c3bb792cbcefbd1542c692be63",
        "comment": "private key and this comment are ignored.  In a real chain, the private key should NOT be stored",
        "balance": "0xad78ebc5ac6200000"
      },
      "627306090abaB3A6e1400e9345bC60c78a8BEf57": {
        "privateKey": "c87509a1c067bbde78beb793e6fa76530b6382a4c0241e5e4a9ec0a0f44dc0d3",
        "comment": "private key and this comment are ignored.  In a real chain, the private key should NOT be stored",
        "balance": "90000000000000000000000"
      },
      "f17f52151EbEF6C7334FAD080c5704D77216b732": {
        "privateKey": "ae6ae8e5ccbfb04590405997ee2d52d2b330726137b875053c36d94e974d162f",
        "comment": "private key and this comment are ignored.  In a real chain, the private key should NOT be stored",
        "balance": "90000000000000000000000"
      }
    }
  },
  "blockchain": {
    "nodes": {
      "generate": true,
      "count": 4
    }
  }
}
```

This is account 2 that I imported into metamask using data from gensis.json file. Note that the gensis.json file is in the documentation. In a professional project do not use these addresses and do not leave the addresses visible!

![add net](/images/account2.jpg)


Figure 1- Account 2 sent resources.

![add net](/images/account3.jpg)


Figure 2- Account 3 received funds.



## References
[1] https://github.com/hyperledger/besu/

[2] https://wiki.hyperledger.org/display/BESU/Deployment+of+Private+Hyperledger+Besu+on+AWS+with+hardware+layer+for+Externally+Owned+Account


