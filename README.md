WiFi Kitchen Timer — Wemos D1 + TM1637 + Buzzer
A WiFi-controlled countdown timer built around the Wemos D1 Mini (ESP8266). Set the time from any browser on your local network, watch the countdown live on a 4-digit TM1637 display, and get an audible alarm from a buzzer when time runs out. No buttons, no remote — just a phone or computer and a web browser.

What It Does
Set time remotely — enter minutes and seconds on a simple web page served by the ESP8266 itself.

Live countdown display — the TM1637 shows MM:SS, with a blinking colon while running.

Start / Pause / Reset — full control from the browser.

End-of-timer alarm — display flashes and the buzzer sounds a repeating beep pattern for 10 seconds.

Built-in buzzer tester — a sweep from 500 Hz to 4 kHz helps you find the loudest tone for your specific buzzer.

Non-blocking design — the countdown uses millis() rather than delay(), so the web page stays responsive the whole time.

Why This Project
Most kitchen timers are either dumb (wind-up, no remote control) or overkill (smart speaker integrations). This one is the middle ground: a standalone gadget that lives on the counter, controlled from a phone, with no cloud, no app, and no account. Everything runs locally on your WiFi network.

Hardware
Part	Notes
Wemos D1 Mini (ESP8266)	Any ESP8266 board works
TM1637 4-digit display	Standard 0.36" or 0.56" module
Passive buzzer	~2–2.7 kHz resonant; active buzzers also supported
Breadboard + jumper wires	
Micro-USB cable	Data cable, not charge-only
Optional for louder sound: 2N2222 or BC547 NPN transistor + 1kΩ resistor.

Wiring
Module	Wemos D1 Pin
TM1637 CLK	D1 (GPIO5)
TM1637 DIO	D2 (GPIO4)
TM1637 VCC	3.3V or 5V
TM1637 GND	GND
Buzzer (+)	D5 (GPIO14)
Buzzer (−)	GND
Software
Two libraries are required, both available via the Arduino IDE Library Manager:

TM1637 by Avishorp

ESP8266 board package (Boards Manager URL from the ESP8266 GitHub)

Before flashing, set your WiFi credentials at the top of the sketch:

cpp
const char* WIFI_SSID = "YOUR_WIFI_NAME";
const char* WIFI_PASS = "YOUR_WIFI_PASSWORD";
After boot, open the Serial Monitor at 115200 baud to see the assigned IP address. The display briefly flashes the last digit of the IP as a hint. Then open http://<ip>/ in any browser on the same network.

How It Works
The sketch runs a small HTTP server on port 80 with five endpoints:

/ — serves the control page (HTML + JavaScript stored in PROGMEM)

/set?min=X&sec=Y — sets the countdown duration

/cmd?action=start|pause|reset — controls the timer state

/status — returns JSON with the current state and remaining seconds

/beep?freq=X&ms=Y — drives the buzzer for the tester

The main loop runs a four-state machine (Idle → Running → Paused → Done) and uses millis() timestamps instead of blocking delays, so all network requests are handled promptly during the countdown.

Buzzer Volume
A GPIO pin on the ESP8266 can only supply a few milliamps, which is often too little for a passive buzzer to sound loud. Two fixes are built in:

Software tuning — the on-page buzzer tester helps you find the resonant frequency (usually 2000–2700 Hz for small modules). Set the loudest one as buzzFreq in the sketch.

Hardware boost — add an NPN transistor as a low-side switch (buzzer + → 5V, buzzer − → collector, emitter → GND, base → D5 through 1kΩ). This roughly doubles the loudness.

Customization
Quick presets — add buttons for 3/5/10 minutes calling /set then /cmd?action=start.

+1 minute during cooking — add remainingMs += 60000; in handleCmd() for a live extension.

mDNS — call MDNS.begin("kitchen-timer") in setup() to reach it at http://kitchen-timer.local.

Quiet mode — add a flag that skips the buzzer for late-night use.

Persistent last-set time — save totalMs to LittleFS or EEPROM and reload at boot.
