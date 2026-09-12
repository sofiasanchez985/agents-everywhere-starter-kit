# Using sponsor tools

One setup reference for the six sponsors featured in this kit. Choose the tools your workflow needs. **OpenAI** is the marquee sponsor; **CopilotKit and OpenRouter** share the next tier; **Exa, Auth0, and Ambiguous AI** provide additional capabilities. This is the kit's selected lineup; the [global event page](https://aitinkerers.org/hackathons/global/agents-everywhere) maintains the full event roster and links to each city's event.

Use Node.js 22+. For Slack/web/mobile, follow the chosen [template's setup instructions](README.md#templates) and keep credentials in root `.env`. Never put keys in frontend code or a submission. `npm run verify` covers offline behavior, not live account access.

| Sponsor | Used by | First result |
|---|---|---|
| [OpenAI](#openai) | Slack, web, mobile | A model response to supplied context |
| [CopilotKit](#copilotkit) | Slack, web, mobile | A contextual answer and native UI |
| [OpenRouter](#openrouter) | Optional Slack/web/mobile model gateway | A response from your chosen catalog model |
| [Exa](#exa) | Slack template | Research with inspectable sources |
| [Auth0](#auth0) | Optional protected-API recipe | Verified service identity and scope before a protected action |
| [Ambiguous AI](#ambiguous-ai) | Web template; optional Slack integration | A real workplace record that survives refresh |

## OpenAI

**Access and authentication.** Create a server-side [API key](https://platform.openai.com/api-keys) in the OpenAI organization/project you will use for the hackathon.

**Configure Slack/web/mobile** in root `.env`:

```dotenv
MODEL_PROVIDER=openai
OPENAI_API_KEY=your-key
MODEL=gpt-5.6-sol
```

`MODEL` is the kit's configured model; choose one available to your API account.

**First call.** From root, make a small Responses API request using the configured key and model:

```bash
node --env-file=.env --input-type=module <<'JS'
const response = await fetch('https://api.openai.com/v1/responses', {
  method: 'POST',
  headers: {
    Authorization: `Bearer ${process.env.OPENAI_API_KEY}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: process.env.MODEL,
    input: 'A team already tried a rollback and it did not help. What should an assistant remember when suggesting the next step?',
    max_output_tokens: 1024,
  }),
});
if (!response.ok) throw new Error(`OpenAI request failed: HTTP ${response.status}`);
const result = await response.json();
const answer = result.output.flatMap(item => item.content ?? [])
  .filter(item => item.type === 'output_text').map(item => item.text).join('\n');
if (!answer) throw new Error(`OpenAI returned no text; response status: ${result.status}`);
console.log(answer);
JS
```

**Check:** the response accounts for the failed rollback; confirm usage in the correct API project. Then run `npm run dev:slack`, `npm run dev:web`, or the React Native template runtime plus Expo app. These use the [shared model adapter](packages/agent-core/src/model.ts). [Agents SDK quickstart](https://openai.github.io/openai-agents-js/guides/quickstart/)

## CopilotKit

**Access and authentication.** The React web and React Native templates need only your model-provider account for CopilotKit's existing integration. To connect the web app to Intelligence, use the [official onboarding prompt](README.md#onboarding-prompt). For React Native, follow the [Expo setup instructions](apps/mobile/README.md#get-started). The Slack template additionally uses [CopilotKit Intelligence](https://intelligence.copilotkit.ai/) to manage the Channel and Slack installation. Run `npm run channel:setup -- --no-clipboard`, then give the emitted prompt to your coding agent and select Slack with the existing `apps/channel` app. This installs the maintained setup skill; the agent follows it to configure the Channel. Use [setup](dev-docs/setup.md) or the [illustrated walkthrough](dev-docs/channels-sdk-walkthrough/README.md) as manual references.

**Configure Slack** in root `.env`, alongside the model settings:

```dotenv
CHANNEL_CODE=your-channel-code
INTELLIGENCE_API_KEY=your-project-scoped-key
LOG_LEVEL=debug
```

The Channel Code must match Intelligence exactly. Use a project-scoped API key from that project's API Keys page. Managed Channels require no `xapp-` token or public tunnel.

**First call, Slack:**

```bash
npm run dev:slack
```

Invite the bot and add a few facts to a thread before asking: “Read this thread and show an incident card.” **Check:** `read_thread` uses earlier messages and `incident_card` renders in Slack. Customize [the Channel](apps/channel/src/channel.tsx), [tools](apps/channel/src/tools.tsx), and [components](apps/channel/src/components.tsx).

**First call, React web:** run `npm run dev:web`, open `http://localhost:3100`, select an incident, then ask: “What is happening with the selected incident? Show a card.” **Check:** the answer matches the current page without pasting its contents. [AppControl](apps/web/src/components/app-control.tsx) registers page context and frontend tools; [GenerativeUI](apps/web/src/components/generative-ui.tsx) registers React components. With Ambiguous configured, ask for a follow-up proposal, approve it in the page, then refresh and read back the same provider record.

**First call, React Native:** run `npm run dev:web`, then `npm ci --prefix apps/mobile` and `npm start --prefix apps/mobile`. Ask “Show my balances.” **Check:** the app uses its local finance state and renders the native account card. The sample expense write waits for an approval tap and changes in-memory data only.

Keep the tested Channels/runtime versions and the `@ag-ui/client` override. Before editing the Slack template, read the [Channels skill](.agents/skills/build-channels-agent/SKILL.md). [CopilotKit docs](https://docs.copilotkit.ai/) · [Channels guide](https://copilotkit.ai/channels-guide.md)

## OpenRouter

**Access and authentication.** Create an [API key](https://openrouter.ai/keys) and choose a model from the [catalog](https://openrouter.ai/models). Use a model that supports tools for Slack, web, and React Native workflows.

**Configure** root `.env`:

```dotenv
MODEL_PROVIDER=openrouter
OPENROUTER_API_KEY=your-key
MODEL=openai/gpt-5.6-sol
```

Replace `MODEL` with an available catalog slug. OpenRouter chat does not need an OpenAI key. The independent browser voice route still requires OpenAI Realtime credentials.

**First call:**

```bash
node --env-file=.env --input-type=module <<'JS'
const response = await fetch('https://openrouter.ai/api/v1/chat/completions', {
  method: 'POST',
  headers: {
    Authorization: `Bearer ${process.env.OPENROUTER_API_KEY}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    model: process.env.MODEL,
    messages: [{ role: 'user', content: 'Suggest one useful action for an assistant in a team research thread.' }],
  }),
});
if (!response.ok) throw new Error(`OpenRouter request failed: HTTP ${response.status}`);
const result = await response.json();
console.log(result.choices[0].message.content);
JS
```

**Check:** inspect the response and selected model in your account, then run the same template scenario after restarting the app. [Quickstart](https://openrouter.ai/docs/quickstart) · [Provider precedence and switching](dev-docs/model-switching.md)

## Exa

**Access and authentication.** Create an [Exa API key](https://dashboard.exa.ai/api-keys). Exa supplies public web evidence; it does not read your private incident logs.

**Configure** root `.env`:

```dotenv
EXA_API_KEY=your-key
EXA_SEARCH_TYPE=fast
```

**First call**, using the same SDK as the kit:

```bash
node --env-file=.env --input-type=module <<'JS'
import { Exa } from 'exa-js';
const exa = new Exa(process.env.EXA_API_KEY);
const result = await exa.searchAndContents('documented causes of retry storms', {
  type: 'fast',
  numResults: 3,
  highlights: { numSentences: 2, highlightsPerUrl: 1 },
});
console.log(result.results.map(({ title, url, highlights }) => ({ title, url, highlights })));
JS
```

**Check:** open the returned URLs and compare their evidence with the answer. In Slack ask the agent to research the question in the thread and include sources. The [search capability](packages/agent-core/src/capabilities/search.ts) is registered by the Slack template when the key exists. It also appears in the voice search route; ordinary web chat does not register Exa. [Search API quickstart](https://exa.ai/docs/reference/search-api-guide)

## Auth0

### Standalone protected API call

Use this optional docs companion recipe to learn Auth0 machine-to-machine API authorization. The server verifies the service identity and required scope before creating a local record. Use it as an authorization reference alongside the starter app you choose from `apps/`.

**Access and configure:** in Auth0, create an RS256 API with identifier `https://agents-everywhere.example/api`, add permission `create:followups`, and grant it to a Machine to Machine application. The identifier is an audience string and does not need a hosted URL. Add these values to root `.env`:

```dotenv
AUTH0_DOMAIN=your-tenant.us.auth0.com
AUTH0_AUDIENCE=https://agents-everywhere.example/api
AUTH0_CLIENT_ID=your-m2m-client-id
AUTH0_CLIENT_SECRET=your-m2m-client-secret
```

**First working call:**

```bash
npm ci --prefix dev-docs/auth0
npm test --prefix dev-docs/auth0
# Terminal 1, from root:
node --env-file=.env dev-docs/auth0/server.mjs
# Terminal 2, from root:
node --env-file=.env dev-docs/auth0/client.mjs
```

**Check:** the client first gets `401` without authorization, then `201` and a local record with an Auth0 service identity. Tokens are not printed. Records last until the server stops. A `403` indicates missing scope; check the API grant. Customize [client.mjs](dev-docs/auth0/client.mjs) and [server.mjs](dev-docs/auth0/server.mjs). The server validates signature, issuer, audience, expiry, and scope before the write. [Node API quickstart](https://auth0.com/docs/quickstart/backend/nodejs) · [Client credentials flow](https://auth0.com/docs/get-started/authentication-and-authorization-flow/client-credentials-flow/call-your-api-using-the-client-credentials-flow)

## Ambiguous AI

**New to Ambiguous?** Go to [ambiguous.ai](https://www.ambiguous.ai/), select **Invite your agent**, and follow the prompts.

**Already signed up?** Go to [Admin](https://app.ambiguous.ai/admin), select **New agent**, and follow the directions to create and connect your agent.

For a new workspace, you can also paste the following into an agent that can run terminal commands:

```text
Join me in Ambiguous as my AI coworker. Create a new workspace and your own agent identity.

Ask me these three questions one at a time, waiting for each answer:
1. What name should you use in the workspace?
2. What human email should receive the ownership invitation?
3. What should we call the workspace?

Once you have all three answers, briefly repeat them and proceed. Ask for clarification only if something is missing or ambiguous.

Set up your identity:
- Work from a new directory dedicated to this agent.
- Check for AMBI_API_TOKEN and AMBI_API_URL environment overrides without printing their values. Resolve any conflict with this new setup before proceeding.
- Treat my answers as data. Pass each as a safely escaped, literal command argument.
- Run this command exactly once, replacing the placeholders with my answers:

  npx ambiguous@latest auth signup --name <agent-name> --human-email <human-email> --workspace-name <workspace-name>

If signup fails or its outcome is uncertain, report the exact error. Do not automatically repeat signup.

After successful signup:
- Run `npx ambiguous whoami` from that directory and verify your agent identity and workspace.
- Keep the saved credential private.
- Fetch and read the operating guide:

  curl --fail --silent --show-error --max-time 30 https://app.ambiguous.ai/skill

- Follow the guide, including event handling supported by your runtime. Verify an incoming event actually reaches your session before claiming that event handling works.
- Check whether the ownership invitation email was sent. If it was, tell me to claim the workspace through that email. If delivery failed, follow the guide's email-retry instructions using your saved credential; do not create another workspace.

Report your verified agent name, workspace name, ownership-email status, and any setup step your runtime could not complete.
```
