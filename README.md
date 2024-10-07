## Create a devbox

this is a single file bash script that will setup a TGI dev box on AWS. It will start a multi-gpu instance, all of the cuda/gpu drivers, setup a basic dev environment, connect to the nix cache, clone the repo and build the flake for the first time.

# Quickstart

download, make executable, and run the script (make sure you read the source code first!)

```bash
curl \
-fsSL https://raw.githubusercontent.com/drbh/zero-to-nix-tgi/refs/heads/main/tginow \
> tginow \
&& chmod +x tginow \
&& ./tginow help
# Usage: ./tginow <command> [options]
# Commands:
#   status          Get the status of the devbox
#   up              Create and setup a new devbox
#   setup           Setup the development environment
#   add-to-local    Add the devbox to local SSH config
#   open            Open a byobu session with three panes
#   ssh             SSH into the devbox
#   down            Shutdown the devbox
#   help            Display this help message
```

🙌 yay, now you have `tginow` on your local machine.

next make sure to setup your `config.ini` file with the following information:

```ini
# EDIT THESE VARIABLES TO CHOSE YOUR ADVENTURE
TAG_NAME="david-holtz-quick-nixos"
KEY_NAME="..." 
MYTOKEN="YOUR_HF_TOKEN"

# Just for the initial setup
MODEL_NAME="meta-llama/Llama-3.1-8B-Instruct"

# Machine related variables
INSTANCE_TYPE="g6.12xlarge"

# Machine base variables
VOLUME_SIZE=512
IMAGE_ID="ami-0b3d51a362efa9e02"

# HF related variables
REGION="us-east-1"
SUBNET_ID="subnet-..."
SECURITY_GROUP_ID="sg-..."
```

# Start the devbox

this takes about 10 minutes but will setup the machine and environment needed to run TGI along with all of the source code.

```bash
./tginow up
# 👍 Creating and setting up a new devbox...
# ✅ NixOS configuration generated at /tmp/insta.nix
# ✅ Instance ID: i-abcd1234fffffffff
# ✅ Instance is running
# ✅ Private IP: 99.90.99.99
# Warning: Permanently added '99.90.99.99' (ED25519) to the list of known hosts.
#  02:28:08  up   0:00,  1 user,  load average: 0.65, 0.20, 0.07
# Connection to 99.90.99.99 closed.
# ✅ Machine is accessible
# insta.nix                                                                  100% 1320    80.3KB/s   00:00
# ✅ Configuration file copied
# unpacking channels...
# Connection to 99.90.99.99 closed.
# ✅ Nix channel set to unstable
# this derivation will be built:
# ... (this takes ~10 minutes)
# ✅ NixOS rebuild switch command run
# Connection to 10.90.1.192 closed.
# ✅ Machine rebooting
# ssh: connect to host 10.90.1.192 port 22: Connection refused
# ...
# ✅ Machine is back up
# Cachix configuration written to /etc/nixos/cachix.nix.
# ...
# ✅ Cachix use text-generation-inference
# Connection to 10.90.1.192 closed.
# ✅ Uncommented the 5th line of the configuration.nix
# building Nix...
# ...
# Connection to 10.90.1.192 closed.
# ✅ NixOS rebuild switch command run
# Cloning into 'text-generation-inference'...
# ...
# ✅ Cloned the repo
# assets	    clients		Dockerfile_intel   flake.nix	      Makefile	 rust-toolchain.toml
# backends    CODE_OF_CONDUCT.md	Dockerfile.nix	   integration-tests  nix	 sagemaker-entrypoint.sh
# benchmark   CONTRIBUTING.md	Dockerfile.trtllm  launcher	      proto	 server
# Cargo.lock  Dockerfile		docs		   LICENSE	      README.md  tgi-entrypoint.sh
# Cargo.toml  Dockerfile_amd	flake.lock	   load_tests	      router	 update_doc.py
# Connection to 10.90.1.192 closed.
# ✅ Setup time: 584 seconds.
```

# Prepare the devbox

This will download the model and run a couple commands to setup vscode server so you can connect to the devbox from your local machine.

```bash
./tginow setup
```

# Add to local ssh

Before we can connect we'll have to add the devbox to our local ssh config. Then we should be able to see it in vscode.

```bash
./tginow add-to-local-ssh
```

## Shutdown the devbox

Once you're done with the devbox you can shut it down with the following command.

```bash
bash down.sh
```

# Notes

helpful commands for the byobu session

```bash
# top terminal
cd server
export HF_TOKEN=YOUR_HF_TOKEN
ATTENTION=flashinfer USE_PREFIX_CACHING=True MASTER_ADDR=127.0.0.1 MASTER_PORT=5555 python text_generation_server/cli.py serve meta-llama/Llama-3.1-8B-Instruct

# middle terminal (shift+down)
ATTENTION=flashinfer USE_PREFIX_CACHING=True cargo run --bin text-generation-router --release -- --tokenizer-name meta-llama/Llama-3.1-8B-Instruct --max-batch-prefill-tokens 1000 --max-input-tokens 1000 --max-total-tokens 1001

# bottom terminal (shift+down and once everything is running)
curl 127.0.0.1:3000/v1/completions \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"model": "tgi", "prompt": "What is Deep Learning?", "max_tokens": 20, "temperature": 0.0, "stream": true}'
```
