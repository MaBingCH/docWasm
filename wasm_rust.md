
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
cargo install wasm-pack
rustup target add wasm32-wasi
rustup target add wasm32-unknown-unknown

curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | bash -s -- -e all -v 0.11.2

# ll ~/.wasmedge/lib/libwasmedge.so
cp ~/.wasmedge/lib/libwasmedge.so /usr/local/lib/
sudo -E sh -c 'echo "/usr/local/lib" > /etc/ld.so.conf.d/libwasmedge.conf'
sudo ldconfig

curl -fsSL https://developer.fermyon.com/downloads/install.sh | bash
sudo mv ./spin /usr/local/bin/spin

curl https://wasmtime.dev/install.sh -sSf | bash
