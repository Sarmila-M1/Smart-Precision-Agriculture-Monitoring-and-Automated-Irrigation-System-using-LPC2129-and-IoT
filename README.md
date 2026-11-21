Smart Precision Agriculture Monitoring and Automated Irrigation System using LPC2129 and IoT
Project Overview
This project is an embedded-based smart agriculture monitoring and
automated irrigation system developed using the LPC2129 ARM7
microcontroller and Embedded C.

The firmware reads field and environmental sensor values, displays
status messages through UART0, and uses an ESP-01 Wi-Fi module to send
data to the ThingSpeak IoT platform. It also controls a water-pump
output according to the soil-moisture, rain and water-level conditions.

Features
- Soil-moisture monitoring through the LPC2129 ADC.
- Water-level monitoring through the ADC.
- Light-level monitoring using an LDR connected to the ADC.
- Temperature and humidity reading using a DHT22 sensor.
- Rain detection through an external interrupt.
- Flame detection through an external interrupt.
- Motion detection through a PIR sensor and external interrupt.
- Automatic pump output control based on soil moisture, rain detection
  and water level.
- Buzzer output control in the flame and PIR interrupt routines.
- Red, green and yellow LED status indication.
- UART0 serial messages for monitoring system status.
- ESP-01 Wi-Fi communication through UART1.
- ThingSpeak update request over TCP/IP using HTTP.
Hardware Components
  -> LPC2129 ARM7 microcontroller (Main controller)
  -> Soil-moisture sensor (Measures soil moisture using ADC) (channel 0)
  -> Water-level sensor (Measures water level using ADC) (channel 1)
  -> LDR  (Measures light intensity using ADC) (channel 2)
  -> DHT22 (Measures temperature and humidity)
  -> Rain sensor (Provides rain status through external interrupt EINT1)
  -> Flame sensor (Provides flame status through external interrupt EINT2)
  -> PIR sensor (Provides motion status through external interrupt EINT3 )
  -> Relay driver and water pump (Switches the irrigation pump)
  -> ESP-01 Wi-Fi module (Provides Wi-Fi connectivity for ThingSpeak updates)
  -> Buzzer (Provides an audible alert)
  -> LEDs (Indicates system status)
  -> RTC device (RTC access routines are present in rtc.c)
  
Software and Interfaces
  -> Programming language: Embedded C
  -> Microcontroller: LPC2129 (ARM7)
  -> Development environment: Keil µVision-compatible LPC21xx project
  -> UART0: Serial status/debug output
  -> UART1: Communication with the ESP-01 module
  -> ADC: Reads soil moisture, water level and LDR values
  -> External interrupts: EINT1, EINT2 and EINT3 for rain, flame and PIR inputs
  -> IoT platform: ThingSpeak
  -> Network protocol in firmware: ESP-01 AT commands, TCP connection and HTTP GET request.
    
How the System Works

1. The firmware initializes CAN, I2C, UART1, UART0, the Wi-Fi
   connection, ADC and the rain, flame and PIR external interrupts.
2. The main loop reads soil moisture, water level and LDR values using
   the ADC.
3. It reads temperature and humidity using the DHT22 routine.
4. It calls the ThingSpeak update routine to send sensor values through
   the ESP-01.
5. It evaluates sensor states and prints the corresponding status
   messages through UART0.
6. It updates the LED indication according to the detected system
   conditions.
7. The rain, flame and PIR interrupt handlers update their state
   variables. The flame and PIR handlers also control the buzzer.
8. The pump output is controlled using the conditions described below.
9. The main loop includes a 250 ms delay before repeating.
Irrigation Control Logic
The pump output is controlled in main.c using the following
conditions:
  -> Soil moisture is greater than 80%  (Pump OFF)
  -> Rain is detected (rainv == 1)     (Pump OFF)
  -> Water level is below 20%          (Pump OFF)
  ->None of the above OFF conditions applies and soil moisture is below 20%  (Pump ON)

DHT22
dth22.c contains a timing-based routine to read temperature and
humidity from the DHT22. The values and checksum status are stored in
the dht22v structure.
External-interrupt sensors
- Rain sensor: The EINT1 handler updates rainv. When the handler
  detects the rain state, it can command the pump OFF.
- Flame sensor: The EINT2 handler updates flamev and controls
  the buzzer.
- PIR sensor: The EINT3 handler updates pirv and controls the
  buzzer.
The interrupt routines toggle their state variables. The resulting
active/inactive interpretation depends on the sensor output behavior and
the configured interrupt edge.
ESP-01 and ThingSpeak Communication
The ESP-01 is controlled through UART1 using AT commands.

The Wi-Fi and cloud routines are implemented in ESP_01_WIF_MODULE.c:
1. Send AT to check communication with the ESP module.
2. Set station mode using AT+CWMODE=1.
3. Join the configured Wi-Fi network using AT+CWJAP.
4. Open a TCP connection to api.thingspeak.com on port 80.
5. Build and send an HTTP GET request containing sensor fields.
6. Close the TCP connection.

To prepare the project in Keil µVision:
1. Create a project for the LPC2129 device.
2. Add the required source files and the correct LPC2129 startup/device
   support files.
3. Configure the oscillator, PLL and peripheral clock settings to match
   the target hardware.
4. Configure the UART terminal for the firmware's selected baud rate.
5. Connect the analog sensors to the ADC inputs used by the firmware.
6. Connect the rain, flame and PIR sensor outputs to the
   external-interrupt inputs configured in the source.
7. Connect the relay driver, buzzer and LEDs to the GPIO pins defined
   in the source.
8. Connect the ESP-01 to the UART1 pins and provide a suitable
   regulated supply for the Wi-Fi module.
9. Configure Wi-Fi credentials and the ThingSpeak Write API key
   locally.
10. Build, program and test the firmware, first checking UART output and
    sensor readings, then testing pump switching and cloud
    communication.
Check the LPC2129 datasheet and the actual board schematic before
wiring. The ESP-01 requires a suitable 3.3 V supply and compatible UART
logic levels; do not power it directly from an unsuitable MCU supply
pin.
Source-Code Notes and Items to Verify
The following limitations are visible in the supplied source and should
be addressed before treating the firmware as a verified, complete build:
1. I2C initialization is missing: main.c calls i2c_init() and
   header.h declares it, but the supplied repository does not contain
   its implementation. The RTC routines in rtc.c depend on I2C. Add
   the correct I2C driver or remove the I2C/RTC initialization call if
   RTC functionality is not required.
2. CAN transmission is not called by the main loop: can1_init()
   is called, but can_send_data() is commented out. CAN functions are
   present, but the current main loop does not transmit the sensor data
   over CAN.
3. ThingSpeak field formatting needs correction: The HTTP request
   format contains field1 through field8, but the sprintf() call
   supplies only six sensor values. Correct the format arguments and
   verify the resulting request.
4. UART1 and LED pin assignment conflict: uart1_init() selects
   P0.8 and P0.9 for UART1, while led.c configures P0.8 and P0.9 as
   LED outputs. These functions cannot independently use the same pins
   at the same time. Resolve the pin assignment before hardware
   testing.
5. Required standard declaration: ESP_01_WIF_MODULE.c uses
   strlen() but does not include <string.h>. Add the appropriate
   include.
6. Sensor calibration: Verify the fixed soil-moisture calibration
   constants and the pump thresholds with the actual sensors and
   irrigation setup.
7. Blocking waits: Sensor and Wi-Fi routines use delays and
   blocking waits. Test the resulting sampling interval and response
   time on the target hardware.
8. Shared buzzer output: Both flame and PIR interrupt handlers
   control the same buzzer. Consider combining alert states so one
   event cannot switch off the buzzer while another alert remains
   active.
9. CAN clock configuration: can1_init() sets VPBDIV=1. Confirm
   that this peripheral clock setting is intentional for the complete
   application and is consistent with UART baud-rate calculations.
Project Scope
This repository's application code covers analog sensor monitoring,
DHT22 readings, external-interrupt sensor handling, pump output control,
LED/buzzer indications and ESP-01/ThingSpeak communication.

