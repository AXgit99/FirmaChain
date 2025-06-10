Hardware Requirements

Minimum

3CPU 4RAM 100GB
Recommended

4CPU 8RAM 200GB
Rent On Hetzner | Rent On OVH
Dependencies Installation

**Install dependencies for building from source**
```
sudo apt update
sudo apt install -y curl git jq lz4 build-essential
```

**Install Go**
```
sudo rm -rf /usr/local/go
curl -L https://go.dev/dl/go1.22.7.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> $HOME/.profile
source .profile
```

Node Installation

Node Name

Your Node Name
Port prefix

164
**Clone project repository**
```
cd && rm -rf firmachain
git clone https://github.com/firmachain/firmachain
cd firmachain
git checkout 0.3.5-patch
```

**Build binary**
```
make install
```

**Prepare cosmovisor directories**
```
mkdir -p $HOME/.firmachain/cosmovisor/genesis/bin
ln -s $HOME/.firmachain/cosmovisor/genesis $HOME/.firmachain/cosmovisor/current -f
```

**Copy binary to cosmovisor directory**
```
cp $(which firmachaind) $HOME/.firmachain/cosmovisor/genesis/bin
```

**Set node CLI configuration**
```
firmachaind config chain-id colosseum-1
firmachaind config keyring-backend file
firmachaind config node tcp://localhost:16457
```

**Initialize the node**
```
firmachaind init "Your Node Name" --chain-id colosseum-1
```

**Download genesis and addrbook files**
```
curl -L https://snapshots.nodejumper.io/firmachain/genesis.json > $HOME/.firmachain/config/genesis.json
curl -L https://snapshots.nodejumper.io/firmachain/addrbook.json > $HOME/.firmachain/config/addrbook.json
```

**Set seeds**
```
sed -i -e 's|^seeds *=.*|seeds = "f89dcc15241e30323ae6f491011779d53f9a5487@mainnet-seed1.firmachain.dev:26656,04cce0da4cf5ceb5ffc04d158faddfc5dc419154@mainnet-seed2.firmachain.dev:26656,940977bdc070422b3a62e4985f2fe79b7ee737f7@mainnet-seed3.firmachain.dev:26656,20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:16456,8542cd7e6bf9d260fef543bc49e59be5a3fa9074@seed.publicnode.com:26656,b85358e035343a3b15e77e1102857dcdaf70053b@seeds.bluestake.net:24156,931a7c680d28c84a8a53e4017a6eae0788ee7cf2@firmachain.ramuchi.tech:57656,35b9e0a0818d2c5e9ef187984872c0ad2dbd447c@firma.peer.stavr.tech:1036,637077d431f618181597706810a65c826524fd74@firmachain.rpc.nodeshub.online:16456"|' $HOME/.firmachain/config/config.toml
```

**Set minimum gas price**
Commit 1 line
Commit 2 line
Commit 3 line
Commit 4 line
Commit 5 line
Commit 6 line
Commit 7 line
Commit 8 line
