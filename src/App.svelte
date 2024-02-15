<script>
  import { BeaconWallet } from "@taquito/beacon-wallet";
  import { NetworkType } from "@airgap/beacon-types";
  import { TezosToolkit } from "@taquito/taquito";

  const rpcUrl = "https://ghostnet.ecadinfra.com";
  const Tezos = new TezosToolkit(rpcUrl);

  let wallet;

  const counterContractAddress = "KT1R2LTg3mQoLvHtUjo2xSi7RMBUJ1sJkDiD";

  const increment = async () => {

  Tezos.setWalletProvider(wallet);
  const contract = await Tezos.wallet.at(counterContractAddress);

  const transactionParams = await contract.methods
    .increment(5)
    .toTransferParams();
  const estimate = await Tezos.estimate.transfer(transactionParams);

  const operation = await Tezos.wallet
    .transfer({
      ...transactionParams,
      ...estimate,
    })
    .send();

  console.log(`Waiting for ${operation.opHash} to be confirmed...`);

  await operation.confirmation(2);

  console.log(
    `Operation injected: https://ghost.tzstats.com/${operation.opHash}`
  );

};

  const connectWallet = async () => {
    const newWallet = new BeaconWallet({
      name: "Simple dApp",
      network: {
        type: NetworkType.GHOSTNET,
      },
    });
    await newWallet.requestPermissions();
    wallet = newWallet;
  };

  const disconnect = async () => {
    Tezos.setWalletProvider(undefined);
    wallet.client.clearActiveAccount();
    wallet = undefined;
  }
</script>

<main>
  <h3>Hi.</h3>
  {#if wallet}
  <button on:click={increment}>Increment</button>
  <button on:click={disconnect}>Disconnect</button>
  {:else}
    <button on:click={connectWallet}>Connect</button>
  {/if}
</main>

<style>
</style>
