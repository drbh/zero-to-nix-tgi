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
