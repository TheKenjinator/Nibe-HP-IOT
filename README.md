# Nibe-HP-IOT
Repository to setup logging from Nibe HP through modbus
- ![Dashboard example](https://github.com/kejes/Nibe-HP-IOT/blob/main/Docs/dashboard.jpg?raw=true)

## Hardware
- Nibe Heatpump works confirmed on F1155 & F1255
- Home Assistant server
- Lilygo TCAN485 ESP32 €14 [link](https://nl.aliexpress.com/item/1005003624034092.html?spm=a2g0o.productlist.main.1.36be49e4pfs1Kd&algo_pvid=694caf32-608a-4161-9e8f-bfc9f61d5a26&algo_exp_id=694caf32-608a-4161-9e8f-bfc9f61d5a26-0&pdp_npi=4%40dis%21EUR%2113.95%2113.95%21%21%2114.87%2114.87%21%4021038e8317232122180598883e3e4d%2112000026545151020%21sea%21NL%21815979081%21X&curPageLogUid=iZmKj08lSeP2&utparam-url=scene%3Asearch%7Cquery_from%3A)

## Software
- Home Assistant
- ESPHome
- [Nibe Heatpump integration](https://www.home-assistant.io/integrations/nibe_heatpump)

# Installation steps

## Step 1: Flash the Lilygo ESP32
- I used the repo of [ESPHome-Nibe](https://github.com/elupus/esphome-nibe/tree/master) for the formatting and edited a few lines. See example file in this repo
- Howto for [installing ESPHome](https://www.youtube.com/watch?v=7PoUWszwaFk&t=1s)
  
## Step 2: Make a static IP for the NibePI
I logged in my router to select a fixed IP for the ESP board such that it won't give any problems if the ips will shuffle 

## Step 3: Wire the board to the heatpump
- See [wiring scheme](https://www.nibe.eu/nl-nl/installateur/schema-s-overzichten-en-technische-ondersteuning/elektrische-aansluitoverzichten)
- [Example](https://www.vanwerkhoven.org/blog/2023/nibe-heatpump-home-automation/)

   

## Step 4: Select Modbus option in your heatpump
- Go to the settings of your heatpump by holding the back button for 7 seconds. It should show a fifth icon called service. Go to menu 5.2.4 and activate modbus in the list of accessories. This will allow the heatpump to send data to the board.

## Step 5: Add the ESP32 to home assistant
- Same steps as you would normally add an ESP32 board to HA

## Step 6: Install the Nibe Heatpump integration
- Select the modbus option
- Activate the relevant entities of the 900+ total. I've added some example fields


# Update 28-07-26 after old Lilygo stopped working



Readme · MD
# NIBE F1155 → Home Assistant via ESPHome
 
Read data from a NIBE heat pump into Home Assistant using a LilyGo T-CAN485 (ESP32 + RS485) as a MODBUS40 gateway.
 
The ESP32 impersonates a NIBE MODBUS40 accessory on the pump's RS485 bus and forwards the traffic over UDP to Home Assistant, where the [Nibe Heat Pump](https://www.home-assistant.io/integrations/nibe_heatpump/) integration decodes it into ~990 entities.
 
Tested on an **F1155-6 PC**. Should work on any F-series pump that supports MODBUS40 (F1145, F1245, F1255, F750, F470, ...).
 
## Credits
 
- [elupus/esphome-nibe](https://github.com/elupus/esphome-nibe) — the ESPHome component doing the actual work
- [vanwerkhoven.org](https://www.vanwerkhoven.org/blog/2023/nibe-heatpump-home-automation/) — wiring reference and writeup
## Hardware
 
| Item | Notes |
|---|---|
| LilyGo T-CAN485 | ESP32 + RS485 transceiver, 5–12 V input, CH9102 USB-serial |
| 12 V supply | Can be taken from the pump's terminal block |
| Twisted pair | For the A/B RS485 run |
 
The T-CAN485 needs specific GPIOs driven to enable the RS485 chip — see the `output:` block in the config. This is board-specific; other ESP32 boards will need different pins.
 
## Wiring
 
Consult the [official NIBE electrical diagrams](https://www.nibe.eu/nl-nl/installateur/schema-s-overzichten-en-technische-ondersteuning/elektrische-aansluitoverzichten) for your exact model — terminal numbering differs between them.
 
| T-CAN485 | Heat pump |
|---|---|
| A | A |
| B | B |
| GND | GND |
| 5–12 V in | 12 V |
 
**Power down the pump before wiring.** Reversed A/B is the most common failure mode and it is completely silent — no error, no data.
 
## Configuration
 
See `nibegw.yaml`. Key points:
 
- **Static IP is mandatory.** Home Assistant sends read/write requests *to* the gateway, so the address must not move. Set it with `manual_ip:` in the ESPHome config rather than relying on a DHCP reservation.
- `target:` and `source:` are your **Home Assistant** IP, not the gateway's.
- `power_save_mode: none` — the device must not miss inbound UDP.
- `fast_connect: true` — required for hidden SSIDs.
- `logger: baud_rate: 0` — serial logging is off. **The USB console will show nothing. This is expected, not a failed flash.** Use the ESPHome dashboard logs over WiFi.
Secrets go in `secrets.yaml` (gitignored):
 
```yaml
wifi_ssid: "..."
wifi_password: "..."
```
 
## First-time setup
 
### 1. Flash the board
 
```bash
python3 -m venv venv && source venv/bin/activate
pip install esphome
esphome run nibegw.yaml
```
 
Plug in over USB-C and pick the serial port (`/dev/cu.wchusbserial*` on macOS, `/dev/ttyUSB*` on Linux).
 
If it won't enter bootloader mode: hold **BOOT**, tap **RST**, release **BOOT**, retry.
 
macOS 11.3+ generally has a working CH34x driver built in. If no port appears, install WCH's `CH34xSER_MAC` and reboot.
 
### 2. Reserve the IP
 
Even with `manual_ip` set, make sure the address sits outside your router's DHCP pool so nothing else is ever handed it.
 
### 3. Enable MODBUS40 on the pump
 
Hold the back button for 7 seconds to reveal the **service** menu → **5.2.4** → activate MODBUS40 in the accessory list.
 
### 4. Add to Home Assistant
 
The ESPHome device is auto-discovered. It contributes only two entities (restart, safe-mode boot) — the pump data comes from the next step.
 
### 5. Add the Nibe Heat Pump integration
 
Choose the **nibegw** connection type (not Modbus — that's for S-series pumps with native Modbus TCP).
 
| Field | Value |
|---|---|
| Model | your pump |
| Remote IP | the ESP32's static IP |
| Remote read port | 9999 |
| Remote write port | 10000 |
| Listening port | 9999 |
 
Most of the ~990 entities are disabled by default. Enable the ones you want.
 
---
 
## Replacing a failed board
 
The T-CAN485 sits on a 12 V rail with RS485 running to the pump, and can die. Replacement is straightforward **provided you don't touch the Nibe Heat Pump integration.**
 
### What survives, and what doesn't
 
Entity IDs come from the Nibe integration's config entry, not from the ESP32. The integration has no idea the hardware behind the IP changed. Leave it alone and every entity, dashboard card and history graph keeps working.
 
**Deleting and re-adding the integration regenerates every unique ID.** That breaks all your cards, resets your enabled-entity selection, and orphans your recorder history. Don't.
 
### Steps
 
1. **Find out why the old one died** before wiring a new one in — check the 12 V rail and inspect the A/B run for shorts.
2. **Flash the new board** with the same config. The binary isn't tied to a MAC or serial number, so the same image works. A blank chip needs the `.factory.bin` (bootloader + partition table + app at 0x0), not the OTA-only binary — `esphome run` over USB handles this for you.
3. **Confirm it takes the old IP.** `ping <ip>` and `nc -zv <ip> 6053` (the ESPHome API port).
4. **Check menu 5.2.4** — MODBUS40 should still be active. Clear any communication alarm the failure left behind.
5. **Rewire** — power off first. A→A, B→B.
6. **Do nothing in the Nibe integration.** Entities come back on their own.
The new board has a different MAC, so ESPHome entity unique IDs change — you may end up with `button.nibegw_restart_2`. Deleting the old ESPHome device *before* adopting the new one avoids that. Cosmetic either way; it doesn't affect the pump entities.
 
---
 
## Troubleshooting
 
**`Please remove the 'platform' key from the [esphome] block`**
 
Config predates ESPHome 2022.x. Move the platform out into its own top-level component:
 
```yaml
esphome:
  name: nibegw
 
esp32:
  board: esp32dev
  framework:
    type: arduino
```
 
Keep `type: arduino` explicit — newer ESPHome defaults ESP32 to esp-idf, and this component is arduino-tested.
 
**Build fails in the Home Assistant ESPHome add-on with PlatformIO toolchain errors**
 
Symptoms include `RuntimeError: at level 0, expected 1 entry`, `Directory not empty`, or `FileNotFoundError: .../tool-cmake/bin/cmake`. These are broken package extraction in the add-on's cache, not a config problem — and on ARM hosts they can recur after clearing.
 
Options, in increasing order of effort: **Clean Build Files** and restart the add-on; wipe `/data/cache/platformio` and `/root/.platformio` inside the container; reinstall the add-on (your YAML lives in `/config/esphome` and is untouched).
 
Or sidestep it entirely and compile on a laptop with `pip install esphome`. The board has to be plugged into that machine to flash anyway.
 
**`FileExistsError: .pioenvs/nibegw`**
 
Two builds ran at once. Both "Plug into this computer" and "Manual download" compile first, and the log can sit silent for minutes. Start one build and leave it alone.
 
**Entities unavailable after a board swap**
 
In order: is the ESP32 actually on the expected IP; is A/B polarity correct; is MODBUS40 still active in 5.2.4; does `source:` in the ESPHome config still match Home Assistant's current IP.
 
**Nothing on the serial console after flashing**
 
`logger: baud_rate: 0` disables it. Working as configured.
 
---
 
## Security note
 
Don't commit `secrets.yaml`. Also check that no API encryption key, OTA password or fallback hotspot password is left in commented-out blocks — comments are fully readable on a public repo, and remain in git history after deletion.
 

