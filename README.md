<p style="font-size:14px" align="right">
<a href="https://t.me/PemulungAirdropID" target="_blank">Join our telegram <img src="https://user-images.githubusercontent.com/72949170/194228482-0f875615-e155-4b12-8716-8111addd6cba.jpg" width="30"/></a>
</p>

<p align="center">
  <img height="auto" height="auto" src="https://user-images.githubusercontent.com/72949170/199922955-c59ee2a5-0425-43b9-942d-15272df61902.png">
</p>


# OKP4 TESTNET TUTORIAL

## Perangkat Minimal

|  Komponen |  Persyaratan Minimum |
| ------------ | ------------ |
| CPU  | Intel 4 Core  |
| RAM | 8 GB  |
| Penyimpanan  | 200 SSD |
| OS |  Ubuntu 18.04 atau lebih tinggi | 

## 1. Instal Otomatis
```
wget -qO okp4.sh https://raw.githubusercontent.com/muhamad-ramadhani/okp4/main/okp4.sh && chmod +x okp4.sh && ./okp4.sh
```
## 2. Load Sistem
```
source $HOME/.bash_profile
```
## 3. Create Wallet

```
okp4d keys add $WALLET
```
Simpan Address & Mnemonic Anda dan Import ke Keplr Wallet

*Note:* Enter Keyring Passpharse = Kalian Isi Password Bebas dan INGETIN

```
okp4d keys add $WALLET --recover
```
(Opsional) Gunakan Peritah di Bawah Jika Anda Memiliki Wallet Lama Bisa menggunakan Perintah di atas

## 4. Claim Faucet OKP4

- Open Website : https://faucet.okp4.network/
- Connect Wallet Keplr Yang Sudah Kalian Import dan Simpan tadi atau Paste Saja Addressnya Untuk Mengambil Tokennya
- Done

## 5. Masukan Variable 2 (Untuk Simpan Informasi Wallet)
```
OKP4_WALLET_ADDRESS=$(okp4d keys show $WALLET -a)
```
Masukan Katasandi Pharse
```
OKP4_VALOPER_ADDRESS=$(okp4d keys show $WALLET --bech val -a)
```
Masukan Kata Sandi Pharse
```
echo 'export OKP4_WALLET_ADDRESS='${OKP4_WALLET_ADDRESS} >> $HOME/.bash_profile
echo 'export OKP4_VALOPER_ADDRESS='${OKP4_VALOPER_ADDRESS} >> $HOME/.bash_profile
source $HOME/.bash_profile
```
## 6. Check Sinkron Block

```
journalctl -fu okp4d -o cat
```

*Note :* Pastikan Block di VPS Sampai Dengan Block Height Saat ini, Yang Ada di Blockchain nya, Silahkan Check di : https://okp4.explorers.guru/ : 

### Pakai Snapshoot dari Nodejumper Jika Ingin Loncat Ke Block Saat Ini

```
sudo systemctl stop okp4d
cp $HOME/.okp4d/data/priv_validator_state.json $HOME/.okp4d/priv_validator_state.json.backup
okp4d tendermint unsafe-reset-all --home $HOME/.okp4d --keep-addr-book
rm -rf $HOME/.okp4d/data 
rm -rf $HOME/.okp4d/wasm
SNAP_NAME=$(curl -s https://snapshots2-testnet.nodejumper.io/okp4-testnet/ | egrep -o ">okp4-nemeton.*\.tar.lz4" | tr -d ">")
curl https://snapshots2-testnet.nodejumper.io/okp4-testnet/${SNAP_NAME} | lz4 -dc - | tar -xf - -C $HOME/.okp4d
mv $HOME/.okp4d/priv_validator_state.json.backup $HOME/.okp4d/data/priv_validator_state.json
sudo systemctl restart okp4d
sudo journalctl -u okp4d -f --no-hostname -o cat
```
Tahap Ini Lumayan Cukup Lama Biarkan Saja Hinggal Block Height Jalan Sekitar 30 menitan

## 7. Check Balance 

```
okp4d q bank balances $OKP4_WALLET_ADDRESS
```

Jika Balance Anda Sudah Masuk, Anda Bisa Lanjut Untuk Membuat Validator

## 8. Membuat Validator Jalankan Perintah

```
okp4d tx staking create-validator \
--amount=1000000uknow \
--pubkey=$(okp4d tendermint show-validator) \
--moniker="karinanoob" \
--chain-id=okp4-nemeton \
--commission-rate="0.01" \
--commission-max-rate="0.10" \
--commission-max-change-rate="0.01" \
--min-self-delegation="1000000" \
--fees=1000uknow \
--from=wallet \
```

## 9 Delete Node (Optional Kalo Pengen hapus)

Perintah ini akan sepenuhnya menghapus node dari server. Gunakan dengan risiko Anda sendiri!
```
sudo systemctl stop okp4d
sudo systemctl disable okp4d
sudo rm /etc/systemd/system/okp4* -rf
sudo rm $(which okp4d) -rf
sudo rm $HOME/.okp4d* -rf
sudo rm $HOME/okp4 -rf
sed -i '/OKP4_/d' ~/.bash_profile
```
## 10. Beberapa Perintah Berguna (Optional)

### You can check the node logs with the command:
```
journalctl -u okp4d -f
```
### Restart node:
```
systemctl restart okp4d
```
### Check your node status:
```
curl localhost:26657/status
```
### Check node synchronization, if results false – node is synchronized
```
curl -s localhost:26657/status | jq .result.sync_info.catching_up
```
### Get your valoper address:
```
okp4d keys show wallet --bech val -a
```
### Bond more tokens (if you want increase your validator stake you should bond more to your valoper address):
```
okp4d tx staking delegate YOUR_VALOPER_ADDRESS 10000000uknow --from wallet --chain-id okp4-nemeton --fees 5000uknow
```
### Active validators list:
```
okp4d query staking validators --limit 2000 -o json | jq -r '.validators[] | select(.status=="BOND_STATUS_BONDED") | [.operator_address, .status, (.tokens|tonumber / pow(10; 6)), .description.moniker] | @csv' | column -t -s"," | sort -k3 -n -r
```
### Inactive validators list:
```
okp4d query staking validators --limit 2000 -o json | jq -r '.validators[] | select(.status=="BOND_STATUS_UNBONDED") | [.operator_address, .status, (.tokens|tonumber / pow(10; 6)), .description.moniker] | @csv' | column -t -s"," | sort -k3 -n -r
```
