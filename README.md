<h1 align="center">Initia</h1>

> Teşvikli olacak. Aktif validatör olma ihtimaliniz yok gibi fakat DC'de ek görevler olacağını söylediler. User tarafından da katılınabilinir -anlık 500k küsür wallet var.

> User tarafından katılmak isterseniz: https://app.testnet.initia.xyz/xp Ref: QNHLDUD8

> [Validator formu](https://docs.google.com/forms/d/e/1FAIpQLSc09Kl6mXyZHOL12n_6IUA8MCcL6OqzTqsoZn9N8gpptoeU_Q/viewform)

<h1 align="center">Sistem Gereksinimleri</h1>


```
4 CPU - 8 RAM 600 GB SSD 
```

<h1 align="center">Kurulum</h1>


```console
sudo apt update && sudo apt upgrade -y
sudo apt install curl git wget htop tmux build-essential jq make lz4 gcc unzip -y

sudo rm -rf /usr/local/go
curl -Ls https://go.dev/dl/go1.21.10.linux-amd64.tar.gz | sudo tar -xzf - -C /usr/local
eval $(echo 'export PATH=$PATH:/usr/local/go/bin' | sudo tee /etc/profile.d/golang.sh)
eval $(echo 'export PATH=$PATH:$HOME/go/bin' | tee -a $HOME/.profile)


git clone https://github.com/initia-labs/initia.git
cd initia

# build
make build

mkdir -p $HOME/.initia/cosmovisor/genesis/bin
mv build/initiad $HOME/.initia/cosmovisor/genesis/bin/
rm -rf build


sudo ln -s $HOME/.initia/cosmovisor/genesis $HOME/.initia/cosmovisor/current -f
sudo ln -s $HOME/.initia/cosmovisor/current/bin/initiad /usr/local/bin/initiad -f

go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.5.0
```

<h1 align="center">Node</h1>


```console

# Tek seferde copy paste
sudo tee /etc/systemd/system/initia.service > /dev/null << EOF
[Unit]
Description=initia node service
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=10
LimitNOFILE=65535
Environment="DAEMON_HOME=$HOME/.initia"
Environment="DAEMON_NAME=initiad"
Environment="UNSAFE_SKIP_BACKUP=true"
Environment="PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:$HOME/.initia/cosmovisor/current/bin"

[Install]
WantedBy=multi-user.target
EOF
```

```console
sudo systemctl daemon-reload
sudo systemctl enable initia.service
```

<h1 align="center">Senkronizasyon</h1>


```console

initiad config set client chain-id initiation-1
initiad config set client keyring-backend test
initiad config set client node tcp://localhost:17957

initiad init $MONIKER --chain-id initiation-1

sudo systemctl stop initiad

cp $HOME/.initia/data/priv_validator_state.json $HOME/.initia/priv_validator_state.json.backup

rm -rf $HOME/.initia/data $HOME/.initia/wasmPath
curl -L https://snapshots.kjnodes.com/initia-testnet/snapshot_latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.initia

mv $HOME/.initia/priv_validator_state.json.backup $HOME/.initia/data/priv_validator_state.json


sudo systemctl restart initiad && sudo journalctl -u initiad -f

sed -i -e "s|^seeds *=.*|seeds = \"3f472746f46493309650e5a033076689996c8881@initia-testnet.rpc.kjnodes.com:17959\"|" $HOME/.initia/config/config.toml

sed -i -e "s|^minimum-gas-prices *=.*|minimum-gas-prices = \"0.15uinit,0.01uusdc\"|" $HOME/.initia/config/app.toml

sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "19"|' \
  $HOME/.initia/config/app.toml

sed -i -e "s%^proxy_app = \"tcp://127.0.0.1:26658\"%proxy_app = \"tcp://127.0.0.1:17958\"%; s%^laddr = \"tcp://127.0.0.1:26657\"%laddr = \"tcp://127.0.0.1:17957\"%; s%^pprof_laddr = \"localhost:6060\"%pprof_laddr = \"localhost:17960\"%; s%^laddr = \"tcp://0.0.0.0:26656\"%laddr = \"tcp://0.0.0.0:17956\"%; s%^prometheus_listen_addr = \":26660\"%prometheus_listen_addr = \":17966\"%" $HOME/.initia/config/config.toml
sed -i -e "s%^address = \"tcp://0.0.0.0:1317\"%address = \"tcp://0.0.0.0:17917\"%; s%^address = \":8080\"%address = \":17980\"%; s%^address = \"0.0.0.0:9090\"%address = \"0.0.0.0:17990\"%; s%^address = \"0.0.0.0:9091\"%address = \"0.0.0.0:17991\"%; s%:8545%:17945%; s%:8546%:17946%; s%:6065%:17965%" $HOME/.initia/config/app.toml

curl -L https://snapshots.kjnodes.com/initia-testnet/snapshot_latest.tar.lz4 | tar -Ilz4 -xf - -C $HOME/.initia
[[ -f $HOME/.initia/data/upgrade-info.json ]] && cp $HOME/.initia/data/upgrade-info.json $HOME/.initia/cosmovisor/genesis/upgrade-info.json

sudo systemctl start initia.service && sudo journalctl -u initia.service -f --no-hostname -o cat
```

<h1 align="center">Cüzdan, Seed Kaydedin.</h1>


```console
# Yeni cüzdan oluşturma
initiad keys add wallet
```

```console
# Recover
initiad keys add wallet --recover
```

> https://faucet.testnet.initia.xyz/

<h1 align="center">Validatör Kur</h1>

```console
initiad config set client keyring-backend test


initiad tx mstaking create-validator \
--amount 1000000uinit \
--pubkey $(initiad tendermint show-validator) \
--moniker "Validatör İsmi" \
--identity "boş geçebilirsiniz" \
--details "kafanıza göre" \
--website "github linkiniz olabilir" \
--chain-id initiation-1 \
--commission-rate 0.05 \
--commission-max-rate 0.20 \
--commission-max-change-rate 0.05 \
--from wallet \
--gas-adjustment 1.4 \
--gas 1000000 \
--gas-prices 0.15uinit \
--node tcp://localhost:17957
```

> tx hash sonrası validator isminizi kontrol edebilirsiniz

> Bol şans.
