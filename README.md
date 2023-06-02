![242064548-f5e9add1-5b55-40d2-83dd-539cbf64c266](https://github.com/molla202/empower-2/assets/91562185/1191c65e-441f-4d71-9acc-5ea06391b7ed)

<h1 align="center"> Empower Chain </h1>

> Topluluk kanalları: - [Sohbet](https://t.me/corenodechat) - [Empower Discord](https://discord.gg/Zs3GMUhg)

<h1 align="center"> Donanım </h1>

```sh
# Sistem
4 CPU
8 RAM
200 SSD
```

<h1 align="center"> Güncelleme </h1>

```
sudo apt update && sudo apt upgrade -y && \
sudo apt install curl tar wget clang pkg-config libssl-dev jq build-essential bsdmainutils git make ncdu gcc git jq chrony liblz4-tool -y
```

<h1 align="center"> Go kurulumu </h1>

```sh
sudo rm -rf /usr/local/go
wget https://golang.org/dl/go1.20.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.20.linux-amd64.tar.gz
export PATH=/usr/local/go/bin:$PATH

# go version 1.20 olmak zorunda yoksa hata alırsınız
go version
```

<h1 align="center"> Binary </h1>

```sh 
cd $HOME
rm -rf empowerchain

git clone https://github.com/EmpowerPlastic/empowerchain

cd empowerchain
cd chain

make install
```
```
# Port Atama (izmir ayarladım isteyen 35 değiştirsin- moniker adınızı değiştirin test olanı değiştirceniz)
echo "export MONIKER="test"" >> $HOME/.bash_profile
echo "export EMPOWERD_PORT="35"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
# config and init app işlemini yapalım
empowerd config chain-id circulus-1
empowerd config keyring-backend test
empowerd config node tcp://localhost:${EMPOWERD_PORT}057
empowerd init $MONIKER --chain-id circulus-1
```
<h1 align="center"> Genesis, addrbook ve servis (port ayarlaması var isteyen değiştirsin 35 ayarlı- monilker isminide değiştiriniz test olanı)</h1>

```sh
# Orjinal dökümasyonda genesis URL'ler yok, manuel ekliyorum:
curl -Ls https://ss-t.empower.nodestake.top/genesis.json > $HOME/.empowerchain/config/genesis.json 

curl -Ls https://ss-t.empower.nodestake.top/addrbook.json > $HOME/.empowerchain/config/addrbook.json



# Port app.toml ayarlaması
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${EMPOWERD_PORT}317\"%;
s%^address = \":8080\"%address = \":${EMPOWERD_PORT}080\"%;
s%^address = \"localhost:9090\"%address = \"localhost:${EMPOWERD_PORT}090\"%; 
s%^address = \"localhost:9091\"%address = \"localhost:${EMPOWERD_PORT}091\"%; 
s%^address = \"localhost:8545\"%address = \"localhost:${EMPOWERD_PORT}545\"%; 
s%^ws-address = \"localhost:8546\"%ws-address = \"localhost:${EMPOWERD_PORT}546\"%" $HOME/.empowerchain/config/app.toml

# Ports config.toml ayarlaması
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${EMPOWERD_PORT}658\"%; 
s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${EMPOWERD_PORT}657\"%; 
s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${EMPOWERD_PORT}060\"%;
s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${EMPOWERD_PORT}656\"%;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${EMPOWERD_PORT}656\"%;
s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${EMPOWERD_PORT}660\"%" $HOME/.empowerchain/config/config.toml
```
# seed peer ve gas ayarları
```
seeds="d6a7cd9fa2bafc0087cb606de1d6d71216695c25@51.159.161.174:26656"
peers="e8b3fa38a15c426e046dd42a41b8df65047e03d5@95.217.144.107:26656,89ea54a37cd5a641e44e0cee8426b8cc2c8e5dfb@51.159.141.221:26656,0747860035271d8f088106814a4d0781eb7b2bc7@142.132.203.60:27656,3c758d8e37748dc692621a0d59b454bacb69b501@65.108.224.156:26656,41b97fced48681273001692d3601cd4024ceba59@5.9.147.185:26656"
sed -i -e 's|^seeds *=.*|seeds = "'$seeds'"|; s|^persistent_peers *=.*|persistent_peers = "'$peers'"|' $HOME/.empowerchain/config/config.toml
sed -i 's/max_num_inbound_peers =.*/max_num_inbound_peers = 50/g' $HOME/.empowerchain/config/config.toml
sed -i 's/max_num_outbound_peers =.*/max_num_outbound_peers = 50/g' $HOME/.empowerchain/config/config.toml
sed -i.bak -e "s/^minimum-gas-prices *=.*/minimum-gas-prices = \"0.025umpwr\"/" $HOME/.empowerchain/config/app.toml
```
```
# Servis dosyası:
sudo tee /etc/systemd/system/empowerd.service > /dev/null <<EOF
[Unit]
Description=empowerd Daemon
After=network-online.target
[Service]
User=$USER
ExecStart=/root/go/bin/empowerd start
Restart=always
RestartSec=3
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF
```
```
# Resetleme ve başlatma
sudo systemctl daemon-reload
sudo systemctl enable empowerd
sudo systemctl restart empowerd
# Log Kontrol
journalctl -u empowerd -f -o cat
```

<h1 align="center"> Cüzdan oluşturma </h1>

```sh
# cüzdan-adı değiştirin.
empowerd keys add cüzdan-adı
```
```sh
empowerd keys add cüzdan-adı --recover    (import etmek için)
```

### Senkronize olmayı bekleyin ardından validatör oluşturun (not: faucetin bir kaç gün içinde açılacağı söylendi)
```
empowerd tx staking create-validator \
  --amount 1000000umpwr \
  --from cüzdan-adı \
  --commission-max-change-rate "0.01" \
  --commission-max-rate "0.2" \
  --commission-rate "0.1" \
  --min-self-delegation "1" \
  --pubkey  $(empowerd tendermint show-validator) \
  --moniker node-adı \
  --website "websiteniz"
  --identity keybase.io idniz \
  --details "Core Node Community" \
  --chain-id circulus-1
  --y
```
## Restart node
```
sudo systemctl restart empowerd
```
## Log Kontrol
```
journalctl -u empowerd -f -o cat
```
# Sadece port değiştirmek isteyenler

# Port Atama (izmir ayarladım isteyen 35 değiştirsin)
```
echo "export EMPOWERCHAİN_PORT="35"" >> $HOME/.bash_profile
source $HOME/.bash_profile
```
```
systemctl stop empowerd
```
```
# Port app.toml ayarlaması
sed -i.bak -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:${EMPOWERCHAİN_PORT}317\"%;
s%^address = \":8080\"%address = \":${EMPOWERCHAİN_PORT}080\"%;
s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:${EMPOWERCHAİN_PORT}090\"%; 
s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:${EMPOWERCHAİN_PORT}091\"%; 
s%^address = \"0.0.0.0:8545\"%address = \"0.0.0.0:${EMPOWERCHAİN_PORT}545\"%; 
s%^ws-address = \"0.0.0.0:8546\"%ws-address = \"0.0.0.0:${EMPOWERCHAİN_PORT}546\"%" $HOME/.empowerchain/config/app.toml
```
```
# Ports config.toml ayarlaması
sed -i.bak -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:${EMPOWERCHAİN_PORT}658\"%; 
s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://0.0.0.0:${EMPOWERCHAİN_PORT}657\"%; 
s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:${EMPOWERCHAİN_PORT}060\"%;
s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:${EMPOWERCHAİN_PORT}656\"%;
s%^external_address = \"\"%external_address = \"$(wget -qO- eth0.me):${EMPOWERCHAİN_PORT}656\"%;
s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":${EMPOWERCHAİN_PORT}660\"%" $HOME/.empowerchain/config/config.toml
```
```
sudo systemctl daemon-reload
sudo systemctl restart empowerd
journalctl -u empowerd -f -o cat
```
