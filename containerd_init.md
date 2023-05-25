# install runwasi for docker/containerd

curl -sSf https://raw.githubusercontent.com/WasmEdge/WasmEdge/master/utils/install.sh | bash -s -- -e all -v 0.11.2
sudo -E sh -c 'echo "$HOME/.wasmedge/lib" > /etc/ld.so.conf.d/libwasmedge.conf'
sudo ldconfig

## Rust 
git clone https://github.com/containerd/runwasi.git
cd runwasi/
cargo test -- --nocapture
make build

# install containerd by gz file
export VERSION="1.6.12"
echo -e "Version: $VERSION"
echo -e "Installing libseccomp2 ..."
sudo apt install -y libseccomp2
echo -e "Installing wget"
sudo apt install -y wget

wget https://github.com/containerd/containerd/releases/download/v${VERSION}/cri-containerd-cni-${VERSION}-linux-amd64.tar.gz
wget https://github.com/containerd/containerd/releases/download/v${VERSION}/cri-containerd-cni-${VERSION}-linux-amd64.tar.gz.sha256sum
sha256sum --check cri-containerd-cni-${VERSION}-linux-amd64.tar.gz.sha256sum

sudo tar --no-overwrite-dir -C / -xzf cri-containerd-cni-${VERSION}-linux-amd64.tar.gz
sudo systemctl daemon-reload
sudo systemctl start containerd


# install containerd apt-get

apt-get install -y containerd

containerd config default | tee /etc/containerd/config.toml

  [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
    SystemdCgroup = true
....
    sandbox_image = "registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.6"


ctr i pull docker.io/wasmedge/example-wasi:latest

ctr run --rm --runc-binary crun --runtime io.containerd.runc.v2 --label module.wasm.image/variant=compat-smart docker.io/wasmedge/example-wasi:latest wasm-example /wasi_example_main.wasm 50000000 Hello WasmEdge

ctr i pull docker.io/wasmedge/example-wasi-http:latest
ctr run -rm -d --net-host --runc-binary crun --runtime io.containerd.runc.v2 --label module.wasm.image/variant=compat-smart docker.io/wasmedge/example-wasi-http:latest xxxxx

curl -d "name=WasmEdge" -X POST http://127.0.0.1:1234

apt-get remove -y containerd
rm -rf /etc/containerd
rm -rf /etc/containers
rm -rf /sys/devices/system/container
rm -rf /sys/bus/container
rm -rf /var/cache/containers
rm -rf /var/lib/containerd


## buildkit  not validate

----------------------------------not validate--------------------------------------------------------------
wget https://github.com/moby/buildkit/releases/download/v0.11.6/buildkit-v0.11.6.linux-amd64.tar.gz
buildkitd --oci-worker=false --containerd-worker=true & 
-----------------------------------------------------------------------------------------------
mkdir -p /etc/buildkit/
vim /etc/buildkit/buildkitd.toml

[worker.oci]
  enabled = false

[worker.containerd]
  enabled = true
  # namespace should be "k8s.io" for Kubernetes (including Rancher Desktop)
  namespace = "default"
  platforms = [ "linux/amd64" ]
  gc = true
  # gckeepstorage sets storage limit for default gc profile, in MB.
  gckeepstorage = 9000


buildkitd --config /etc/buildkit/buildkitd.toml & 

# install docker by apt-get
rm -rf /var/lib/docker
rm -rf /var/lib/containerd

sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/ \
  $(lsb_release -cs) \
  stable"
sudo apt-get update

apt-cache madison docker-ce

sudo apt-get install docker-ce docker-ce-cli containerd.io

containerd config default > /etc/containerd/config.toml

docker pull docker.io/wasmedge/example-wasi:latest
docker pull docker.io/wasmedge/example-wasi-http:latest

docker run --runtime io.containerd.wasmedge.v1 wasmedge/example-wasi /wasi_example_main.wasm 50000000 Hello WasmEdge

docker run --runtime io.containerd.wasmedge.v1 docker.io/wasmedge/example-wasi /wasi_example_main.wasm 50000000 Hello WasmEdge

docker run -p 1234:1234 --runtime io.containerd.wasmedge.v1 wasmedge/example-wasi-http

curl -d "name=WasmEdge" -X POST http://127.0.0.1:1234



-------------------------------------------------------------------------------
sudo apt-get install ./docker-desktop-<version>-<arch>.deb