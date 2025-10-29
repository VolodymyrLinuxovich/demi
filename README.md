# DEMI - Deodorizing Engineering Majors Intelligently

[▶️ Watch the demo](https://youtu.be/5mRBLSm8bQI)
[🧩 Devpost - Project](https://devpost.com/software/demi-deodorizing-engineering-majors-intelligently)

## Problem
When you walk into a hackathon, what hits you first? <br> **The smell** (and a whole buffet of other aromas).
Sleep debt + crowded rooms + warm labs = VOC spikes. Manual deodorant is easy to forget and hard to time.
We need a low-cost, safe, and configurable way to **detect** odor indicators and **act** immediately, without constant monitoring.

## What is DEMI?
DEMI is a wearable/clip-on prototype that **automatically applies deodorant** when VOC levels exceed a configurable threshold.
It combines:
<br>• a TVOC sensor to quantify odor indicators;
<br>• an optional front-facing camera for context/heuristics;
<br>• a servo actuator that presses a standard aerosol can.

The system smooths noisy readings (moving average), decides with threshold + cooldown, and triggers a safe actuation trajectory.
Everything is configurable; a dry-run mode lets you test logic without spraying.

## Tech Stack & What’s Included
• Language: Python 3.10+
<br>• Core libs: Sensor I/O & GPIO (Pi), opencv-python (camera),
  fastapi + uvicorn (local control), requests + paramiko (HTTP/SSH), transformers + torch (optional depth)
<br>• Repository layout:

  - main.py
  - requirements.txt
  - README.md
  - __init__.py
  - .gitignore
  - .DS_Store
  - control/
    - run_trajectory.py
    - teleop.py
    - record_dataset.sh
    - replay_trajectory.sh
  - cv/
    - cv_pipeline.py
    - depth_estimator.py
    - depth_mask_strategies.py
    - image_segmenter.py
    - mask_intersection.py
    - run_offensive_cv.py
    - utils.py
    - temp/
  - hardware/
    - actuator.py
    - camera.py
    - filter.py
    - main.py
    - sensors.py
  - server/
    - collect_pi_data.py

## Quick Start
1) Clone & environment
- git clone https://github.com/VolodymyrLinuxovich/demi.git
- cd demi
- python -m venv .venv
  - Linux/macOS: source .venv/bin/activate
  - Windows: .venv\Scripts\activate
- pip install -r requirements.txt
- (If large binaries are tracked) git lfs install

2) Run (choose what you need)
- Top-level main loop:
  - python -u main.py
- Embedded hardware integration:
  - python -u hardware/main.py
- Local control server (if FastAPI present in server/):
  - uvicorn server.main:app --host 0.0.0.0 --port 8088   (adjust module path if different)
  - or: python -u server/main.py                         (if the module exposes __main__)
- Computer-vision helpers:
  - python -u cv/run_offensive_cv.py
  - python -u cv/cv_pipeline.py
- Trajectory tools:
  - python -u control/run_trajectory.py
  - bash control/record_dataset.sh
  - bash control/replay_trajectory.sh

3) Minimal config (example)
Create a .config.json in the repo root if you need remote JSONL logging via SSH:
{
  "MAC_IP": "192.168.x.y",
  "MAC_USER": "your-username",
  "MAC_REMOTE_PATH": "/Users/your-username/demi_logs/stream.jsonl"
}

4) Useful endpoints (if server provides them; adjust to your app)
- Switch filter strategy:
  curl -X POST http://127.0.0.1:8088/set_filter_mode -H 'Content-Type: application/json' -d '{"mode":"ma15"}'
- Set tri-mix weights:
  curl -X POST http://127.0.0.1:8088/set_tri_weights -H 'Content-Type: application/json' -d '{"short":0.6,"mid":0.3,"long":0.1}'
- Stream controls:
  curl http://127.0.0.1:8088/stream/status
  curl -X POST http://127.0.0.1:8088/stream/start
  curl -X POST http://127.0.0.1:8088/stream/stop

5) Optional environment variables (if you use vision/LLM/controller)
- export ANTHROPIC_API_KEY=sk-...
- export CONTROLLER_BASE_URL=http://localhost:8000
- export CONTROLLER_TIMEOUT=2.0

## How It Works
1) Sense - poll TVOC; optionally capture camera frames; smooth via moving average.
2) Decide - threshold + debounce + cooldown to avoid chatter/over-spray.
3) Actuate - drive servo/arm along a safe trajectory to press the can.
4) Log - append timestamped JSON lines (VOC, decision, optional frame).
5) Control - if enabled, control via simple HTTP endpoints.

## Hardware (reference)
- Compute: Raspberry Pi (or MCU + serial bridge)
- Sensor: TVOC (SGP30 / CCS811 / compatible)
- Actuator: hobby servo or small robotic arm
- Optional: USB camera
- Mounts: backpack clip + can holder
Note: verify pinouts, current draw, and power rails against datasheets.

## Safety & Ethics
- Start with a dry run or a water can.
- Keep spray away from face; don’t use on others without explicit consent.
- Many deodorants are flammable - avoid flames and hot surfaces.
- Respect venue rules for aerosols and robots.

## Roadmap
- [ ] Humidity and temperature-aware VOC calibration
- [ ] Lightweight on-device ML for camera context
- [ ] Trajectory shaping (PID) for smoother actuation
- [ ] Local web dashboard (live charts + logs)

## Acknowledgments
- Original concept & implementation: ethanhharrison/demi - special thanks to **E Harrison**.

## Created by
- Eric Ji
- E Harrison
- Volodymyr Borysenko
- Mohamed Tarek Homsi
- Nick Bui

## License
See LICENSE. Mirrors the original project’s license.
