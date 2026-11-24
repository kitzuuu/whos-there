# Who's There? Real-time intruder detection with AI and IoT

Ordinary motion sensors raise false alarms for pets, shadows and swaying trees. This system runs on a Raspberry Pi 5: a PIR motion sensor wakes up a YOLO11n person detector on the Pi camera, and only a confident detection of a person counts as an intrusion. You then get a short GIF of the event on Telegram.

Built for the Internet of Things course at Maastricht University (BSc Computer Science, 2025) by a team of seven.

## How it works

1. **Motion trigger.** A PIR sensor on GPIO 21 (read with `gpiozero`) opens a 10-second detection window. Between triggers, the camera frames are not run through the model, which saves energy.
2. **Person detection.** YOLO11n, exported to NCNN so it runs on the Pi's CPU, checks each frame. Detections below 0.5 confidence are ignored, which cuts false alarms.
3. **Alert.** On a detection the app saves the first frames as JPGs, builds a GIF and sends it to a Telegram chat. A cooldown stops repeated alerts for the same event.
4. **Dashboard.** A Streamlit page shows the live feed, the system status and recent detections, and logs CPU and RAM use to a CSV file for the energy experiments.

## Results

We compared the PIR + camera system with a motion sensor on its own in ten test situations, in daylight and at night. Adding the camera and YOLO removed the false alarms in both conditions, but missed more real events, so the confidence threshold trades some missed detections for far fewer false alarms. We also measured CPU, temperature and RAM: CPU use stays low while the system waits for motion and rises sharply during detection, while RAM use goes up as soon as the model is loaded.

## Running it

You need a Raspberry Pi 5 with the Pi camera and a PIR sensor on GPIO 21.

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
cp config.example.json config.json   # add your Telegram bot token and chat id, or set them in the app
streamlit run app.py
```

`camera_test.py` opens a plain camera preview to check the camera before running the app.

## Project structure

| File | What it does |
|---|---|
| `app.py` | Streamlit app: camera loop, detection window, alerts, detection history, resource logging |
| `raspberry_motion_detector.py` | PIR sensor and the detection window |
| `raspberry_object_detector.py` | Loads the NCNN YOLO11n model, runs detection and draws boxes |
| `telegram_notifier.py` | Sends GIF alerts through the Telegram Bot API, with a cooldown |
| `utils.py` | Saves frames as JPGs and builds GIFs |
| `yolo11n_ncnn_model/` | YOLO11n weights exported to NCNN |

## Team

Stefani Parvina, Melodie Prudhomme, John Fourlas, Deniz Derviş, Raman Yousefi Avarzaman, Tadiwanashe Matara and Cristian Niţu.

My part: I was one of the two main developers. I built the object-detection module (`raspberry_object_detector.py`: loading the NCNN-exported YOLO11n model, safe weights-only loading, detection with confidence filtering), co-wrote the motion-sensor integration that runs YOLO only after motion, wrote the energy monitoring used in our experiments, and set up the Raspberry Pi.
