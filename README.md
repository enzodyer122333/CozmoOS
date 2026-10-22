# CozmoOS

**An unofficial, AI-powered "operating system" for the Cozmo robot.**

CozmoOS turns your Anki/DDL Cozmo into an autonomous companion. He roams on his own, looks after his own battery, obeys spoken or typed commands, and chats with you using AI. Everything runs as a Python script on your computer. Nothing is installed on or changed in the robot, and no firmware is touched.

It works with the current Cozmo app (3.6.0), using a small patch that gets the old Anki SDK past its version check. CozmoOS is a script, not a real operating system. The name is just for fun.

> **Unofficial fan project.** Not affiliated with or endorsed by Digital Dream Labs or Anki. "Cozmo" is a trademark of its owners. It drives a physical robot, so use it at your own risk, on the floor, away from edges.

## What it does

- **Roams freely** using Cozmo's own freeplay behaviors, for as long as the connection holds.
- **Looks after its battery:** when the battery is low he drives to his charger, charges, then goes back to playing.
- **Voice and typed commands** (see the table below), including picking up, stacking and knocking over cubes.
- **AI conversation:** say "Hey Cozmo" (or just type) and chat. Replies come from an LLM via [Groq](https://console.groq.com)'s free tier, spoken in Cozmo's own voice with a matching emotion animation.
- **Back to roaming 15 seconds after your last command** (configurable). Say or type `stop` at any time.

## Requirements

- Cozmo, his charger and cubes
- An iPhone/iPad with the official Cozmo app, and a USB cable. Tested on iOS with app version **3.6.0**. Android is untested.
- A computer with **Python 3.9**. Tested on macOS (Python 3.9.13).
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

Type them in the terminal, or say them (start with "Hey Cozmo", or just say the simple ones).

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

Everything you might want to change is at the top of `autonomous.py`: the roam delay, battery thresholds, charging time, knock-over lift height, and the Groq model.

## How it connects to the current app

The old Anki SDK refuses to connect to newer app versions because of a version check. The script gets past it by importing `cozmo` first and then patching two values in `cozmoclad` (see the top of `autonomous.py`). Nothing on the robot is modified.

## Troubleshooting

- **`python3.9: command not found`**: install Python 3.9 (Installation step 2).
- **`pip: command not found`**: the virtual environment isn't active. Run `source venv/bin/activate`.
- **"Hmm, my brain is a bit fuzzy"**: the AI call failed. Read the `LLM error` line above it in the terminal. If it says `model_not_found`, Groq has retired the model: list the current ones with `curl -s https://api.groq.com/openai/v1/models -H "Authorization: Bearer $(cat groq_key.txt)"` and change `GROQ_MODEL` at the top of the script.
- **He ignores spoken commands:** check the `Heard:` lines in the terminal, and that the microphone is allowed (System Settings -> Privacy & Security -> Microphone).
- **He can't find his charger:** use good light, a clear floor, and face the charger's black-and-white marker into the room.
- **Disconnects:** the script can't turn SDK mode back on for you. Keep the phone unlocked with the app on screen, and keep Cozmo charged.
- **Battery thresholds are estimates.** The script prints the voltage every 15 seconds so you can tune `LOW_BATTERY_VOLTS` and `CHARGE_MINUTES`.

## Privacy

Speech is turned into text on your computer. Only that text is sent to Groq to generate replies. No audio leaves your machine.

## Credits

- Built on the [Anki Cozmo Python SDK](https://github.com/anki/cozmo-python-sdk), [Vosk](https://alphacephei.com/vosk/) for offline speech recognition, and [Groq](https://groq.com) for the AI replies.
- The version-check patch that lets the SDK connect to the current app comes from a community write-up. **(Add a link to it here.)**

## License

MIT. See the `LICENSE` file.
