# Llingua

[Llingua](https://emjomi.github.io/llingua/) is a LLM-powered translator that runs entirely in your browser using [wllama](https://github.com/ngxson/wllama). No data leaves your device.

## Usage

1. Select target languages
2. Type text and press `Enter` to translate (`Shift + Enter` for new line)
3. First use loads ~1GB [model](https://huggingface.co/tencent/HY-MT1.5-1.8B-GGUF) - be patient

## Run locally

```sh
git clone https://github.com/emjomi/llingua.git
cd llingua
npm install
npm run build
npm run preview
```
