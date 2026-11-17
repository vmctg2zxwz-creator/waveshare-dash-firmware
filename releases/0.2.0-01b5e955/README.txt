Waveshare KIT-B / 10.1-DSI-TOUCH-A / ESP32-P4 rev3.x / 32 MB ONLY
Firmware: 0.2.0-01b5e955
PQ35 CAN COMPARISON: NO simulated values. Listen-only, 500 kbit/s, TX GPIO2 / RX GPIO3.
Target CAN: Leon 1P Cupra 2.0 TFSI 2008 powertrain; compare with actual ECU/ABS data in Autel.
Documented profile: RPM 0x280, speed 0x1A0, coolant 0x288, intake air 0x380, battery 0x570.
The profile is not yet verified on the actual vehicle. Battery appears only if its frame is present.
Missing/faulty/stale (>1 second) readings show --. No guessed fallback values.
Fuel is phase 2. Oil and boost remain -- pending protocol/reference validation.
Swipe down > CAN / AUTEL shows readings, ages, raw bytes and receive/error counters.

FIRST INSTALL VIA USB: select dash-kitb-usb.bin at address 0x2000.
This installs an OTA partition table and replaces the existing firmware.
Use Flash Download Tool, ESP32-P4, COM8, 115200, DoNotChgBin if available.
Hold BOOT, press/release RESET, release BOOT, then START. Do not erase all.
After completion press RESET. The merged USB image resets old NVS settings.

WI-FI: swipe down > Wi-Fi. Scan/select your 2.4 GHz network and CONNECT.
Swipe down > CHECK UPDATE installs the public GitHub version over the saved Wi-Fi.
Existing OTA installations retain Wi-Fi settings. CAN pauses and queued samples are discarded for flash writes.
The optional phone setup page at http://192.168.4.1 has Download CAN data for a bounded JSON snapshot.
Alternatively upload dash-kitb-ota.bin on the local browser page.
NEVER upload dash-kitb-usb.bin to the OTA form.
Subsequent OTA updates retain Wi-Fi settings. Keep power connected.
Software tests/build are not physical CAN, Autel calibration, Wi-Fi/C6 or OTA hardware validation.

Protocol definitions: commaai/opendbc vw_pq.dbc, commit 5cb4ca34c7abb2a976fb835ba9fca6bf4386ed4c.
https://github.com/commaai/opendbc/blob/5cb4ca34c7abb2a976fb835ba9fca6bf4386ed4c/opendbc/dbc/vw_pq.dbc

Copyright (c) 2020, Comma.ai, Inc.

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
