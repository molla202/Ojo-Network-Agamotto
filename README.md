<img width="1024" alt="header@2x" src="https://github.com/molla202/Ojo-Network/assets/91562185/979e2833-c9c2-4e36-90ba-0a1ac7ad2bd0">


<h1 align="center"> Ojo Network </h1>

 * [Topluluk kanalımız](https://t.me/corenodechat)<br>
 * [Topluluk Twitter](https://twitter.com/corenodeHQ)<br>
 * [Ojo Network Website](https://ojo.network/)<br>
 * [Blockchain Explorer](https://explorer.kjnodes.com/ojo-testnet/staking)<br>
 * [Discord](https://discord.gg/tygWr7JM2w)<br>
 * [Twitter](https://twitter.com/ojo_network)<br>


## Sistem Gereksinimleri
| Bileşenler | Minimum Gereksinimler | 
| ------------ | ------------ |
| CPU |	4 |
| RAM	| 8 GB |
| Storage	| 250 GB SSD |

## Oto kurulum
```
curl -sSL -o ojo.sh https://raw.githubusercontent.com/molla202/Ojo-Network-Agamotto/main/ojo.sh && chmod +x ojo.sh && bash ./ojo.sh
```
## Update & Kütüphane 
```
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y
```
## Go kurulumu
```
cd $HOME
! [ -x "$(command -v go)" ] && {
VER="1.19.3"
wget "https://golang.org/dl/go$VER.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go$VER.linux-amd64.tar.gz"
rm "go$VER.linux-amd64.tar.gz"
[ ! -f ~/.bash_profile ] && touch ~/.bash_profile
echo "export PATH=$PATH:/usr/local/go/bin:~/go/bin" >> ~/.bash_profile
source $HOME/.bash_profile
}
[ ! -d ~/go/bin ] && mkdir -p ~/go/bin
```
## Varyasyonları atayalım
Not: cüzdan adı ve node adı değiştiriniz.
```
echo "export WALLET="cüzdan-adı"" >> $HOME/.bash_profile
echo "export MONIKER="node-adı"" >> $HOME/.bash_profile
echo "export OJO_CHAIN_ID="agamotto"" >> $HOME/.bash_profile
echo "export OJO_PORT="12"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## binary dosyaları indirelim
```
cd $HOME
rm -rf ojo
git clone https://github.com/ojo-network/ojo.git
cd ojo
git checkout v0.3.0-rc2
make install
```
## init işlemi
Not: Molla202 kendi adınızla değiştiriniz.
```
ojod config node tcp://localhost:${OJO_PORT}657
ojod config keyring-backend os
ojod config chain-id agamotto
ojod init "molla202" --chain-id agamotto
```
## genesis ve addrbook indirelim
```
curl -Ls https://snapshots.kjnodes.com/ojo-testnet/genesis.json > $HOME/.ojo/config/genesis.json
curl -Ls https://snapshots.kjnodes.com/ojo-testnet/addrbook.json > $HOME/.ojo/config/addrbook.json
```
## seeds ve peers ekleyelim
```
sed -i -e "s|^seeds *=.*|seeds = \"3f472746f46493309650e5a033076689996c8881@ojo-testnet.rpc.kjnodes.com:15059\"|" $HOME/.ojo/config/config.toml
```
## port yapılandırması app.toml
```
sed -i.bak -e "s%:1317%:${OJO_PORT}317%g;
s%:8080%:${OJO_PORT}080%g;
s%:9090%:${OJO_PORT}090%g;
s%:9091%:${OJO_PORT}091%g;
s%:8545%:${OJO_PORT}545%g;
s%:8546%:${OJO_PORT}546%g;
s%:6065%:${OJO_PORT}065%g" $HOME/.ojo/config/app.toml
```
## port yapılandırması config.toml
```
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${OJO_PORT}658\"%; 
s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${OJO_PORT}657\"%; 
s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${OJO_PORT}060\"%;
s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${OJO_PORT}656\"%;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${OJO_PORT}656\"%;
s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${OJO_PORT}660\"%" $HOME/.ojo/config/config.toml
```
## puring
```
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.ojo/config/app.toml
```
## gas ayarı prometheus, index kapayalım
```
sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0uojo\"|" $HOME/.ojo/config/app.toml
```
## servis dosyası oluşturalım
```
sudo tee /etc/systemd/system/ojod.service > /dev/null <<EOF
[Unit]
Description=Ojo node
After=network-online.target
[Service]
User=$USER
WorkingDirectory=$HOME/.ojo
ExecStart=$(which ojod) start --home $HOME/.ojo
Restart=on-failure
RestartSec=5
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
## servisleeri başlatalım 
```
sudo systemctl daemon-reload
sudo systemctl enable ojod
sudo systemctl restart ojod
```
## Logları kontrol edelim
```
sudo journalctl -u ojod -fo cat
```
## validator kurulumu (cüzdan adı ve validator adınızı değiştiriniz)
not: başlamadık daha
```
ojod tx staking create-validator \
--amount 1000000uojo \
--from cüzdan-adı \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(ojod tendermint show-validator) \
--moniker "node-adı" \
--identity "" \
--details "" \
--chain-id agamotto \
--gas auto --gas-adjustment 1.5 \
-y
```
