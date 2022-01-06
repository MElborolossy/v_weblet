<svelte:options tag="tf-vdemo" />

<script lang="ts">
import type { HTTPMessageBusClient } from "ts-rmb-http-client";


  const { HTTPMessageBusClient } = window.$libs.ts_rmb_http_client;
  let command: string = "";
  let payload: string = "{}";
  let twinId: number = 0;
  let mnemonics: string = "";
  let secret: string = "";
  let envs: string[] = ["dev", "test", "main"];
  let selectedEnv: string;
  let isReady: boolean = false;
  let gridProxyUrls = {
    dev: "https://gridproxy.dev.grid.tf",
    test: "https://gridproxy.test.grid.tf",
    main: "https://gridproxy.dev.grid.tf",
  };
  let result;
  let display_result;
  let rmb: HTTPMessageBusClient;

  $: {
    isReady = twinId === 0 ? false : true;
    rmb = new HTTPMessageBusClient(twinId, gridProxyUrls[selectedEnv]);
    if (display_result !== result){
      display_result = result
    }
  }
  const onDeploy = async () => {
    if (command.includes("twinserver")){
      command = command.replaceAll("twinserver", "vserver");
      console.log(command)
    }
    else if (!command.startsWith("vserver")){
      command = "vserver." + command;
      console.log(command)
    }
    console.log(command)
    const msg = rmb.prepare(command, [twinId], 0, 2);
    const message = await rmb.send(msg, payload);
    result = await rmb.read(message);
  };
</script>

<div class="twinclient">
  <h1>Twin Client Weblet</h1>
  <div class="configurations">
    <h2>Configurations</h2>
    <div class="configurations__env">
      <p class="label">Network Environment</p>
      <select bind:value={selectedEnv}>
        {#each envs as env}
          <option value={env}>{env}</option>
        {/each}
      </select>
    </div>
    <div class="configurations__twinId">
      <p class="label">Twin Id</p>
      <input
        type="number"
        value={twinId}
        on:change={(e) => {
          twinId = +e.target["value"];
        }}
      />
      <div class="configurations__mnemonics">
        <p class="label">Mnemonics</p>
        <input
          type="password"
          value={mnemonics}
          on:change={(e) => {
            mnemonics = e.target["value"];
          }}
        />
      </div>
      <div class="configurations__kvsecret">
        <p class="label">KVStore Secret</p>
        <input
          type="password"
          value={secret}
          on:change={(e) => {
            secret = e.target["value"];
          }}
        />
      </div>
    </div>
  </div>

  <div class="message">
    <h2>Message</h2>
    <div class="message__command">
      <p class="label">Command</p>
      <input
        type="text"
        value={command}
        on:change={(e) => {
          command = e.target["value"];
        }}
      />
    </div>
    <div class="message__payload">
      <p class="label">Payload</p>
      <textarea
        type="text"
        value={payload}
        on:change={(e) => {
          payload = e.target["value"];
        }}
      />
    </div>
    {#if display_result}
      <div>
        {#if display_result[0].err}
          <p style="color:red">{display_result[0].err}</p>
        {:else}
          <p style="color:green">{display_result[0].dat}</p>
        {/if}
      </div>
    {/if}
    <div>
      <button on:click={onDeploy} disabled={!isReady}> Deploy </button>
      {#if !isReady}
        <p style="color:red">Enter your Twin Id to enable deploy button</p>
      {/if}
    </div>
  </div>
</div>
