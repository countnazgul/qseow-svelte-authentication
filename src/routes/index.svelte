<script context="module">
  export const ssr = false;

  export const load = async ({ url, params }) => {
    let qlikTicket = url.searchParams.get("qlikTicket");
    return { props: { qlikTicket } };
  };
</script>

<script>
  import { goto } from "$app/navigation";
  import enigma from "enigma.js";
  import schema from "enigma.js/schemas/12.67.2.json";

  const qsHost = "wss://my-sense-instance.com";
  const reloadURI = `http://localhost:3000`;

  let qlikApps = [];

  export let qlikTicket;

  let qlikTicketString = qlikTicket ? `&QlikTicket=${qlikTicket}` : "";

  const session = enigma.create({
    schema,
    url: `${qsHost}/app/engineData?reloadURI=${encodeURIComponent(
      reloadURI
    )}${qlikTicketString}`,
    createSocket: (url) => new WebSocket(url),
  });

  session.on("notification:OnAuthenticationInformation", (data) => {
    if (data.loginUri) goto(data.loginUri);
  });

  (async function () {
    let global = await session.open();
    qlikApps = await global.getDocList();
    await session.close();
  })();
</script>

<main>
  <h1>Qlik apps:</h1>

  {#each qlikApps as qlikApp}
    <div title={qlikApp.qDocId}>{qlikApp.qDocName}</div>
  {/each}
</main>

<style>
  main {
    padding: 1em;
    margin: 0 auto;
  }

  h1 {
    color: #ff3e00;
    text-transform: uppercase;
    font-size: 4rem;
    font-weight: 100;
    line-height: 1.1;
  }
</style>
