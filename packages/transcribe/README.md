# @clipy/transcribe

Converts speech from audio or video recordings into timestamped text segments using local models or external transcription services.

## Requirements

- Node.js 20.19 or later
- pnpm
- FFmpeg installed and available on PATH

Check FFmpeg using:

```sh
ffmpeg -version
```

If FFmpeg is installed in a custom location, set the `CLIPY_FFMPEG` environment variable to its executable path.

Example for Windows Command Prompt:

```bat
set CLIPY_FFMPEG=C:\ffmpeg\bin\ffmpeg.exe
```

## Supported providers

| Configuration kind | Provider |
| --- | --- |
| none | Transcription disabled |
| local | On-device speech recognition |
| openai-compatible | OpenAI, Groq or compatible endpoint |
| deepgram | Deepgram |
| assemblyai | AssemblyAI |
| elevenlabs | ElevenLabs |
| gemini | Google Gemini |

## Usage

Import the public transcription API:

```ts
import {
  transcribe,
  downloadModel,
  modelStatus
} from "@clipy/transcribe";
```

### Disable transcription

```ts
await transcribe("recording.mp4", {
  kind: "none"
});
```

### Local transcription

```ts
await downloadModel("english-fast");

const result = await transcribe("recording.mp4", {
  kind: "local",
  model: "english-fast"
});

console.log(result.segments);
```

### OpenAI-compatible provider

```ts
await transcribe("recording.mp4", {
  kind: "openai-compatible",
  baseUrl: "https://api.openai.com/v1",
  apiKey: process.env.OPENAI_API_KEY!,
  model: "whisper-1"
});
```

### Deepgram

```ts
await transcribe("recording.mp4", {
  kind: "deepgram",
  apiKey: process.env.DEEPGRAM_API_KEY!
});
```

### AssemblyAI

```ts
await transcribe("recording.mp4", {
  kind: "assemblyai",
  apiKey: process.env.ASSEMBLYAI_API_KEY!
});
```

### ElevenLabs

```ts
await transcribe("recording.mp4", {
  kind: "elevenlabs",
  apiKey: process.env.ELEVENLABS_API_KEY!
});
```

### Gemini

```ts
await transcribe("recording.mp4", {
  kind: "gemini",
  apiKey: process.env.GEMINI_API_KEY!
});
```

## Local models

| Model ID | Model | Download size |
| --- | --- | --- |
| english-fast | NVIDIA Parakeet TDT-CTC 110M | 104 MB |
| multilingual | Whisper base | 161 MB |

Local inference uses sherpa-onnx-node,
which must be provided by the host.

Download models explicitly before use.

```ts
for (const model of [
  "english-fast",
  "multilingual"
] as const) {
  if (!modelStatus(model).installed) {
    await downloadModel(model);
  }

  console.log(modelStatus(model));
}
```

`modelStatus()` returns the installation status
and model directory.

`downloadModel()` verifies downloads using
SHA-256 checksums and returns the installed
model directory.

## Transcription result

The result contains:

- `segments`: recognized text with start and
  end timestamps in milliseconds.
- `engine`: the transcription engine used.
- `language`: language code, when available.
- `gaps`: intervals without transcription.

Each gap contains `startMs`, `endMs` and
`reason`. A gap indicates missing transcript
coverage, not necessarily silence.

Disabled transcription or recordings without
audio can produce a gap covering the entire
recording.

## Privacy

The `none` and `local` configurations do not
upload recording audio for transcription.

OpenAI-compatible, Deepgram, AssemblyAI,
ElevenLabs and Gemini send audio to their
configured service endpoints.

Local model downloads require an internet
connection but do not upload recording audio.

Keep API keys in environment variables and
never commit them to the repository.

## Source

- [Public API](src/index.ts)
- [Configuration types](src/types.ts)
- [Model management](src/models.ts)
- [Audio processing](src/audio.ts)
