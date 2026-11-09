Waveshare KIT-B / 10.1-DSI-TOUCH-A / ESP32-P4 rev3.x / 32 MB ONLY
Firmware: 0.1.0-87ef4d4d
DEMO: simulated values, no CAN/vehicle connection.

FIRST INSTALL VIA USB: select dash-kitb-usb.bin at address 0x2000.
This installs an OTA partition table and replaces the existing firmware.
Use Flash Download Tool, ESP32-P4, COM8, 115200, DoNotChgBin if available.
Hold BOOT, press/release RESET, release BOOT, then START. Do not erase all.
After completion press RESET. The merged USB image resets old NVS settings.

WI-FI: tap Wi-Fi / SETUP. Join the network with the password on the display.
Open http://192.168.4.1 and enter your Internet Wi-Fi or hotspot settings.
Use CHECK UPDATE on the display or browser to install the public GitHub version.
Alternatively upload dash-kitb-ota.bin on the local browser page.
NEVER upload dash-kitb-usb.bin to the OTA form.
Subsequent OTA updates retain Wi-Fi settings. Keep power connected.
Build validation does not prove Wi-Fi/C6 compatibility, orientation/touch or OTA hardware behavior.
