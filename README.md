# Internet-of-Things
A serie of several mini projects made for the Internet of Things course at Politecnico di Milano
Assignments completed in pairs, overall grade: 28.

--------------------------------------------------------------------------------
# IoT Challenges & Exercises Summary

This repository summarizes the work completed across various Internet of Things (IoT) challenges and homework exercises, covering hardware integration, communication protocols, power management, network optimization, and data processing.

## Challenge 1: IoT Device Design & Power Optimization (ESP32 & HC-SR04)

This challenge focused on designing and optimizing an **ESP32-based IoT device** for distance measurement and status reporting, with a strong emphasis on power efficiency.

*   **Project Overview**: An ESP32 integrated with an **HC-SR04 ultrasonic sensor** measures distance, communicating via **ESP-NOW protocol**. The primary goal was to optimize power consumption using **deep sleep mode**. The device detects if an object is within a predefined minimum distance (50 cm) and sends an "OCCUPIED" or "FREE" status message to a receiver [1].
*   **Key Technologies & Implementations**:
    *   **ESP32**: Main processing unit [1].
    *   **HC-SR04 Ultrasonic Sensor**: Used for distance measurement, triggered by a 10-microsecond pulse on `TRIG_PIN` (pin 5) and reading pulse width from `ECHO_PIN` (pin 18) [2-4].
    *   **ESP-NOW Protocol**: Utilized for communication with a master device, sending status messages based on distance threshold [1, 5-8].
    *   **Deep Sleep Mode**: Enabled for power saving using `esp_sleep_enable_timer_wakeup()` for a calculated duty cycle `X` (36 seconds) and initiated via `esp_deep_sleep_start()` [1, 6, 9].
    *   **Wi-Fi Management**: Wi-Fi is enabled in Station mode (`WIFI_STA`) only for ESP-NOW initialization and data transmission, then immediately disabled (`WIFI_OFF`) to conserve power [6, 8, 9].
    *   **RTC Memory**: `RTC_DATA_ATTR` was used to store `lastValue` persistently across deep sleep cycles, enabling conditional notification only when the status changes, further optimizing energy [10, 11].
*   **Energy Consumption & Lifetime Analysis**:
    *   The project calculated total battery energy `Y = 15,181 J` [12].
    *   Average power consumption for different states (Deep Sleep, Idle/Boot/WiFi Off, WiFi On, Transmission, Sensor Read) was estimated [13, 14].
    *   Initial lifetime was estimated at **13,627 cycles** (approx. 1114 mJ per transmission cycle) [10, 15].
    *   Through optimizations like conditional notification and Wi-Fi management, energy consumption was reduced to **966.41 mJ** per cycle, extending the device's lifetime to **15,708 cycles** [11, 16]. This represents a significant improvement in battery life [11].

## Challenge 1 (Exercise): Wireless Sensor Network Sink Optimization

This exercise focused on optimizing the sink position in a Wireless Sensor Network (WSN) to maximize its lifetime.

*   **Problem**: A parking lot with 10 sensors, each transmitting status updates every 10 minutes (2000 bits packet size, 5 mJ initial energy). The goal was to determine system lifetime with a fixed sink and then find an optimal sink position [17, 18].
*   **Energy Model**: Energy consumption depends on TX/RX circuitry (`Ec = 50 nJ/bit`) and transmission energy (`Etx(d) = k · d^2` where `k = 1 nJ/bit/m^2`) [18, 19].
*   **Key Findings**:
    *   **Fixed Sink Position (20, 20)**: Sensor 1, being the furthest (26.17m), consumed the most energy (1.47 mJ per transmission), limiting the system lifetime to **3 transmissions (30 minutes)** [20-22].
    *   **Optimal Sink Position**: An optimization problem was solved to find the coordinates that minimize energy consumption for the worst-case sensor [19, 23, 24]. The optimal position was identified as **(8.10, 6.80)**. While the source notes (1,2) as the worst case, the calculated optimal position reduces the maximum energy consumption.
    *   With the optimal sink position, the energy consumption for the "worst-case" sensor was significantly reduced (e.25 mJ), extending the system lifetime to **20 cycles** [24, 25].
*   **Trade-offs (Fixed vs. Dynamic Sink)**:
    *   **Fixed Sink**: Low complexity, easy to implement, but higher energy consumption for distant nodes and shorter network lifetime [26].
    *   **Dynamic Sink**: High complexity, requires adaptive control, but offers more balanced energy consumption and longer network lifetime due to energy balancing [26].

## Challenge 2: CoAP & MQTT Protocol Analysis

This challenge involved a detailed analysis of CoAP and MQTT protocols, including packet capture analysis and energy consumption comparison.

*   **Tools**: **Python** with the **`pyshark` library** was used to scan and filter `.pcap` files for protocol analysis [27-33].
*   **Key Queries & Findings (from `challenge2.pcap` analysis)**:
    *   **CQ1 (CoAP PUT Unsuccessful Responses)**: Identified **8** unique confirmable PUT requests that received an unsuccessful response from the local CoAP server [27, 34].
    *   **CQ2 (CoAP Equal GET Requests)**: Found **3** CoAP resources (`secret`, `validate`, `large`) on `coap.me` public server that received the same number of unique Confirmable and Non-Confirmable GET requests [28, 35].
    *   **CQ3 (MQTT Multi-level Wildcard Subscriptions)**: Counted **4** different MQTT clients subscribing to the public HiveMQ broker using multi-level wildcards (`#`) [29, 36].
    *   **CQ4 (MQTT Last Will 'university' Topic)**: Determined that **1** MQTT client specified a Last Will Message directed to a topic with "university" as the first level [36, 37].
    *   **CQ5 (MQTT Last Will without Wildcard)**: Identified **3** MQTT subscribers that would receive a Last Will message derived from a subscription without a wildcard [31, 38].
    *   **CQ6 (MQTT Publish Retain & QoS 0)**: Found **208** MQTT PUBLISH messages directed to the public Mosquitto broker that were sent with the retain option and QoS "At most once" (QoS 0) [32, 33].
    *   **CQ7 (MQTT-SN on Port 1885 Local Broker)**: Counted **0** MQTT-SN messages sent by clients to a broker on the local machine via UDP port 1885 [33, 39].

### Energy Consumption Comparison (CoAP vs. MQTT)

A comparative analysis of energy consumption for a battery-powered sensor and valve over 24 hours, interacting with a Raspberry Pi broker.

*   **Assumptions**: `ETX = 50 nJ/bit`, `ERX = 58 nJ/bit`, `Ec = 2.4 mJ` (valve processing), ideal Wi-Fi [40]. Sensor transmits every 5 minutes, valve computes average every 30 minutes [41-43].
*   **CoAP Case**: Communication only between sensor and valve (Raspberry Pi doesn't support CoAP). Sensor acts as client (PUT requests), valve as server (PUT responses, ACKs) [44].
    *   **Sensor Consumption**: **18.73 mJ** over 24 hours [43, 45].
    *   **Valve Consumption**: **133.85 mJ** over 24 hours (includes 48 computations) [43, 45].
*   **MQTT Case**: Sensor (publisher), Valve (subscriber), Raspberry Pi (broker) [45].
    *   **Sensor Consumption (Publisher)**: **7.88 mJ** over 24 hours [46, 47].
    *   **Valve Consumption (Subscriber)**: **9.18 mJ** over 24 hours (includes processing) [48, 49].
    *   **Total System Consumption (MQTT)**: **17.06 mJ** [50].
*   **Energy Saving Solution (MQTT)**:
    *   **Batching Transmissions**: By encoding temperature readings (2 bytes/reading) and transmitting 6 readings in a 12-byte payload every 30 minutes (instead of single readings every 5 minutes), significant savings were achieved [50, 51].
    *   This strategy reduced the sensor's transmission operations and the overall number of publish operations.
    *   **Estimated Energy Savings**: Approximately **50.77%** in transmission energy and **31.85%** in total system energy [52, 53].

## Challenge 3: Node-RED & LoRaWAN

This challenge involved developing a Node-RED flow for IoT data management and analyzing LoRaWAN network performance.

*   **Node-RED Flow Development**:
    *   **Objective**: Manage and monitor sensor data lifecycle: reading from CSV, publishing/receiving MQTT messages, logging acknowledgments, filtering, and visualizing data locally and via Thingspeak [54].
    *   **Key Components & Logic**:
        *   **Init Read CSV**: At startup, a CSV file is read, parsed, and stored in a flow variable for fast access [55].
        *   **Trigger Send 5s**: Every 5 seconds, a message with a random ID (0-30000) and timestamp is generated, published to `challenge3/id_generator` (max 80 messages), and logged to `id_log.csv` [55, 56].
        *   **Read from Topic**: Subscribes to the MQTT topic. Received IDs are modulo 7711 to compute an index `N` for matching rows in the parsed CSV data [57, 58].
        *   **Conditional Processing**:
            *   If the CSV row indicates a "Publish Message", MQTT topics are extracted, raw payloads parsed into JSON, and formatted messages are prepared for publishing. Temperature data in Fahrenheit is plotted and logged to `filtered_pubs.csv` [58-60]. A rate limit of 4 messages/minute is applied [61].
            *   If the row indicates an "ACK message" (e.g., Connect Ack, Sub Ack), an ACK counter is incremented, logged to `ack_log.csv`, and the count is sent to **Thingspeak via HTTP API** [61, 62].
        *   **Flow Termination**: A message counter limits total processed subscription messages to 80 [62].
    *   **Visualization**: Valid Fahrenheit temperature data is parsed, averaged, and visualized in a real-time Node-RED chart [63].
*   **LoRaWAN Network Analysis**:
    *   **Scenario**: Europe (868 MHz, 125 kHz bandwidth), 1 gateway, 50 sensor nodes. Payload size `L = 84 Bytes` (based on person code 10710181) [64, 65]. Packet transmission follows a Poisson process (1 packet/minute) [64].
    *   **Objective**: Find the largest Spreading Factor (SF) for a success rate of at least 70% [64].
    *   **Finding**: Based on calculations and `thethingsnetwork.org` airtime calculator, **SF7** was determined to be the only option meeting the 70% success rate requirement, achieving **75.4%** [66-68].
*   **System Design (Arduino MKR WAN 1310)**:
    *   A system design was proposed for reading temperature/humidity from a **DHT22 sensor** and sending data to **Thingspeak over LoRaWAN** [68].
    *   **Steps**:
        1.  Set up a LoRaWAN server via **The Things Network (TTN)** and register the Arduino device [69].
        2.  Create two distinct **Thingspeak channels** (for temperature and humidity) and obtain API keys [69].
        3.  Configure a **webhook in TTN** to trigger upon receiving payloads from the LoRaWAN device, forwarding data to the corresponding Thingspeak channels via HTTP [69].
*   **LoRaSim Figure Reproduction**:
    *   Reproduced Figure 5 (single-sink) and Figure 7 (multi-sink) from the paper "Do LoRa Low-Power Wide-Area Networks Scale?" using the **LoRa simulator (`lorasim/loraDir.py` and `lorasim/loraDirMulBS.py`)** [70-72].
    *   The simulation involved varying the number of nodes and sinks, with the collision flag enabled [70, 72].

## Homework Exercise 1: Forklift IoT System Design

This exercise focused on designing a comprehensive IoT system for real-time localization and status monitoring of forklifts in an industrial warehouse.

*   **Hardware Components**:
    *   **ESP32**: Main microcontroller [73].
    *   **IMU (Accelerometer & Gyroscope)**: For motion tracking, distance estimation, and collision detection [73, 74].
    *   **LoRa Module**: For long-range, low-power wireless communication with LoRa gateways [74].
    *   **BLE Module**: Scans fixed BLE beacons for indoor positioning via trilateration [74, 75].
    *   **GPS Module**: For accurate outdoor positioning [74].
*   **BLE Beacon Setup (Indoor Localization)**:
    *   Fixed BLE beacons broadcast ID and signal strength in the 500m² underground warehouse [75].
    *   Forklift ESP32 scans RSSI values, sends to backend for trilateration [75].
*   **Communication Strategy (Hybrid)**:
    *   **LoRa Dual-Frequency**:
        *   **Sub-GHz (e.g., 433 MHz)**: For underground indoor communication (better penetration) [76].
        *   **Higher LoRa Frequency (e.g., 868/915 MHz)**: For outdoor yard communication (better throughput/range) [76].
    *   **BLE**: Exclusively indoors for proximity-based positioning via trilateration [77].
*   **Data Transmission Frequency (Optimized)**:
    *   **Position Updates**: Every 5 seconds (active movement), every 30 seconds (stationary) [77].
    *   **Impact/Collision Events**: Sent immediately using interrupt-driven logic [78].
    *   **Aggregated Daily Metrics**: Sent at end of shift or when docked [78].
*   **Backend Architecture**: Designed for low-latency ingestion, efficient processing, scalable storage, and real-time visualization [79].
    *   **Data Ingestion**: LoRa Gateway publishes telemetry data (BLE RSSI, GPS, accelerometer) to an **MQTT Broker** (e.g., Mosquitto) [79].
    *   **Data Processing**: A **Stream Processor** subscribes to MQTT topics, parses data, performs BLE trilateration, merges GPS/trilaterated coordinates, detects impacts, and aggregates metrics [80].
    *   **Data Storage**: **Time-Series Database** for telemetry, **Relational Database** for metadata (forklifts, beacons) [80].
    *   **Visualization & Monitoring**: A **Web Dashboard** displays real-time forklift positions, KPIs (distance, speed, impacts, battery levels) [81].

## Homework Exercise 2: IEEE 802.15.4 CFP Slot Assignment

This exercise analyzed an ESP32-based camera system using IEEE 802.15.4 in beacon-enabled mode (CFP only).

*   **System**: ESP32-based IoT camera monitoring system. Payload size varies based on person count in frame: 1 KB (0 persons), 3 KB (1 person), 6 KB (>1 person) [82, 83].
*   **Probabilistic Model**: Number of people `k` follows a **Poisson distribution** with `λ = 0.15` persons/frame [82, 83].
*   **Key Calculations**:
    *   **Output Rate Probability Mass Function (PMF)**:
        *   `P(r = r0 | 0 persons, 1KB payload, 8kb)`: `e^(-0.15)` ≈ **0.8607** [83, 84].
        *   `P(r = r1 | 1 person, 3KB payload, 24kb)`: `0.15 * e^(-0.15)` ≈ **0.1291** [83, 84].
        *   `P(r = r2 | ≥2 persons, 6KB payload, 48kb)`: `1 - P(0) - P(1)` ≈ **0.0102** [83, 84].
    *   **CFP Slot Assignment & Duty Cycle**: For a system with 1 PAN coordinator and 3 camera nodes, nominal bit rate `R = 250 kbps`, packets `L = 128 bytes` [85, 86]:
        *   **Slot Time (Ts)**: **4.096 ms** [87].
        *   **Number of slots in CFP (nCFP)**: **18** (for 3 nodes transmitting max 6KB payload) + 1 beacon control slot = **19 active slots** [87, 88].
        *   **Active Period (Tactive)**: **77.82 ms** [88].
        *   **Inactive Period (Tinactive)**: **1.202 s** (with Beacon Interval `BI = 1.28 s`) [88].
        *   **Duty Cycle**: **6.08%**, well below the 10% target [88].
    *   **Maximum Additional Cameras**: To keep the duty cycle below 10%, considering the worst-case scenario (every node transmitting 1KB every beacon interval), an additional **2 cameras** can be added, for a total of N=5 nodes [89, 90].

## Homework Exercise 3: RFID Dynamic Frame ALOHA

This exercise involved simulating an RFID system using the Dynamic Frame ALOHA (DFA) protocol to evaluate collision resolution efficiency.

*   **System**: RFID system with `N = 4` tags [91, 92].
*   **Protocol**: Dynamic Frame ALOHA (DFA), where the frame size is dynamically updated to match the current backlog after the first frame.
*   **Simulation Method**: **Monte Carlo simulation** (100,000 executions for each `r1`) using a Python script.
*   **Objective**: Find the overall collision resolution efficiency `η = N / L` for different initial frame sizes `r1 = {1, 2, 3, 4, 5, 6}`. `L` is the total number of time slots needed to identify all tags.
*   **Results & Conclusion**:
    *   The simulation results show varying efficiencies based on `r1`.
    *   The **maximum efficiency `η ≈ 0.4540` was obtained when the initial frame size `r1 = 4`**.
    *   This highlights that an appropriate choice of `r1` is crucial for performance, balancing the trade-off between too many collisions (small `r1`) and too many empty slots (large `r1`).

---
