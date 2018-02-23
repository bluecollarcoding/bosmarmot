# Bosmarmot

Bosmarmot is a monorepo containing condensed and updated versions of the monax tooling. This repo intends to provide the basic tooling required to interact with a [Burrow](https://github.com/hyperledger/burrow) chain.

It also contains the interpreter for the Monax packages specification language 'epm'. This README will cover setting up a Burrow chain with the Monax tooling from start to finish.

## Install

We're going to need four (4) binaries:

First, ensure you have `go` installed and `$GOPATH` set

```
burrow
burrow-client (maybe?)
bos
monax-keys
solc
```

For `burrow` and `burrow-client`:

```
go get github.com/hyperledger/burrow
cd $GOPATH/src/github.com/hyperledger/burrow
make build
```

For `bos` and `monax-keys`:

```
go get github.com/monax/bosmarmot
cd $GOPATH/src/github.com/monax/bosmarmot
make build
```

To install the solidity compiler - `solc` - see [here](https://solidity.readthedocs.io/en/develop/installing-solidity.html) for platform specific instructions.

## Configre

The end result will be a `burrow.toml` which contains the genesis spec and burrow configuration options required when starting the `burrow` node.

### Accounts

First, let's create some accounts. In this case, we're creating one of each a Participant and Full account:

```
burrow spec --participant-accounts=1 --full-accounts=1 > accounts.json
```

and writing the output to an `accounts.json`. This file should look like:

```
{
	Accounts": [
		{
			"Amount": 99999999999999,
			"AmountBonded": 9999999999,
			"Name": "Full_0",
			"Permissions": [
				"all"
			]
		},
		{
			"Amount": 9999999999,
			"Name": "Participant_0",
			"Permissions": [
				"send",
				"call",
				"name",
				"hasRole"
			]
		}
	]
}

```

Because the next command will be making keys, let's open a new terminal window and start the keys server:

```
monax-keys server
```

Then, we pass the `accounts.json` in the following command:

```
burrow configure --genesis-spec=accounts.json --validator-index=0 > burrow.toml
```

which creates `burrow.toml` in that looks like:

```
ValidatorAddress = "0A40DC874BC932B78AC390EAD1C1BF33469597AB"

[GenesisDoc]
  GenesisTime = 2018-02-20T14:13:41Z
  ChainName = "BurrowChain_2A0FC2"
  [GenesisDoc.GlobalPermissions]
    Roles = []
    [GenesisDoc.GlobalPermissions.Base]
      Perms = 2302
      SetBit = 16383

  [[GenesisDoc.Accounts]]
    Address = "0A40DC874BC932B78AC390EAD1C1BF33469597AB"
    PublicKey = "{\"type\":\"ed25519\",\"data\":\"CF6D9A53B8BD4F08BD31C1F5643FA1688B9145DB1A644BD9E3A8AC3801FB69DB\"}"
    Amount = 99999999999999
    Name = "Full_0"
    [GenesisDoc.Accounts.Permissions]
      [GenesisDoc.Accounts.Permissions.Base]
        Perms = 16383
        SetBit = 16383

  [[GenesisDoc.Accounts]]
    Address = "179955FD376C71A793886CCB0FDBE011CC6537E0"
    PublicKey = "{\"type\":\"ed25519\",\"data\":\"09781EA8C79C238A1D91992D29E330F7011ADF1B16118AFD0E17D2F3004A2F40\"}"
    Amount = 9999999999
    Name = "Participant_0"
    [GenesisDoc.Accounts.Permissions]
      [GenesisDoc.Accounts.Permissions.Base]
        Perms = 2118
        SetBit = 2118

  [[GenesisDoc.Validators]]
    Address = "0A40DC874BC932B78AC390EAD1C1BF33469597AB"
    PublicKey = "{\"type\":\"ed25519\",\"data\":\"CF6D9A53B8BD4F08BD31C1F5643FA1688B9145DB1A644BD9E3A8AC3801FB69DB\"}"
    Amount = 9999999999
    Name = "Full_0"

    [[GenesisDoc.Validators.UnbondTo]]
      Address = "0A40DC874BC932B78AC390EAD1C1BF33469597AB"
      PublicKey = "{\"type\":\"ed25519\",\"data\":\"CF6D9A53B8BD4F08BD31C1F5643FA1688B9145DB1A644BD9E3A8AC3801FB69DB\"}"
      Amount = 9999999999

[Tendermint]
  Seeds = ""
  ListenAddress = "tcp://0.0.0.0:46656"
  Moniker = ""
  TendermintRoot = ".burrow"

[Keys]
  URL = "http://localhost:4767"

[RPC]
  [RPC.V0]
    [RPC.V0.Server]
      ChainId = ""
      [RPC.V0.Server.bind]
        address = ""
        port = 1337
      [RPC.V0.Server.TLS]
        tls = false
        cert_path = ""
        key_path = ""
      [RPC.V0.Server.CORS]
        enable = false
        allow_credentials = false
        max_age = 0
      [RPC.V0.Server.HTTP]
        json_rpc_endpoint = "/rpc"
      [RPC.V0.Server.web_socket]
        websocket_endpoint = "/socketrpc"
        max_websocket_sessions = 50
        read_buffer_size = 4096
        write_buffer_size = 4096
  [RPC.TM]
    ListenAddress = ":46657"

[Logging]
  [Logging.RootSink]
    [Logging.RootSink.Output]
      OutputType = "stderr"
      Format = ""
```

## Keys

Recall that we ran `monax-keys server` to start the keys server. The previous command (`burrow configure --genesis-spec`) created two keys. Let's look at them:

```
ls $HOME/.monax/keys/data
```

will show you the existing keys that the `monax-keys server` can use to sign transactions.


## Run Burrow

In another (third) window, we run `burrow`:

```
burrow
```

and the logs will stream in. See further below for more information about configuring the logging.

## Deploy Contracts

Now that the burrow node is running, we can deploy contracts.

For this step, we need two things: one or more solidity contracts, and an `epm.yaml`.
