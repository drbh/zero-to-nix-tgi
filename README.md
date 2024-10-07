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
# ✅ 1. Instance ID: i-abcd1234fffffffff
# ✅ 2. Instance is running
# ✅ 3. Private IP: 99.90.99.99
# Warning: Permanently added '99.90.99.99' (ED25519) to the list of known hosts.
# ...
# ✅ 4. Machine is accessible
# insta.nix                                                                  100% 1320    80.3KB/s   00:00
# ✅ 5. Configuration file copied
# unpacking channels...
# ✅ 6. Nix channel set to unstable
# this derivation will be built:
# ... (this takes ~10 minutes)
# ✅ 7. NixOS rebuild switch command run
# ...
# ✅ Machine rebooting
# ssh: connect to host 99.90.99.99 port 22: Connection refused
# ...
# ✅ 10. Machine is back up
# Cachix configuration written to /etc/nixos/cachix.nix.
# ...
# ✅ 11. Cloned the repo
# ...
# ✅ 12. Server files prepared for development
# ...
# ✅ 13. Weights downloaded
# ...
# ✅ 14. Added to ~/.ssh/config
# ...
# ✅ 15. Setup time: 584 seconds.
```

## Shutdown the devbox

Once you're done with the devbox you can shut it down with the following command.

```bash
./tginow down
```

# Notes

helpful commands for the byobu session

top terminal

```bash
cd server
export HF_TOKEN=YOUR_HF_TOKEN
ATTENTION=flashinfer USE_PREFIX_CACHING=True MASTER_ADDR=127.0.0.1 MASTER_PORT=5555 python text_generation_server/cli.py serve meta-llama/Llama-3.1-8B-Instruct
```

middle terminal (shift+down)

```bash
ATTENTION=flashinfer USE_PREFIX_CACHING=True cargo run --bin text-generation-router --release -- --tokenizer-name meta-llama/Llama-3.1-8B-Instruct --max-batch-prefill-tokens 1000 --max-input-tokens 1000 --max-total-tokens 1001
```

bottom terminal (shift+down and once everything is running)

```bash
curl 127.0.0.1:3000/v1/completions \
    -X POST \
    -H 'Content-Type: application/json' \
    -d '{"model": "tgi", "prompt": "What is Deep Learning?", "max_tokens": 20, "temperature": 0.0, "stream": true}'
```
