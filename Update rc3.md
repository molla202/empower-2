```
sudo systemctl stop empowerd
cd $HOME
rm -rf empowerchain
git clone https://github.com/EmpowerPlastic/empowerchain
cd empowerchain
git checkout v1.0.0-rc3
cd chain
make build
sudo mv $HOME/empowerchain/chain/build/empowerd $(which empowerd)
sudo systemctl restart empowerd && sudo journalctl -u empowerd -f
```
### Ardından ctrl+c durduralım snap atalım
```
sudo systemctl stop empowerd

cp $HOME/.empowerchain/data/priv_validator_state.json $HOME/.empowerchain/priv_validator_state.json.backup

rm -rf $HOME/.empowerchain/data $HOME/.empowerchain/wasm
curl https://testnet-files.itrocket.net/empower/snap_empower.tar.lz4 | lz4 -dc - | tar -xf - -C $HOME/.empowerchain

mv $HOME/.empowerchain/priv_validator_state.json.backup $HOME/.empowerchain/data/priv_validator_state.json

sudo systemctl restart empowerd && sudo journalctl -u empowerd -f
```
