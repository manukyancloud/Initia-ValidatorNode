 ## Dosyaları Çekiyoruz
```
cd $HOME
rm -rf slinky
git clone https://github.com/skip-mev/slinky.git
cd slinky
git checkout v0.4.3
```
```
make build
```

```
mv build/slinky /usr/local/bin/
```

## slinkyd.service
```
sudo tee /etc/systemd/system/slinkyd.service > /dev/null <<EOF
[Unit]
Description=Initia Slinky Oracle
After=network-online.target

[Service]
User=$USER
ExecStart=$(which slinky) --oracle-config-path $HOME/slinky/config/core/oracle.json --market-map-endpoint 127.0.0.1:15090
Restart=on-failure
RestartSec=30
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF
```
## Oluşturduğumuz Servisi Başlatıyoruz
```
sudo systemctl daemon-reload
sudo systemctl enable slinkyd.service
sudo systemctl restart slinkyd.service
```
## app.toml'da Oracle Düzenliyoruz, Görseldeki değerleri uygun yerlere girelim, sayfanın en altında.
```
nano /root/.initia/config/app.toml
```
<img width="678" alt="Ekran Resmi 2024-05-19 16 12 51" src="https://github.com/manukyancloud/Initia-ValidatorNode/assets/84817402/1c9792a7-249e-49e5-9d68-311651214cee">
# Ctrl+x y enter ile çıkış yapıyoruz.

## initiad ve slinkyd servislerini resetleyip logları kontrol edelim.
```
sudo systemctl daemon-reload
sudo systemctl restart initiad
sudo systemctl restart slinkyd.service
sudo journalctl -u initiad.service -f --no-hostname -o cat
```
