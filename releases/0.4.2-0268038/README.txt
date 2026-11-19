Waveshare KIT-B / 10.1-DSI-TOUCH-A / ESP32-P4 rev3.x / 32 MB ONLY
Firmware: 0.4.2-0268038
PQ35 CAN DIAGNOSTICS: NO simulated values. 500 kbit/s, TX GPIO2 / RX GPIO3.
Passive/listen-only CAN reception starts automatically at boot. ECU requests remain OFF.
Swipe down > START CAN starts passive/listen-only reception; PAUSE CAN stops/uninstalls the receiver.
Keep H/L connected for this software isolation test. Start/pause does not enable ECU polling.
A high bus-error rate latches reception OFF: at least 128 errors and at least 1280 errors/second.
Worker samples every 20 ms over 100 ms windows; stopping is subject to actual task/ISR scheduling.
The error latch never retries automatically. Inspect the message; START CAN explicitly retries in listen-only.
A manual/error pause survives OTA completion. A restart starts passive CAN reception.
Swipe left/right between the round dashboard and 24-field diagnostics.
READ ECU is OFF at every boot and must be enabled explicitly on diagnostics.
When enabled: engine KWP2000 over VW TP2.0, logical address 01, setup 0x200/0x201; data IDs negotiated.
KWP standard diagnostic session 0x89, then readDataByLocalIdentifier 0x21, max 5 block requests/second.
No standard Mode 01 polling fallback, clearing faults, coding, actuator tests, programming or security access.
Switch OFF before using Autel. Reads stop on a page swipe, settings opening or an update.
No compatible response: finite timeout, then return to listen-only. Channel and CAN API failures are distinguished.
Target CAN: Leon 1P Cupra 2.0 TFSI 2008 powertrain; compare with actual ECU/ABS data in Autel.
Documented profile: RPM 0x280, speed 0x1A0, coolant 0x288, intake air 0x380, battery 0x570.
RPM, ABS speed, coolant and IAT match the S3 code verified by the user against Autel.
The P4 installation and its reported flashing remain separately unverified. Battery is not vehicle-verified.
Missing/faulty/stale readings show --. CAN timeout 1s; fast ECU values 2.5s, slow ECU values 10s, with sample ages.
User-defined blocks: lambda control 001.3, voltage 004.2, lambda actual 031.1, alternator load 053.4,
fuel supply pressure 103.1, fuel rail pressure 106.1, boost pressure as reported by ECU 115.4.
Each block has four values. Each selected cell must have a supported ECU-supplied formula and matching units.
Pressure is normalized to bar. 115.4 is not converted to gauge boost by subtracting an assumed 1 bar.
Oil 134.4 is NOT polled: sensor is not fitted. It remains --. Other unbound/unavailable values also remain --.
Fuel LEVEL is phase 2. Main dashboard oil and boost stay -- pending their broadcast/reference validation.
The reported brightness surge/freeze is unresolved; this feature release does not claim a hardware stability fix.
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

Protocol and numeric formula references. The bounded C++ implementation is written for this project.

Reference: https://github.com/I-CAN-hack/pq-flasher/tree/95d283075714c9476cacc6ef041fd810abc86f8a

MIT License

Copyright (c) 2021 Willem Melching

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.


Reference: https://github.com/gmenounos/kw1281test/tree/6b3567f68db5aa8ce205d43694ad5decf3db787d

MIT License

Copyright © 2021 Greg Menounos

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

Reference: https://github.com/fazyamir/KLineScan/tree/251490c6ac425055eb323c2446c4722b00ecc184

MIT License

Copyright (c) 2026 fazyamir

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

