# Raah — a walking companion for blind and low-vision people

Raah turns any phone into a talking guide for Indian roads. Point the back camera forward, and Raah tells you what is in your path, which side it is on and how close it is. It speaks in English, Hindi or Tamil, and gives a vibration pattern for each direction.

> "Car on your left, very close." · "रुकिए! बाइक सामने, बहुत पास" · "கவனம். முன்னால் பள்ளம் இருக்கலாம்."

Everything runs **on the phone**. No video leaves the device, no server is needed and no API key is needed.

## Features

| Feature | How it works |
|---|---|
| Object detection | TensorFlow.js + COCO-SSD (MobileNet v2 lite), in the browser, about 5–15 fps on a mid-range phone |
| Direction | Frame split into three lanes: left / ahead / right |
| Distance | Box size relative to the frame: far / near / very close |
| "Coming closer" | Tracks each object across frames and flags it when its box grows quickly |
| Priority | Vehicles beat animals, animals beat people, and "ahead" beats the sides. Only the single most dangerous thing is spoken, so the user isn't flooded |
| STOP alert | A vehicle very close and straight ahead interrupts any speech with "Stop!" |
| Ground scan (beta) | Flags a dark, high-texture patch in the walking lane (pothole, open drain, step down) |
| Haptics | Left = one long buzz, Right = two short buzzes, Ahead = three pulses. Stronger when very close |
| Describe | One tap reads out everything around you: "Ahead: a car, 2 people. Left: a cow." |
| Screen-reader mode | Turn off Raah's voice and TalkBack or VoiceOver reads the alerts instead |
| Repeat | Double-tap the camera view to hear the last alert again |
| Demo video | Load any road video, so you can demo indoors on a laptop |

Things it detects: car, bike, bus, truck, cycle, cow, horse, dog, cat, person, bench, chair, plant pot, hydrant, traffic signal, stop sign.

## Run it

The camera only works on **https** or **localhost**.

**On your laptop**

```bash
cd raah
python -m http.server 8000
# open http://localhost:8000
```

**On your phone (needed for the real demo): deploy to GitHub Pages**

1. Create a new repo on GitHub, for example `raah`, and upload `index.html` and `README.md` (already done: this repo).
2. Go to **Settings → Pages → Branch: main → / (root) → Save**.
3. After a minute, open `https://icebergabhay1111-a11y.github.io/RAAH-2.0/` on your phone in Chrome.
4. Tap **Start** and allow the camera.

Vercel also works: drag the `raah` folder onto vercel.com/new.

## 2-minute demo script for judges

1. **Problem (20 s).** India has the world's largest blind population, and our footpaths have potholes, open drains, parked bikes and stray cows. A white cane only reaches about one metre and cannot tell you a bike is coming.
2. **Live demo (60 s).** Turn on English and close your eyes or blindfold a teammate. Walk toward a chair, then have a teammate walk across from the left. Switch to Hindi or Tamil mid-demo, which always gets a reaction. Tap **Describe**. Keep the boxes on so judges can see what the model sees.
3. **Backup.** If the room is bad, open **Settings → Demo video** with a road clip recorded on your phone beforehand.
4. **Why it wins (20 s).** It runs on a ₹8,000 phone, works offline once loaded, needs no hardware, keeps video private, and speaks Indian languages.
5. **Roadmap (20 s).** Train a custom YOLO model on Indian pothole, drain and auto-rickshaw datasets, add currency-note reading, offer a bone-conduction earphone mode, and add an SOS location share.

Record a backup video of the demo working the night before.

## Honest limitations (say these before judges ask)

- Distance is estimated from box size, not measured. A depth model (MiDaS) or a phone with LiDAR would improve it.
- COCO-SSD has no auto-rickshaw class. It is usually detected as a car or truck, which is still announced as a vehicle.
- Ground scan is a simple brightness and texture rule. It can false-alarm on shadows. A trained pothole model is the next step.
- Phone speech voices for Tamil and Hindi depend on the device. Most Android phones have Google's voices installed.

## Ideas to extend during the hackathon

- **Custom pothole model.** Train on a public Indian pothole dataset with Teachable Machine or YOLOv8, export to TF.js, and swap it into `groundScan()`.
- **Currency notes.** Use a Teachable Machine image model with ₹10–₹500 classes.
- **SOS.** Long-press to share live location with a family member.
- **PWA.** Add a manifest and service worker so it installs to the home screen and works fully offline.

## Code map (`index.html`)

- `OBJ`: which objects matter and how dangerous each is
- `L10N`: all spoken phrases in English, Hindi and Tamil
- `updateTracks()`: matches detections frame to frame
- `analyze()`: works out direction, distance and approach for each object
- `decide()`: picks the one thing to say, with cooldowns
- `groundScan()`: pothole heuristic
- `draw()` / `updateLanes()`: visuals for sighted helpers
