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

# Build binary
make install

# Prepare cosmovisor directories
mkdir -p $HOME/.firmachain/cosmovisor/genesis/bin
ln -s $HOME/.firmachain/cosmovisor/genesis $HOME/.firmachain/cosmovisor/current -f

# Copy binary to cosmovisor directory
cp $(which firmachaind) $HOME/.firmachain/cosmovisor/genesis/bin

# Set node CLI configuration
firmachaind config chain-id colosseum-1
firmachaind config keyring-backend file
firmachaind config node tcp://localhost:16457

# Initialize the node
firmachaind init "Your Node Name" --chain-id colosseum-1

# Download genesis and addrbook files
curl -L https://snapshots.nodejumper.io/firmachain/genesis.json > $HOME/.firmachain/config/genesis.json
curl -L https://snapshots.nodejumper.io/firmachain/addrbook.json > $HOME/.firmachain/config/addrbook.json

# Set seeds
sed -i -e 's|^seeds *=.*|seeds = "f89dcc15241e30323ae6f491011779d53f9a5487@mainnet-seed1.firmachain.dev:26656,04cce0da4cf5ceb5ffc04d158faddfc5dc419154@mainnet-seed2.firmachain.dev:26656,940977bdc070422b3a62e4985f2fe79b7ee737f7@mainnet-seed3.firmachain.dev:26656,20e1000e88125698264454a884812746c2eb4807@seeds.lavenderfive.com:16456,8542cd7e6bf9d260fef543bc49e59be5a3fa9074@seed.publicnode.com:26656,b85358e035343a3b15e77e1102857dcdaf70053b@seeds.bluestake.net:24156,931a7c680d28c84a8a53e4017a6eae0788ee7cf2@firmachain.ramuchi.tech:57656,35b9e0a0818d2c5e9ef187984872c0ad2dbd447c@firma.peer.stavr.tech:1036,637077d431f618181597706810a65c826524fd74@firmachain.rpc.nodeshub.online:16456"|' $HOME/.firmachain/config/config.toml

# Set minimum gas price
sed -i -e 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.1ufct"|' $HOME/.firmachain/config/app.toml

# Set pruning
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "17"|' \
  $HOME/.firmachain/config/app.toml

# Enable prometheus
sed -i -e 's|^prometheus *=.*|prometheus = true|' $HOME/.firmachain/config/config.toml

# Change ports
sed -i -e "s%:1317%:16417%; s%:8080%:16480%; s%:9090%:16490%; s%:9091%:16491%; s%:8545%:16445%; s%:8546%:16446%; s%:6065%:16465%" $HOME/.firmachain/config/app.toml
sed -i -e "s%:26658%:16458%; s%:26657%:16457%; s%:6060%:16460%; s%:26656%:16456%; s%:26660%:16461%" $HOME/.firmachain/config/config.toml

# Download latest chain data snapshot
curl "https://snapshots.nodejumper.io/firmachain/firmachain_latest.tar.lz4" | lz4 -dc - | tar -xf - -C "$HOME/.firmachain"

# Install Cosmovisor
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.6.0

# Create a service
sudo tee /etc/systemd/system/firmachain.service > /dev/null << EOF
[Unit]
Description=FirmaChain node service
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.firmachain
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.firmachain"
Environment="DAEMON_NAME=firmachaind"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable firmachain.service

# Start the service and check the logs
sudo systemctl start firmachain.service
sudo journalctl -u firmachain.service -f --no-hostname -o cat
Secure Server Setup (Optional)

# generate ssh keys, if you don't have them already, DO IT ON YOUR LOCAL MACHINE
ssh-keygen -t rsa

# save the output, we'll use it later on instead of YOUR_PUBLIC_SSH_KEY
cat ~/.ssh/id_rsa.pub
# upgrade system packages
sudo apt update
sudo apt upgrade -y

# add new admin user
sudo adduser admin --disabled-password -q

# upload public ssh key, replace YOUR_PUBLIC_SSH_KEY with the key above
mkdir /home/admin/.ssh
echo "YOUR_PUBLIC_SSH_KEY" >> /home/admin/.ssh/authorized_keys
sudo chown admin: /home/admin/.ssh
sudo chown admin: /home/admin/.ssh/authorized_keys

echo "admin ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

# disable root login, disable password authentication, use ssh keys only
sudo sed -i 's|^PermitRootLogin .*|PermitRootLogin no|' /etc/ssh/sshd_config
sudo sed -i 's|^ChallengeResponseAuthentication .*|ChallengeResponseAuthentication no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PasswordAuthentication .*|PasswordAuthentication no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PermitEmptyPasswords .*|PermitEmptyPasswords no|' /etc/ssh/sshd_config
sudo sed -i 's|^#PubkeyAuthentication .*|PubkeyAuthentication yes|' /etc/ssh/sshd_config

sudo systemctl restart sshd

# install fail2ban
sudo apt install -y fail2ban

# install and configure firewall
sudo apt install -y ufw
sudo ufw default allow outgoing
sudo ufw default deny incoming
sudo ufw allow ssh
sudo ufw allow 9100
sudo ufw allow 26656

# make sure you expose ALL necessary ports, only after that enable firewall
sudo ufw enable

# make terminal colorful
sudo su - admin
source <(curl -s https://raw.githubusercontent.com/nodejumper-org/cosmos-scripts/master/utils/enable_colorful_bash.sh)

# update servername, if needed, replace YOUR_SERVERNAME with wanted server name
sudo hostnamectl set-hostname YOUR_SERVERNAME

# now you can logout (exit) and login again using ssh admin@YOUR_SERVER_IP
