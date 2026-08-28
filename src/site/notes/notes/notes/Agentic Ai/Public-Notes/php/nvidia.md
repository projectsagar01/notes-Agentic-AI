---
{"dg-publish":true,"permalink":"/notes/notes/agentic-ai/public-notes/php/nvidia/","dg-note-properties":{}}
---

# RTX Broadcast for Linux

A lightweight, fully functional GPU-accelerated alternative to NVIDIA Broadcast for Linux. Built with Python, ONNX Runtime (CUDA), and `v4l2loopback`, this tool provides real-time AI background replacement for applications like Discord, OBS, and web browsers.

## 🚀 Features & Architecture
* **Real-Time AI Matting:** Utilizes the Robust Video Matting (RVM) model via ONNX Runtime to process webcam feeds directly on RTX CUDA cores.
* **Dynamic Modes:** Switch instantly between Cinematic Blur, Custom Image Background, Solid Green (for OBS Chroma Key), and Normal Camera bypass.
* **Floating UI:** A sleek Tkinter-based control panel that avoids GNOME system tray limitations and automatically spawns the backend AI engine.
* **Virtual Camera Output:** Streams the final processed frames securely to `/dev/video10` using the `v4l2loopback` kernel module.

## 🛠️ Installation & Setup
* Install system kernel modules: `sudo apt install v4l2loopback-dkms v4l2-utils python3-tk`
* Create and activate a Python 3.14 virtual environment: `python3.14 -m venv venv` and `source venv/bin/activate`
* Install required Python dependencies: `pip install opencv-python onnxruntime-gpu pyfakewebcam requests`
* Verify your physical webcam index (e.g., `/dev/video2`) using `ls -l /dev/video*` and update the `INPUT_CAM` variable inside `rtx_broadcast.py`.

## 🎮 How to Run
* Initialize the virtual camera bridge: `sudo modprobe v4l2loopback devices=1 video_nr=10 card_label="RTX_Virtual_Cam" exclusive_caps=1`
* Launch the GUI control panel: `python gui.py`
* Open Discord, OBS, or your browser and select `RTX_Virtual_Cam` from the camera dropdown to view the live processed feed.

## 🔧 Troubleshooting
* **Discord doesn't detect the camera:** Ensure you strictly included the `exclusive_caps=1` flag during the `modprobe` initialization step.

--------------------------
# RTX Broadcast for Linux

A lightweight, fully functional GPU-accelerated virtual camera for Linux. This tool provides real-time AI background replacement (Blur, Custom Image, Solid Green Chroma Key) for applications like OBS, Discord, and web browsers, acting as an open-source alternative to NVIDIA Broadcast.

## 📦 Required Libraries & Dependencies

This project relies on a mix of system-level kernel modules and Python packages to achieve low-latency video processing.

*   **`onnxruntime-gpu`**: The core AI inference engine. It executes the Robust Video Matting (RVM) model directly on NVIDIA CUDA cores for real-time performance.
*   **`opencv-python` & `numpy`**: Used for capturing hardware webcam feeds, performing matrix operations on alpha channels, applying Gaussian blurs, and blending the foreground with custom backgrounds.
*   **`pyfakewebcam`**: A crucial bridge that takes the final processed RGB arrays from OpenCV and writes them directly to the Linux virtual camera device (`/dev/video10`).
*   **`requests`**: Automates the initial setup by downloading the necessary RVM `.onnx` AI model directly from the release servers if it is not found locally.
*   **System Packages**: Requires `v4l2loopback-dkms` for creating the virtual `/dev/video*` pipeline, and `python3-tk` to render the floating desktop control panel.

## 🚀 Installation & Setup

1.  **System Preparation:** Install the required kernel modules and GUI framework using your package manager (e.g., `sudo apt install v4l2loopback-dkms python3-tk v4l2-utils`).
2.  **Environment Setup:** Create a Python virtual environment and install the dependencies via `pip install opencv-python onnxruntime-gpu pyfakewebcam requests`.
3.  **Kernel Initialization:** You must load the virtual camera driver with specific flags so apps like Discord can detect it. Run: `sudo modprobe v4l2loopback devices=1 video_nr=10 card_label="RTX_Virtual_Cam" exclusive_caps=1`.

## 🧠 Architecture & Usage

The application uses a decoupled architecture. The frontend GUI and the backend AI engine run as separate processes and communicate seamlessly via a persistent `config.json` file.

*   **Launching the Engine:** Simply run `python gui.py`. The interface will automatically detect if the CUDA backend is offline and spawn it securely in the background.
*   **Camera Configuration:** By default, the engine captures from `/dev/video0`. If your system maps the integrated webcam differently, use `v4l2-ctl --list-devices` to find the correct index and update the `INPUT_CAM` variable in the backend script.
*   **Using the Output:** Open your streaming or conferencing application, navigate to the video settings, and select **RTX_Virtual_Cam**. Use the floating control panel to instantly switch backgrounds.