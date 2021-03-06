# Truffle Migrate
This project is an example that shows how to create a new project from scratch 
that depends on DutchX and how you can deploy the DutchX contracts in a 
`ganache-cli` local node.

It shows how to deploy easily the contracts in a local ganache-cli,
so we don't depend on testnets like Rinkeby and we can speed up the development.

Also, using truffle brings many other advantages like test, migrations, 
easier interaction, sharing compiled contracts into NPM packages, etc...

> For more info on truffle see:
> * https://truffleframework.com

> This example is part of the guide http://dutchx.readthedocs.io/en/latest/
>
> You can find other examples in the [Build on top of DutchX project](https://github.com/gnosis/dx-examples-dev)

# Have you done the first example?
Maybe you should consider doing the 
[basic web example first](https://github.com/gnosis/dx-examples-dev/tree/master/01_basic-web-deposit).

That example doesn't use truffle, and it shows the basics on how to integrate
with DutchX.

## Create a basic truffle contract
Create your `my-cool-app` project:
```bash
mkdir my-cool-app
cd my-cool-app
npm init
```

Make it a truffle project:
```bash
mkdir contracts migrations
npm install truffle@4.1.5 --save-dev
npm install truffle-contract@^3.0.6 truffle-hdwallet-provider@^0.0.5 --save
```

> We set up a fixed version for truffle, so the compiler is the same that the
> dx used.
>
> Truffle 5.0 is still a beta, this version will allow to set any compiler 
> version soon we'll migrate the scripts to this version (it has breaking 
> changes)

Add a truffle config:
* Add [truffle.js](./truffle.js) into the root of the project. It has a pretty
  standard boilerplate config with:
    * **Networks**: Define networks: ganache-cli (localhost), Rinkeby, Kovan and Mainnet.
    * **Mnemonic**: Configured with a default test mnemonic. It allows to change
      it by providing the `MNEMONIC` environment variable.
        * Default Mnemonic: `candy maple cake sugar pudding cream honey rich smooth crumble sweet treat`
        * Default address: `0x627306090abab3a6e1400e9345bc60c78a8bef57`
        * `IMPORTANT`: Don't use this mnemonic for mainnet with real funds. This
          is a public test mnemonic used in many projects.
    * **Gas price**: Configured with a default gas price. It allows
      to change it by providing the `GAS_PRICE_GWEI` environment variable.
        * Default Gas Price: `5 Gwei`
    * It prints useful information

Add the first migration and it's contract:
* [contracts/Migrations.sol](./contracts/Migrations.sol) into `contracts` dir
* [migrations/1_initial_migration.js](./migrations/1_initial_migration.js) into `migrations` dir

## Get DutchX dependencies and compiling it

Add the required dependencies
```bash
npm install @gnosis.pm/dx-contracts --save
```

To ensure that for local development, truffle finds all DutchX contract, we can
create a dummy contract call `CoolAppDependencies.sol`.

This contract won't do anything, it won't event have code, the important thing
is that it must import `@gnosis.pm/dx-contracts/contracts/DxDevDependencies.sol`
so truffle will pull and compile all the required contracts.

Copy the code from:
* [contracts/CoolAppDependencies.sol](./contracts/CoolAppDependencies.sol).

> For more information, check:
>   * https://github.com/gnosis/dx-contracts/blob/master/contracts/DxDevDependencies.sol

Now, if we compile the contracts, we'll see how the `DutchExchange` is being 
compiled among all the other contracts it depends on:

```bash
npx truffle compile
````

Now we should see how all DutchX contracts are being compiled:

![Compiling contracts](./docs/compiling-contracts.png "Compiling contracts")

## Add a new migration for the DutchX

For the ease of use, following criteria should be met for the migration:
* Only executes in the `development` network. Note that `rinkeby` and `mainet` 
contracts are already deployed.
* Uses the contracts that are in our NPM dependencies, so we avoid repetition.
* Uses the migration scripts that are in our NPM dependencies. **DutchX** NPM 
module provides a simple migration source that can be referenced to ease the 
local development.

Hence, we just create our second migration file:
* [migrations/2_DEV_migrate_dependencies.js](./migrations/2_DEV_migrate_dependencies.js)

> For more information, check:
>   * https://github.com/gnosis/dx-contracts/blob/master/src/migrations/index.js

## Apply the migrations
```bash
# Run a ganache cli
npx ganache-cli -d

# Run migrations
#   It's also a good idea to add it a a package.json script
npx truffle migrate
```

Truffle will migrate automatically all contracts:

![Migrating contracts](./docs/migrating-contracts.png "Migrating contracts")

## Truffle console
If we execute `npx truffle networks` at this point, we can verify the addresses
of the contracts deployed to the local ganache-cli.

![Deployed contracts](./docs/local-ganache-deployed-contracts.png "Deployed contracts")

If we run `truffle console`, we'll be able to play arround with the deployed 
DutchX smart contracts.

```bash
truffle console

truffle(development)> dutchx = DutchExchange.at(DutchExchangeProxy.address)
truffle(development)> dutchx.auctioneer.call()
truffle(development)> dutchx.thresholdNewAuction.call().then(n => n.toNumber())
```

![Truffle console](./docs/truffle-console.png "Truffle console")

## Next steps
Congratulations! you now have a truffle project that deploys all the DutchX contracts as 
part of it's first migration.

Now you can continue with [Example 03: Onchain integration - Use it as an oracle](https://github.com/gnosis/dx-examples-dev/tree/master/03_onchain-integration-oracle): 
* This example shows how to create your own contract and migrations that makes 
use of the DutchX.
* In this example we will create a `Safe` contract to deposit fund. We will
use the DutchX to help us estimate the price in `USD` for any token in our 
`Safe`.
