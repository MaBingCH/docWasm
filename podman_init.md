sudo apt-get -y update

sudo apt-get -y install buildah
sudo apt-get -y install podman

sudo apt install -y make git gcc build-essential pkgconf libtool \
    libsystemd-dev libprotobuf-c-dev libcap-dev libseccomp-dev libyajl-dev \
    go-md2man libtool autoconf python3 automake

git clone https://github.com/containers/crun
cd crun
./autogen.sh
./configure --with-wasmedge
make
sudo make install

crun --version
   +WASM:wasmedge
   


/etc/containers
/usr/share/containers
$HOME/.config/containers/


apt-get remove podman
rm -rf /usr/share/containers
rm -rf /var/lib/containers
rm -rf /home/vagrant/.local/share/containers
rm -rf /usr/share/containers
rm -rf /etc/containers

# 容器存储
/var/lib/containers/storage

#  
/etc/docker/daemon.json
