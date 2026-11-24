# Claude Code with local Ollama

## Start from your project folder

```sh
claude-local          # Start on Gemma, the default
claude-local gemma    # Gemma 4 E2B
claude-local qwen     # Qwen 3.5 4B MLX
claude-local granite  # Granite 4.2 3B
```

The launcher starts the standalone Ollama server when needed. If Claude Code asks whether you trust the folder, confirm only if you trust that project.

## Switch models without starting over

1. Let the current reply finish.
2. In Claude Code, type `/model`.
3. Highlight Gemma, Qwen, or Granite and press `s` to switch for this session.

To reopen the latest conversation, type `/exit` and run the following from the **same project folder**. Replace `granite` with `gemma` or `qwen` to resume on that model.

```sh
claude-local granite --continue
```

## Pick a model

| Model | Use it for |
| --- | --- |
| **Gemma 4 E2B** | Everyday tasks and quick file reads. |
| **Qwen 3.5 4B MLX** | A second opinion on tricky code or logic. It can take longer in Claude Code. |
| **Granite 4.2 3B** | Code fixes that need file reads. It fixed the test bug correctly on this Mac, but the full test took about 1 minute 49 seconds. |

Your 8 GB M1 setup uses an **8,192-token context window** and a **2,048-token reply limit**. Work on a few files at a time and run your project tests before accepting code changes.

## If the launcher gets stuck

- Open a new Terminal tab, return to your project folder, and run `claude-local`.
- If you're switching models, let the current reply finish first so Ollama can load the next model.
- If the launcher says Ollama failed to start, its log is at `~/.local/state/claude-local/ollama.err.log`.

## Set up Z shell on a Mac

macOS includes Z shell (zsh) at `/bin/zsh`. In Terminal, check that it's available and see your account's default shell:

```sh
/bin/zsh --version
echo "$SHELL"
```

If the second line shows `/bin/zsh`, Z shell is already your default. To try it in the current Terminal tab, run `/bin/zsh`. Type `exit` to return to the previous shell.

To set Z shell as your default for new tabs and windows, run:

```sh
chsh -s /bin/zsh
```

Then open a new Terminal tab and run `echo "$SHELL"` again. You can also choose a shell in **Terminal > Settings > General > Shells open with**. [Apple Terminal guide](https://support.apple.com/guide/terminal/change-the-default-shell-trml113/mac)

If you specifically want to install a separate copy of zsh and you already have Homebrew, run `brew install zsh`. For the setup above, use the copy that comes with macOS. [Homebrew zsh formula](https://formulae.brew.sh/formula/zsh)

## Add another local model to Claude Code

Z shell runs the commands. Ollama downloads and serves the model. Start in Terminal and look up the exact model tag in the [Ollama model library](https://ollama.com/library). On this 8 GB Mac, start with a small quantized build. For Claude Code workflows that read or edit files, look for tool support.

### 1. Download it to Ollama

The Granite tag below is an example you already have. Replace it with the exact tag for the model you want.

```sh
claude-local --version
export OLLAMA_HOST=127.0.0.1:11435
MODEL='granite4.2:3b-q4_K_M'
ollama pull "$MODEL"
ollama show "$MODEL"
```

The first command starts your standalone Ollama server if needed. In the output from `ollama show`, check **Capabilities** for `tools` if you want the model to use Claude Code tools. [Ollama CLI reference](https://docs.ollama.com/cli)

### 2. Try it in Claude Code

In the same Terminal tab, go to your project folder and run:

```sh
CLAUDE_CODE_MAX_CONTEXT_TOKENS=8192 \
CLAUDE_CODE_MAX_OUTPUT_TOKENS=2048 \
CLAUDE_CODE_DISABLE_THINKING=1 \
  ollama launch claude --model "$MODEL" --yes -- --bare --effort low \
    --tools Read,Glob,Grep,Edit,Write,Bash --permission-mode default
```

Use a smaller context value if the model supports fewer than 8,192 tokens. Try a small task first, such as asking it to read a short project file and summarize it. [Ollama's Claude Code guide](https://docs.ollama.com/integrations/claude-code)

### 3. Add it to your shortcut and model picker

Back up and open your existing launcher:

```sh
cp -p ~/.local/bin/claude-local ~/.local/bin/claude-local.backup-$(date +%Y%m%d-%H%M%S)
nano ~/.local/bin/claude-local
```

In the `case` block near the top, add the line below before `esac`. Replace `newmodel` with the short name you want to type and `example-model:example-tag` with your downloaded model tag.

```sh
    newmodel) model=example-model:example-tag; shift ;;
```

In `model_picker`, add a row to the `options` list with that same model tag. Use a comma between rows and omit a comma after the final row. [Claude Code model picker settings](https://code.claude.com/docs/en/settings-reference#modelpicker)

```json
{"model": "example-model:example-tag", "label": "My new local model"}
```

In Nano, press **Ctrl+O**, **Enter**, then **Ctrl+X**. Check and start the updated launcher:

```sh
/bin/sh -n ~/.local/bin/claude-local
claude-local newmodel
```

Open `/model` to check that your new model appears. To pick up an existing chat after updating the launcher, exit and start it again from the same project folder with `claude-local newmodel --continue`.

## Recreate the Claude Local launcher

This is a copy of the launcher installed on this Apple Silicon Mac. It uses Homebrew Ollama at `/opt/homebrew/bin/ollama` and native Claude Code at `~/.local/bin/claude`. If either is missing on a future Mac, install Homebrew Ollama with `brew install ollama` and follow the [native Claude Code setup guide](https://code.claude.com/docs/en/setup).

1. In Terminal, create the command directory and open the launcher in Nano:

   ```sh
   mkdir -p ~/.local/bin
   nano ~/.local/bin/claude-local
   ```

2. Paste the full script below. If you are restoring an existing launcher, replace its contents with this script. In Nano, press **Ctrl+O**, **Enter**, then **Ctrl+X**.

```sh
#!/bin/sh
set -eu

ollama=/opt/homebrew/bin/ollama
model=gemma4:e2b-claude
case "${1-}" in
    gemma) shift ;;
    qwen) model=qwen3.5:4b-mlx; shift ;;
    granite) model=granite4.2:3b-q4_K_M; shift ;;
esac
model_picker='{
    "modelPicker": {
        "replaceBuiltInOptions": true,
        "options": [
            {"model": "gemma4:e2b-claude", "label": "Gemma 4 E2B"},
            {"model": "qwen3.5:4b-mlx", "label": "Qwen 3.5 4B MLX"},
            {"model": "granite4.2:3b-q4_K_M", "label": "Granite 4.2 3B"}
        ]
    }
}'
state_dir="$HOME/.local/state/claude-local"
job=local.claude-local.ollama
export OLLAMA_HOST=127.0.0.1:11435
export CLAUDE_CODE_MAX_CONTEXT_TOKENS=8192
export CLAUDE_CODE_MAX_OUTPUT_TOKENS=2048
export CLAUDE_CODE_DISABLE_THINKING=1

if [ ! -x "$ollama" ] || [ ! -x "$HOME/.local/bin/claude" ]; then
    printf '%s\n' 'This launcher needs Homebrew Ollama and the native Claude Code CLI.' >&2
    exit 1
fi

ollama_ready() {
    /usr/bin/curl -fsS --max-time 1 "http://$OLLAMA_HOST/api/version" >/dev/null 2>&1
}

if ! ollama_ready; then
    mkdir -p "$state_dir"
    if ! /bin/launchctl print "gui/$(/usr/bin/id -u)/$job" >/dev/null 2>&1; then
        /bin/launchctl submit -l "$job" -o "$state_dir/ollama.out.log" -e "$state_dir/ollama.err.log" -- \
            /usr/bin/env "OLLAMA_HOST=$OLLAMA_HOST" OLLAMA_NO_CLOUD=1 OLLAMA_CONTEXT_LENGTH=8192 \
            OLLAMA_FLASH_ATTENTION=1 OLLAMA_KV_CACHE_TYPE=q8_0 "$ollama" serve
    fi
    attempts=0
    until ollama_ready; do
        if [ "$attempts" -ge 30 ]; then
            printf 'Local Ollama did not start. See %s/ollama.err.log\n' "$state_dir" >&2
            exit 1
        fi
        attempts=$((attempts + 1))
        sleep 1
    done
fi

if ! "$ollama" show "$model" >/dev/null 2>&1; then
    printf 'The local model %s is missing from Ollama.\n' "$model" >&2
    exit 1
fi

export PATH="$HOME/.local/bin:$PATH"
exec "$ollama" launch claude --model "$model" --yes -- \
    --bare --settings "$model_picker" --effort low \
    --tools Read,Glob,Grep,Edit,Write,Bash --permission-mode default "$@"
```

3. Make it executable and check the script:

   ```sh
   chmod u+x ~/.local/bin/claude-local
   /bin/sh -n ~/.local/bin/claude-local
   command -v claude-local
   ```

   If the last command prints nothing, run `export PATH="$HOME/.local/bin:$PATH"`. Add that same export line once to `~/.zshrc` to use the shortcut in future Terminal tabs.

4. In your project folder, run `claude-local --version` to check startup, then run `claude-local` to open Claude Code.

### Restore the models on a fresh Mac

Run `claude-local --version` once to start the standalone Ollama server. If Gemma is missing, the launcher will say so after starting Ollama. Then, in Terminal, run the block below to download the three models and create the Gemma preset used by the launcher.

```sh
export OLLAMA_HOST=127.0.0.1:11435
ollama pull gemma4:e2b-it-qat
ollama pull qwen3.5:4b-mlx
ollama pull granite4.2:3b-q4_K_M

preset="$(mktemp)"
printf '%s\n' 'FROM gemma4:e2b-it-qat' 'PARAMETER num_ctx 8192' > "$preset"
ollama create gemma4:e2b-claude -f "$preset"
rm "$preset"
```

Run `claude-local --version` again after the downloads finish. Use the model-picker and session commands earlier in this README to start working.
