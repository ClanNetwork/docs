# Cland Installation and setup

{% hint style="info" %}
For the tutorial, it is assumed that you are using an Ubuntu LTS release.

If you have chosen a different operating system, you will need to modify your commands to suit your operating system.
{% endhint %}

## 1. Install cland

There are 2 options available:

- [Download binary from github](#option-1-download-binary-from-github)
- [Build from source](#option-2-build-from-source)

### Option 1: Download binary from github

1. Download the binary for your platform: [releases](https://github.com/ClanNetwork/clan-network/releases/tag/v1.0.4-alpha).
2. Copy it to a location in your PATH, i.e: `/usr/local/bin` or `$HOME/bin`.

```sh
wget https://github.com/ClanNetwork/clan-network/releases/download/v1.0.4-alpha/clan-network_v1.0.4-alpha_linux_amd64.tar.gz
sudo tar -C /usr/local/bin -zxvf clan-network_v1.0.4-alpha_linux_amd64.tar.gz
```

### Option 2: Build from source

#### Install local package list and toolchain

```sh
# update the local package list and install any available upgrades
sudo apt-get update && sudo apt upgrade -y

# install toolchain
sudo apt install build-essential jq -y
```

#### Install go (1.18+ required):

```sh
# 1. Download the archive

wget https://go.dev/dl/go1.18.linux-amd64.tar.gz

# Optional: remove previous /go files:

sudo rm -rf /usr/local/go

# 2. Unpack:

sudo tar -C /usr/local -xzf go1.18.linux-amd64.tar.gz

# 3. Add the path to the go-binary to your system path:
# (for this to persist, add this line to   your ~/.profile or ~/.bashrc or  ~/.zshrc)

export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export GO111MODULE=on
export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin

# After updating your ~/.profile you will need to source it:
source ~/.profile

# 4. Verify your installation:

go version

# go version go1.18 linux/amd64
```

#### Download source code and build

```sh
# Close and checkout to needed version
git clone https://github.com/ClanNetwork/clan-network
cd clan-network
git fetch origin --tags
git checkout v1.0.4-alpha

# Install
make install
```

## 2. Verify installation

Verify that everything is OK.

```sh
cland version --long
name: Clan-Network
server_name: clan-networkd
version: 1.0.4-alpha
commit: 7a6a92d782c978ac730e337b28d2bc927e809739
build_tags: ""
go: go version go1.18 darwin/amd64
```
