# Connect OMP to Demerzel

Demerzel's home-page **OMP** wizard walks through choosing an access key, copying a provider configuration, and checking the discovered models. The provider lists the models available to that key dynamically; you do not need to add each model to YAML. The browser cannot edit files on your machine. These steps also work without the browser.

## 1. Choose an access key

Start Demerzel and choose an access key that allows `openai-completions`. The key's group and model filters determine which configured models OMP can discover. To make the entire Chat Completions catalogue available, leave those two filters unrestricted while keeping the protocol restricted to `openai-completions`. Use a separate client key rather than the management or unrestricted default key.

Demerzel must be reachable from the machine running OMP. On a local installation the OpenAI-compatible base URL is `http://127.0.0.1:3001/v1` (adjust the host and port to your installation). The `/v1` suffix is required for inference requests.

## 2. Merge the provider and store its key separately

Merge the `demerzel` provider into `~/.omp/agent/models.yml` under `providers:`. If the file already has a `demerzel:` entry, **replace only that entry**; never add a duplicate YAML mapping key. If it does not exist, add it beneath the existing `providers:` mapping. Preserve every other provider.

```yaml
providers:
  demerzel:
    baseUrl: http://127.0.0.1:3001/v1
    api: openai-completions
    apiKey: DEMERZEL_API_KEY
    discovery:
      type: openai-models-list
```

In `~/.omp/agent/.env`, set `DEMERZEL_API_KEY` to the separate Demerzel access key. Create the file as owner-only (`0600`), and retain any existing variables. Never put the secret in `models.yml` or in a shell command, issue, screenshot or log. OMP treats `apiKey: DEMERZEL_API_KEY` as an environment-variable reference **only when the variable exists**; otherwise it sends the literal string and the request fails authentication. Restart OMP after changing the files.

Do **not** put a credential into `auth:`. That field accepts only `apiKey`, `none` or `oauth`. The `discovery:` block belongs inside `providers.demerzel`, not at the file root. OMP asks Demerzel's authenticated `GET /v1/models` for the models allowed by this key. Non-chat, decision-only models do not appear as chat choices; a provider listing a model does not itself prove that the upstream can answer a chat request.

## 3. Confirm model selection and inference

```sh
omp models demerzel
omp --model demerzel/YOUR_MODEL
```

The first command must list multiple models if the key exposes them. Select **any** listed ID as `demerzel/YOUR_MODEL`, then ask OMP to answer a short prompt. In an existing OMP session, use `/model` to switch between discovered Demerzel models. OMP may consolidate effort-tier `-thinking` aliases into one selectable model, so its picker can show fewer rows than Demerzel's raw list. Run `omp models refresh` after changing Demerzel's groups or model catalogue if the local model cache has not updated yet. A custom `models.yml` provider does **not** appear in OMP's `/login` list or first-run provider setup; those screens are for built-in login methods.

If nothing is listed, check the YAML indentation, the same OMP profile/agent directory, the `DEMERZEL_API_KEY` environment variable and the key's group/protocol filters. If a listed model fails inference, check its specific Demerzel route and upstream availability; model discovery alone does not prove every upstream supports Chat Completions. Keep credentials out of diagnostic output.
