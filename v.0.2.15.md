<h1 align="center">GÜNCELLEME</h1>

```
systemctl stop initiad
cd $HOME
rm -rf initia
git clone https://github.com/initia-labs/initia.git
cd initia
git checkout v0.2.15
make build
./build/initiad version
```

```
mv /root/initia/build/initiad $HOME/.initia/cosmovisor/genesis/bin/
```
```
sudo systemctl daemon-reload
sudo systemctl restart initiad
sudo journalctl -u initiad.service -f --no-hostname -o cat
```
