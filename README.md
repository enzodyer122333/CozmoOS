# CozmoOS

**An unofficial, AI-powered "operating system" for the Cozmo robot.**

CozmoOS turns your Anki/DDL Cozmo into an autonomous companion. He roams on his own, looks after his own battery, obeys spoken or typed commands, and chats with you using AI. Everything runs as a Python script on your computer. Nothing is installed on or changed in the robot, and no firmware is touched.

It works with the current Cozmo app (3.6.0), using a small patch that gets the old Anki SDK past its version check. CozmoOS is a script, not a real operating system. The name is just for fun.

> **Unofficial fan project.** Not affiliated with or endorsed by Digital Dream Labs or Anki. "Cozmo" is a trademark of its owners. It drives a physical robot, so use it at your own risk, on the floor, away from edges.

**Contents:** [Features](#features) · [How it works](#how-it-works) · [Requirements](#requirements) · [Installation](#installation) · [Running it](#running-it) · [Commands](#commands) · [Settings](#settings) · [Better speech recognition](#better-speech-recognition-optional) · [Shortcut](#one-word-shortcut-optional-macos) · [Troubleshooting](#troubleshooting) · [Limitations](#known-limitations) · [Privacy](#privacy) · [Contributing](#contributing) · [Credits](#credits)

## Features

- **Roams freely** using Cozmo's own freeplay behaviors, for as long as the connection holds.
- **Looks after his battery:** when it's low he finds his charger, docks, charges, then goes back to playing.
- **Voice and typed commands:** drive, turn, pick up and put down cubes, stack two cubes, knock over a stack, dock, explore, stop and more.
- **AI conversation:** say "Hey Cozmo" (or just type) and chat. Replies come from an LLM via [Groq](https://console.groq.com)'s free tier, spoken in Cozmo's own voice with a matching emotion animation.
- **Offline speech recognition** with [Vosk](https://alphacephei.com/vosk/), plus optional more accurate models.
- **Back to roaming 15 seconds after your last command** (configurable). Say or type `stop` at any time.

## How it works

```
 You speak --> computer microphone --> Vosk (offline speech to text)
                                            |
                      +---------------------+----------------------+
                      | a known command                            | anything else
                      v                                            v
          runs directly on Cozmo                      text sent to Groq (LLM)
          (drive, pick up, dock...)                   --> reply + emotion, spoken by Cozmo

 Python script --(Cozmo SDK, USB)--> phone running the Cozmo app in SDK mode --(Wi-Fi)--> Cozmo
```

Commands are understood by the script itself, so they work instantly and offline. Only free-form chat is sent to the AI. The phone stays Cozmo's brain, and your computer runs the code.

## Requirements

- Cozmo, his charger and cubes
- An iPhone/iPad with the official Cozmo app, and a USB cable. Tested on iOS with app version **3.6.0**. Android is untested.
- A computer with **Python 3.9**. Tested on macOS (Python 3.9.13), using the default zsh terminal.
- A free [Groq API key](https://console.groq.com/keys)
- Internet on the computer (only for the AI chat; the speech recognition runs locally)

## Installation

**1. Get the code.** On this page click the green **Code** button, then **Download ZIP**, and unzip it. (Or run `git clone` with this repository's address.) Then open a terminal in the unzipped folder:

```bash
cd path/to/CozmoOS
```

**2. Install Python 3.9** if you don't have it. Download it from [python.org](https://www.python.org/downloads/). Check with:

```bash
python3.9 --version
```

**3. Create a virtual environment and install the packages:**

```bash
python3.9 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

You should now see `(venv)` at the start of your terminal prompt. Run `source venv/bin/activate` again whenever you open a new terminal window.

**4. Download the small English speech model** (about 40 MB, runs offline):

```bash
curl -LO https://alphacephei.com/vosk/models/vosk-model-small-en-us-0.15.zip
unzip vosk-model-small-en-us-0.15.zip
```

Want better recognition? See [Better speech recognition](#better-speech-recognition-optional) below.

**5. Save your Groq API key.** Create a free key at [console.groq.com/keys](https://console.groq.com/keys), then save it (never share or upload this file):

```bash
echo "PASTE_YOUR_KEY_HERE" > groq_key.txt
```

## Running it

1. Put Cozmo on his charger and turn him on.
2. Connect the phone to the `Cozmo_XXXXXX` Wi-Fi, open the Cozmo app and wait until he wakes up.
3. In the app: menu (top right) -> swipe left -> turn on **SDK mode**. This resets every time the app reconnects.
4. Plug the phone into the computer, unlock it and tap **Trust**. Keep the app open on screen (set Auto-Lock to **Never**).
5. In the terminal, with `(venv)` showing, run:

```bash
caffeinate -i python autonomous.py
```

`caffeinate` stops a Mac from sleeping while the script runs. The first time, allow microphone access for your terminal. Press `Ctrl+C` to stop.

## Commands

Type them in the terminal, or say them (start with "Hey Cozmo", or just say the simple ones). After a command he stays in command mode for a few seconds, so you can give the next one without repeating "Hey Cozmo".

| Say or type | What happens |
|---|---|
| `dock`, `go home`, `go to the charger` | Finds his charger, docks and charges |
| `explore`, `roam around`, `let's play` | Leaves the charger and roams |
| `drive forward`, `go forward 50 centimeters`, `back up` | Drives (default 20 cm; numbers can be digits or words) |
| `turn left`, `turn right 45 degrees`, `turn around` | Turns on the spot |
| `pick up the cube`, `put it down` | Finds and lifts a cube / sets it down |
| `stack the cubes` | Picks up one cube and puts it on another |
| `knock over the stack` | Looks up at the top cube and knocks the stack over |
| `look around`, `look up`, `look down`, `pop a wheelie` | Fun actions |
| `say <something>` | He says it out loud |
| `stop` | Cancels what he's doing |
| anything else | Chats with you |

## Settings

Everything you might want to change is at the top of `autonomous.py`:

| Setting | Default | What it does |
|---|---|---|
| `VOSK_MODEL_PATH` | `vosk-model-small-en-us-0.15` | Which speech model to use |
| `GROQ_MODEL` | `openai/gpt-oss-20b` | Which Groq model answers chat |
| `ROAM_DELAY_SECONDS` | `15` | Seconds after your last command before he roams again |
| `LOW_BATTERY_VOLTS` | `3.65` | Below this he goes to charge |
| `CRITICAL_BATTERY_VOLTS` | `3.5` | Below this he charges no matter what |
| `CHARGE_MINUTES` | `35` | How long he stays on the charger |
| `VOICE_COMMANDS` | `True` | Say simple commands without "Hey Cozmo" |
| `HEAD_ANGLE` | `0` | Head angle when searching for the charger (try `-10` if he can't see it) |
| `KNOCK_HEAD_ANGLE`, `KNOCK_LIFT_HEIGHT`, `KNOCK_SPEED` | `30`, `0.55`, `150` | Tuning for the knock-over |

## Better speech recognition (optional)

The default speech model is small and fast, but not very accurate. Better options (macOS commands; run them in the project folder with `(venv)` showing):

| Model | Download | Word error rate* | Notes |
|---|---|---|---|
| `vosk-model-small-en-us-0.15` (default) | 40 MB | 9.85% | Lightest |
| `vosk-model-en-us-0.22-lgraph` | 128 MB | 7.82% | **Recommended upgrade.** Light, and keeps the strict command listener |
| `vosk-model-en-us-0.22` | 1.8 GB | 5.69% | Most accurate, but Vosk says big models can need up to 16 GB of RAM while running. Not recommended on 8 GB machines |

\*On clean speech, from the [Vosk model list](https://alphacephei.com/vosk/models). Real-world results will be worse.

To switch to the recommended model:

```bash
curl -L -C - -O https://alphacephei.com/vosk/models/vosk-model-en-us-0.22-lgraph.zip
unzip vosk-model-en-us-0.22-lgraph.zip
sed -i '' 's|^VOSK_MODEL_PATH = .*|VOSK_MODEL_PATH = "vosk-model-en-us-0.22-lgraph"|' autonomous.py
```

Big (static) models can't be limited to a few phrases, so with one of those CozmoOS automatically turns its extra command listener off. To go back, point `VOSK_MODEL_PATH` at `vosk-model-small-en-us-0.15` again. Watch your Mac's Memory Pressure graph in Activity Monitor when trying a bigger model.

Tips: sit 20 to 30 cm from the microphone in a quiet room, speak clearly with a short pause before and after, and use a headset or USB microphone if you have one. Typed commands always work.

## One-word shortcut (optional, macOS)

To start everything by just typing `CozmoOS` in any terminal window, run this once (adjust the folder name if yours isn't `cozmo_project`):

```bash
cat >> ~/.zshrc << 'EOT'

# CozmoOS shortcut
CozmoOS() {
  (
    cd "$HOME/cozmo_project" || { echo "Can't find the cozmo_project folder in your home folder."; exit 1; }
    caffeinate -i "$HOME/cozmo_project/venv/bin/python" autonomous.py
  )
}
alias cozmoos=CozmoOS
# end CozmoOS shortcut
EOT
source ~/.zshrc
```

Then get Cozmo ready as above and type `CozmoOS`. To remove it: `sed -i '' '/# CozmoOS shortcut/,/# end CozmoOS shortcut/d' ~/.zshrc`

## How it connects to the current app

The old Anki SDK refuses to connect to newer app versions because of a version check. The script gets past it by importing `cozmo` first and then patching two values in `cozmoclad` (see the top of `autonomous.py`). Nothing on the robot is modified.

## Troubleshooting

- **`python3.9: command not found`**: install Python 3.9 (Installation step 2).
- **`pip: command not found`**: the virtual environment isn't active. Run `source venv/bin/activate`.
- **"Hmm, my brain is a bit fuzzy"**: the AI call failed. Read the `LLM error` line above it in the terminal. If it says `model_not_found`, Groq has retired the model: list the current ones with `curl -s https://api.groq.com/openai/v1/models -H "Authorization: Bearer $(cat groq_key.txt)"` and change `GROQ_MODEL` at the top of the script.
- **He ignores or mishears spoken commands:** check the `Heard:` lines in the terminal, make sure the microphone is allowed (System Settings -> Privacy & Security -> Microphone), and try a better speech model (see above).
- **He can't find his charger:** use good light, a clear floor, and face the charger's black-and-white marker into the room.
- **He won't roam:** he may be charging (type `explore`), or holding a cube (he puts it down before roaming).
- **Disconnects:** the script can't turn SDK mode back on for you. Keep the phone unlocked with the app on screen, and keep Cozmo charged.
- **Battery thresholds are estimates.** The script prints the voltage every 15 seconds so you can tune `LOW_BATTERY_VOLTS` and `CHARGE_MINUTES`.

## Known limitations

- The phone is still Cozmo's brain, and a computer still has to run the code. This is not a standalone robot.
- SDK mode must be switched on in the app each time, and the script stops if the connection drops.
- Cozmo has no microphone, so speech is picked up by your computer's microphone.
- English only (Cozmo's voice is English).
- Tested on macOS with an iPhone. Other setups may need changes.
- Cube actions work best in good light with cubes on a clear floor. He has no sensor at his back, so only use `back up` on the floor, never near an edge.

## Privacy

Speech is turned into text on your computer. Only that text is sent to Groq to generate replies. No audio leaves your machine.

## Contributing

Issues and pull requests are welcome. When reporting a problem, include the terminal output around the error, and remove any API keys first.

## Credits

- Built on the [Anki Cozmo Python SDK](https://github.com/anki/cozmo-python-sdk), [Vosk](https://alphacephei.com/vosk/) for offline speech recognition, and [Groq](https://groq.com) for the AI replies.
- The version-check patch that lets the SDK connect to the current app comes from a community write-up on getting the Cozmo SDK working with the current app.

## License

MIT. See the `LICENSE` file.
