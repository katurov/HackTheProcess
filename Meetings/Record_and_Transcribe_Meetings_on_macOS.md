# How to Record and Transcribe Meetings on macOS (Pet Project Guide)

## Prerequisites

You will need to install the following free software via [Homebrew](https://brew.sh):

* **[ffmpeg](https://formulae.brew.sh/formula/ffmpeg)**: `brew install ffmpeg`
* **[Blackhole](https://github.com/ExistentialAudio/BlackHole) 2ch**: `brew install blackhole-2ch`
* **whisper.cpp**: Follow the [official build instructions](https://github.com/ggml-org/whisper.cpp).

> **Note:** It is considered good etiquette (and often a legal requirement) to inform participants that you are recording the meeting.

## Setup & Preparation

Once the software is installed (and you have restarted your system), you need to configure your audio devices using **[Audio MIDI Setup](https://support.apple.com/en-gb/guide/audio-midi-setup/ams59f301fda/mac)**:

1. Create a new **Multi-Output Device** consisting of **BlackHole 2ch** and your current **headphones**.
2. **Important**: You cannot adjust the volume of a Multi-Output Device using system controls. Set a comfortable volume on your headphones *before* switching the system output to the Multi-Output Device.
3. In your meeting software (Zoom, Teams, etc.), select the **Multi-Output Device** as your speaker/output.

### Identify Device IDs

Run the following command to find the correct device IDs for your mic and system audio:

```bash
ffmpeg -f avfoundation -list_devices true -i ""

```

Look for the IDs in the output. For example:
```bash
[AVFoundation indev @ 0x131e04290] AVFoundation video devices:
[AVFoundation indev @ 0x131e04290] [0] FaceTime HD Camera
[AVFoundation indev @ 0x131e04290] [1] Fataffe Camera
[AVFoundation indev @ 0x131e04290] [2] Fataffe Desk View Camera
[AVFoundation indev @ 0x131e04290] [3] Capture screen 0
[AVFoundation indev @ 0x131e04290] AVFoundation audio devices:
[AVFoundation indev @ 0x131e04290] [0] Mic and rec
[AVFoundation indev @ 0x131e04290] [1] Fataffe Microphone
[AVFoundation indev @ 0x131e04290] [2] External Microphone
[AVFoundation indev @ 0x131e04290] [3] MacBook Pro Microphone
[AVFoundation indev @ 0x131e04290] [4] Microsoft Teams Audio
[AVFoundation indev @ 0x131e04290] [5] BlackHole 2ch
```

* **ID 2**: Your physical microphone.
* **ID 5**: The BlackHole channel (which captures the meeting audio).

## The Recording Process

When the meeting starts, run the following command to record high-quality audio:

```bash
ffmpeg -f avfoundation -i ":2" -f avfoundation -i ":5" -filter_complex \
"[0:a]aresample=44100[mic]; \
 [1:a]pan=mono|c0=c0,aresample=44100[sys]; \
 [mic][sys]amix=inputs=2:duration=first[aout]" \
 -map "[aout]" -ac 2 -c:a aac -b:a 192k ~/Downloads/output.m4a

```

However, for **whisper.cpp**, the following settings are optimized for transcription:

```bash
ffmpeg -f avfoundation -i ":2" -f avfoundation -i ":5" -filter_complex \
"[0:a]aresample=16000[mic]; \
 [1:a]pan=mono|c0=c0,aresample=16000[sys]; \
 [mic][sys]amix=inputs=2:duration=first[aout]" \
 -map "[aout]" -ac 1 -c:a pcm_s16le ~/Downloads/output.wav

```

* **To stop recording:** Press `q` in the terminal.
* **Crucial Note:** This records everything picked up by your mic hardware. Even if you "Mute" yourself in the meeting app, the terminal will still record your local audio.

## Transcription

Once you have your `.wav` file, use **whisper.cpp** to transcribe it.

```bash
./build/bin/whisper-cli -m models/ggml-large-v3.bin -p 3 -f ~/Downloads/output.wav -pp -otxt -l en

```

> **Pro Tip:** To significantly improve accuracy for technical terms or names, use the `--prompt` flag. Provide a string of project names, technologies, or participant names.
> **Example:** `--prompt "Alexander, Popantopoulo, MediaReport, Neverwinter"`

The result will be a file named `~/Downloads/output.wav.txt`, which is ready to be processed by your favorite LLM (ChatGPT, Claude, etc.).

## Post-Processing

Once you have your summary or action items, **delete the temporary audio and text files**. In a pet project or professional environment, the only thing worth keeping is the final, well-formatted result.
