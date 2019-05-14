# BlockChain-Sample
Block chained for engineers

### Environment Setting

cd C:\Users\Sunghyun\AppData\Roaming\Ethereum Wallet\binaries\Geth\geth-windows-amd64-1.8.23-c9427004

geth --datadir home\eth_private_net init home\eth_private_net\genesis.json

geth --networkid "10" --nodiscover --datadir "home\eth_private_net" --rpc --rpcaddr "localhost" --rpcport "8545" --rpccorsdomain "*" --rpcapi "eth,net,web3,personal" --targetgaslimit "20000000" console 2>> home\eth_private_net\geth_err.log

