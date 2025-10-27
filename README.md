# USB_CAM_Crosshairs
Draws a bulls eye on a USB cam feed - linux - python


Thanks! Shifting gears to your USB microscope setup—great project for precise lathe tool centering.

### Pre-Made Utilities
After checking around (forums like Practical Machinist, Adafruit, Microforum, and GitHub), there's no off-the-shelf Linux utility that's microscope-specific and adds a dynamic bull's eye (concentric circles or crosshair) directly to the live USB feed out of the box. Tools like **guvcview** (a solid UVC viewer for USB cams/microscopes) or **Cheese** handle the feed fine but lack built-in overlays—there's an old feature request for crosshairs in guvcview, but it's unimplemented. 

A close-ish option is **CrossOver** (GitHub: lacymorrow/crossover), a lightweight screen overlay tool for adding a static crosshair to *any* window (including your cam viewer). It's not integrated (won't "move with" image shifts like pan/zoom), but it's dead simple:
- Install: `sudo apt update && sudo apt install libxdo-dev libx11-dev` (Ubuntu/Debian), then clone/build from GitHub or grab the binary.
- Run: Launch your viewer (e.g., `guvcview -d /dev/video0`), then `crossover --center` for a basic crosshair on top.
It's gaming-oriented but works for this—tweak config for bull's eye style (lines/circles via CSS-like options).

For something more tailored (e.g., measurement overlays), microscopy software like **ImageJ/Fiji** or **IC Capture** (from Imaging Source) supports USB cams with reticles, but they're heavier and Windows-leaning (Fiji runs on Linux but needs tweaks for live USB).

If those don't cut it, the best bet is a quick custom script—Python + OpenCV is standard, lightweight, and integrates the bull's eye *directly on the feed* so it scales/moves with the image. It runs at 30+ FPS on modest hardware.

### DIY Instructions: Python + OpenCV Script for Bull's Eye Overlay
This assumes Ubuntu/Debian (adapt for your distro, e.g., `dnf` on Fedora). Your microscope shows as `/dev/video0` (check with `ls /dev/video*` or `v4l2-ctl --list-devices`).

1. **Install Dependencies**:
   ```
   sudo apt update
   sudo apt install python3-pip python3-opencv v4l-utils
   ```
   - `python3-opencv`: Handles cam capture + drawing.
   - `v4l-utils`: For device listing/tweaks (optional).

2. **Test Your Cam**:
   - Run `guvcview -d /dev/video0` to confirm feed (install if needed: `sudo apt install guvcview`).
   - Note resolution/FPS (default 640x480@30 works; adjust in script if needed).

3. **The Script** (`bullseye_overlay.py`—save and run with `python3 bullseye_overlay.py`):
   ```python
   import cv2

   # Open USB microscope (change index if not 0; e.g., 1 for second device)
   cap = cv2.VideoCapture(0)
   cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)   # Adjust resolution as needed
   cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)
   cap.set(cv2.CAP_PROP_FPS, 30)

   if not cap.isOpened():
       print("Error: Could not open camera.")
       exit()

   print("Bull's eye overlay active. Press 'q' to quit.")

   while True:
       ret, frame = cap.read()
       if not ret:
           print("Error: Failed to grab frame.")
           break

       height, width = frame.shape[:2]
       center_x, center_y = width // 2, height // 2  # Dynamic center (moves with feed)

       # Draw bull's eye: Concentric circles (adjust radii/colors as needed)
       colors = [(0, 255, 0), (0, 255, 255), (0, 0, 255)]  # Green, yellow, red rings
       radii = [50, 30, 10]  # Outer to inner
       for i, radius in enumerate(radii):
           cv2.circle(frame, (center_x, center_y), radius, colors[i], 1)  # Thickness=1

       # Optional: Add crosshair lines for precision
       cv2.line(frame, (center_x - 20, center_y), (center_x + 20, center_y), (255, 255, 255), 1)
       cv2.line(frame, (center_x, center_y - 20), (center_x, center_y + 20), (255, 255, 255), 1)

       # Display the overlaid feed
       cv2.imshow('USB Microscope with Bull\'s Eye', frame)

       # Quit on 'q'
       if cv2.waitKey(1) & 0xFF == ord('q'):
           break

   cap.release()
   cv2.destroyAllWindows()
   ```

4. **Run & Tweak**:
   - Launch: `python3 bullseye_overlay.py`
   - It captures live, draws a green-yellow-red bull's eye + white crosshair at the image center (auto-adjusts if you resize or the feed shifts).
   - Customize:
     - Radii/colors: Edit the lists for bigger/smaller rings or hues.
     - Position: Hardcode `(center_x, center_y)` if you want off-center.
     - Save frames: Add `cv2.imwrite('snapshot.jpg', frame)` on keypress (e.g., 's').
     - For lathe use: Run on a secondary monitor; low CPU hit (~10-20%).
   - Troubleshooting:
     - Wrong device? Change `0` to `1` in `VideoCapture(0)`.
     - Blurry/low FPS? Tweak props or microscope focus.
     - Permissions: Add your user to `video` group (`sudo usermod -aG video $USER`; log out/in).

This'll give you a rock-solid, real-time overlay for spotting tool runout—way better than static stickers. If your distro's quirky (e.g., Arch) or you hit snags (e.g., cam index), drop details for fixes. What's the microscope model—any quirks?
