<script lang="ts">
  export let title: string = '';
  import {chainName} from '../config';
  import NavButton from '../components/navigation/NavButton.svelte';
  import Toast from '../components/notification/Toast.svelte';
  import Modal from '../components/Modal.svelte';

  import {
    wallet,
    builtin,
    chain,
    transactions,
    balance,
    flow,
    fallback,
  } from '../stores/wallet';

  const base: string = window.basepath || '/';

  $: executionError = $flow.executionError as any;

  let options: {img: string; id: string; name: string}[] = [];
  $: builtinNeedInstalation =
    $wallet.options.filter((v) => v === 'builtin' && !$builtin.available)
      .length > 0;
  $: options = $wallet.options
    .filter((v) => v !== 'builtin' || $builtin.available)
    .map((v) => {
      return {
        img: ((v) => {
          if (v === 'builtin') {
            if ($builtin.state === 'Ready') {
              if ($builtin.vendor === 'Metamask') {
                return 'images/metamask.svg';
              } else if ($builtin.vendor === 'Opera') {
                return 'images/opera.svg';
              }
            }
            return 'images/web3-default.png';
          } else {
            if (v.startsWith('torus-')) {
              const verifier = v.slice(6);
              return `images/torus/${verifier}.svg`;
            }
            return `images/${v}.svg`;
          }
        })(v),
        id: v,
        name: v,
      };
    });

  function connect(e: Event) {
    flow.connect();
    e.stopImmediatePropagation();
  }

  function disconnect(e: Event) {
    wallet.disconnect();
    e.stopImmediatePropagation();
  }

  function formatError(message: string): string {
    const messageInside = message.indexOf('"message":');
    if (messageInside >= 0) {
      const subMessage = message.substr(messageInside + 11);
      const nextQuote = message.indexOf('"');
      return subMessage.substr(0, nextQuote);
    } else {
      return message;
    }
  }
</script>

<slot />

{#if $chain.state === 'Idle' && !$chain.connecting && $fallback.error}
  <div
    class="w-full flex items-center justify-center fixed bottom-0"
    style="z-index: 5;">
    <p
      class="w-64 text-center rounded-tl-xl rounded-tr-xl text-gray-200 bg-pink-600 p-1">
      Network Issues, Please
      <button class="underline" on:click={connect}>Connect</button>.
    </p>
  </div>
{:else if $chain.state === 'Idle' && !$chain.connecting && $fallback.state === 'Idle' && !$fallback.connecting}
  <div
    class="w-full flex items-center justify-center fixed bottom-0"
    style="z-index: 5;">
    <p
      class="w-64 text-center rounded-tl-xl rounded-tr-xl text-gray-200 bg-pink-600 p-1">
      Please
      <button class="underline" on:click={connect}>Connect</button>.
    </p>
  </div>
{:else if $chain.notSupported}
  <div
    class="w-full flex items-center justify-center fixed bottom-0"
    style="z-index: 5;">
    <p
      class="w-64 text-center rounded-tl-xl rounded-tr-xl text-gray-200 bg-pink-600 p-1">
      Wrong network, use
      {chainName}
      or
      <button class="underline" on:click={disconnect}>Disconnect</button>
    </p>
  </div>
{/if}

{#if $flow.inProgress}
  <Modal
    {title}
    cancelable={!$wallet.connecting}
    on:close={() => flow.cancel()}
    closeButton={false}>
    {#if $wallet.state === 'Idle'}
      {#if $wallet.loadingModule}
        Loading module:
        {$wallet.selected}
      {:else if $wallet.connecting}
        Connecting to wallet...
      {:else}
        <div class="text-center">
          <p>You need to connect your wallet.</p>
        </div>
        <div class="flex flex-wrap justify-center pb-3">
          {#each options as option}
            <img
              class="cursor-pointer p-2 m-2 border-2 h-12 w-12 object-contain"
              alt={`Login with ${option.name}`}
              src={`${base}${option.img}`}
              on:click={() => wallet.connect(option.id)} />
          {/each}
        </div>
        {#if builtinNeedInstalation}
          <div class="text-center">OR</div>
          <div class="flex justify-center">
            <NavButton
              label="Download Metamask"
              blank={true}
              href="https://metamask.io/download.html"
              class="m-4 w-max-content">
              <img
                class="cursor-pointer p-0 mx-2 h-10 w-10 object-contain"
                alt={`Download Metamask}`}
                src={`${base}images/metamask.svg`} />
              Download metamask
            </NavButton>
          </div>
        {/if}
      {/if}
    {:else if $wallet.state === 'Locked'}
      {#if $wallet.unlocking}
        Please accept the application to access your wallet.
      {:else}
        <NavButton label="Unlock Wallet" on:click={() => wallet.unlock()}>
          Unlock
        </NavButton>
      {/if}
    {:else if $chain.state === 'Idle'}
      {#if $chain.connecting}Connecting...{/if}
    {:else if $chain.state === 'Connected'}
      {#if $chain.loadingData}
        Loading contracts...
      {:else if $chain.notSupported}Please switch to {chainName}{/if}
    {:else if $wallet.pendingUserConfirmation}
      Please accept transaction...
    {:else if executionError}
      {#if executionError.code === 4001}
        You rejected the request
      {:else if executionError.message}
        {formatError(executionError.message)}
      {:else}Error: {executionError}{/if}
      <NavButton label="Retry" on:click={() => flow.retry()}>Retry</NavButton>
    {/if}
  </Modal>
{/if}
