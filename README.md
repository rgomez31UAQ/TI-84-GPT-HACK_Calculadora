# TI-84

A hardware mod that gives your TI-84 calculator internet access, ChatGPT integration, and more.

![built pcb](./pcb/built.png)

## About

This project turns your dusty graphing calculator into a WiFi-enabled device capable of querying ChatGPT, downloading programs over the air, and chatting with other modded calculators.

The original concept came from ChromaLock, who made a [video about it](https://www.youtube.com/watch?v=Bicjxl4EcJg) back in 2024. His repo got nuked, so I rebuilt the whole thing from scratch based on what I could piece together. Most of the code, PCB design, and implementation is my own work at this point.

**Author:** Andy (ChinesePrince07)

## Features

- ChatGPT integration - ask questions directly from your calculator
- Wi-Fi connectivity via ESP32 with **captive portal configuration**
- Program downloads over the air
- Image display support (96x63 monochrome)
- Chat functionality
- **Pre-configured server** - no need to run your own!

## How It Works

The TI-84 has a 2.5mm link port originally designed for calculator-to-calculator communication using Texas Instruments' proprietary link protocol. This mod exploits that by wiring an ESP32 microcontroller to impersonate another calculator.

When your calculator sends a variable (like a command number), the ESP32 intercepts it using the CBL2 library, which reverse-engineers TI's protocol. The ESP32 then performs the actual work (WiFi requests, API calls, etc.) and sends the results back as calculator variables (strings, numbers, or even pictures).

The TI-BASIC launcher program on your calculator provides the UI - it sends commands to the ESP32, waits for responses, and displays the results. From the calculator's perspective, it's just talking to another calculator. It has no idea there's WiFi involved.

## Hardware Required

- TI-84 Plus Silver Edition or TI-84 Plus C Silver Edition
- Seeeduino XIAO ESP32-C3 (or ESP32-S3 Sense for camera support)
- Custom PCB (gerber files in `/pcb/0.1/`)
- 4x 1kΩ resistors
- 2x N-channel MOSFETs (MMBF170 or similar)
- Magnet wire (30-34 AWG recommended)
- Soldering iron with fine tip
- Patience

## Hardware Assembly

**Warning:** This will void your warranty and possibly destroy your calculator if done wrong.

### Understanding the Circuit

The calculator's link port uses TIP and RING signals (like a headphone jack) at 3.3V logic levels - except when it doesn't. The ESP32 also runs at 3.3V, but the two don't play nice directly due to timing and drive strength issues. The MOSFETs act as bidirectional level shifters, cleaning up the signals.

### PCB Assembly

1. Order the PCB using the gerber files in `/pcb/0.1/`. Any cheap fab house works (JLCPCB, PCBWay, etc.). 1.6mm thickness, any color.

2. Solder the components in this order:
   - MOSFETs first (they're the smallest and most annoying)
   - 1kΩ resistors
   - Pin headers for the ESP32 (or solder it directly if you hate yourself)

3. The silkscreen labels show: 5V, GND, TIP, RING. These connect to the calculator.

### Calculator Surgery

1. Remove the 6 screws from the back of your TI-84. Keep track of which ones go where - they're different lengths.

2. Carefully separate the case halves. There's a ribbon cable connecting them - don't yank it.

3. Locate the link port on the PCB. It's the 2.5mm jack at the top. You need to solder wires to the TIP and RING pads on the calculator's mainboard. These are usually labeled or you can trace them from the jack.

4. Find 5V and GND. The battery terminals are an easy source. 5V is the positive terminal of the battery pack, GND is negative.

5. Route the wires to wherever you're mounting the ESP32 PCB. Most people tuck it behind the screen or in the battery compartment (requires removing a battery or using a slimmer pack).

6. Connect:
   - Calculator TIP → PCB TIP
   - Calculator RING → PCB RING
   - Calculator 5V → PCB 5V
   - Calculator GND → PCB GND

7. Triple-check your connections. Reversed polarity will fry the ESP32. Wrong TIP/RING will just not work.

8. Reassemble and pray.

### Testing

Power on the calculator. If it still works, good. If the ESP32's LED blinks, even better. Connect to the "calc" WiFi network and configure your credentials. Run the launcher program and try the connect command.

## Software Setup

### 1. ESP32 Setup
1. Open `/esp32/esp32.ino` in Arduino IDE
2. Install required libraries: TICL, CBL2, TIVar, WiFi, HTTPClient, UrlEncode, Preferences
3. Flash the code to your ESP32
4. No configuration files needed!

### 2. WiFi Configuration (Captive Portal)
1. Power on your calculator
2. Connect to the WiFi network named **"calc"** from your phone/computer
3. A captive portal will automatically open (or navigate to `192.168.4.1`)
4. Enter your WiFi name and password
5. Click "Save & Connect"
6. The ESP32 will connect to your network and remember the settings

The server is pre-configured - no need to run your own!

### 3. Calculator Setup
Transfer the LAUNCHER program to your calculator using the TI-84 menu (Settings → Update).

## Reconfiguring WiFi

If you need to change WiFi settings:
- The captive portal is available for a few seconds on every boot
- Or erase ESP32 flash and re-upload the firmware

## Commands

| ID | Command | Description |
|----|---------|-------------|
| 0 | connect | Connect to configured WiFi |
| 1 | disconnect | Disconnect from WiFi |
| 2 | gpt | Query ChatGPT |
| 5 | launcher | Transfer launcher program |
| 9 | image_list | List available images |
| 10 | fetch_image | Download image to calculator |
| 11 | fetch_chats | Get chat messages |
| 12 | send_chat | Send chat message |
| 13 | program_list | List available programs |
| 14 | fetch_program | Download program |

## Troubleshooting

- **ESP32 not responding:** Check your TIP/RING connections. They might be swapped.
- **WiFi won't connect:** Make sure you're connecting to a 2.4GHz network. The ESP32-C3 doesn't support 5GHz.
- **ChatGPT returns garbage:** The response might contain characters the calculator can't display. Working on it.
- **Calculator freezes:** The link protocol is timing-sensitive. Try again, or power cycle both devices.

## Planned Features

- Support for color images
- Multi-page GPT responses
- Chat history
- Basic web browsing
- Weather integration
- Camera support (ESP32-S3)

## Known Issues

- Images don't work consistently
- App transfer occasionally fails

## Credits

- Original concept by [ChromaLock](https://www.youtube.com/watch?v=Bicjxl4EcJg) (RIP his repo)
- Everything else by Andy

## License

GPL v3 - See [LICENSE](LICENSE) for details.
