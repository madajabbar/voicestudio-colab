# VoiceStudio on Google Colab (GPU)

Generate high-quality **Indonesian TTS** (OmniVoice, 600+ languages) on a free Colab GPU.
The model cache persists on **Google Drive** — download once, reuse in later sessions.

Works as a remote backend for local pipelines: expose `/generate` via a Cloudflare tunnel
and point your local code at the tunnel URL.

## Quick start

1. Open [VoiceStudio_Colab.ipynb](VoiceStudio_Colab.ipynb) in Colab
   (File → Upload notebook → select file, or GitHub import).
2. Runtime → Change runtime type → **T4 GPU**.
3. Run cells top-to-bottom.
   - First run downloads the model (~2.5GB) into `MyDrive/omnivoice_colab/hf` — subsequent runs mount it.
4. Paste your narration into `TEXT` in the last cell and run.
   - Output: `MyDrive/omnivoice_colab/audio_out.mp3`.
   - Long text is auto-chunked (≤28 words) and stitched, to stay under the server's compute cap.
5. Copy the printed **tunnel URL** and **API key**.

## Using it from a local pipeline

Server base URL: `https://<tunnel>.trycloudflare.com`

```bash
# generate audio for a text snippet
curl -X POST "$TUNNEL_URL/generate" \
  -H "Authorization: Bearer $API_KEY" \
  -F "text=Selamat pagi, ini channel jam tiga pagi." \
  -F "language=id" \
  -F "instruct=female, young adult, high pitch" \
  -o out.mp3
```

Python:

```python
import requests
r = requests.post(url + "/generate",
    files={"text": (None, "narasi di sini"),
           "language": (None, "id"),
           "instruct": (None, "female, young adult, high pitch")},
    headers={"Authorization": "Bearer " + key}, timeout=600)
open("out.mp3", "wb").write(r.content)
```

You can then apply your usual post-processing (EQ, compressor, reverb — the DSP chain) locally.

## Notes / limits

- **Colab free is ephemeral**: idle sessions may disconnect (~90 min). Re-run the notebook to restart; the model cache on Drive makes this fast.
- The tunnel URL changes every session — update your pipeline's base URL accordingly.
- API key is generated once and saved to `api_key.txt` on Drive, so it stays stable across sessions.
