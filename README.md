# Nibe-HP-IOT
Repository to setup logging from Nibe HP through modbus
- ![Dashboard example]()

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

