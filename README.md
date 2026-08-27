# Synthesizable Full-Duplex UART Controller in Verilog HDL

A cycle-accurate, synthesizable Universal Asynchronous Receiver-Transmitter (UART) RTL core implemented in Verilog HDL. The design supports full-duplex 8-N-1 serial communication featuring configurable baud generation, finite-state machine (FSM) serialization, and 16x oversampling deserialization.

---

## Architecture & Features

- **Protocol Format:** 8-N-1 (1 Start Bit, 8 Data Bits, No Parity, 1 Stop Bit).
- **Baud Rate Generator:** Configurable clock divider for target baud rates (e.g., 9600 baud from a 50 MHz system clock).
- **Transmitter (`uart_sender`):** 4-state FSM handling Start bit transmission, 8-bit LSB-first serialization, and Stop bit generation with status feedback (`busy`).
- **Receiver (`uart_receiver`):** 16x oversampling architecture with midpoint sampling (at the 8th sub-clock tick) to eliminate noise and prevent false framing errors.
- **Top Module (`uart_top`):** Integrated transmitter, receiver, and baud generator with internal/external loopback capability.

---

## Project Structure

├── rtl/
│   ├── baud_rate_generator.v
│   ├── uart_sender.v
│   ├── uart_receiver.v
│   └── uart_top.v
├── tb/
│   └── uart_top_tb.v
└── README.md


## Simulation & Verification

The core was functionally validated via **Icarus Verilog** and visualized using **EPWave** on EDA Playground.

### Waveform Verification
<img width="1870" height="712" alt="UART_Output_Waveform" src="https://github.com/user-attachments/assets/2347e64b-5dd6-4c2d-a37b-c6b5d96877af" />

- Verified multi-byte loopback transmissions (e.g., `0x41` and `0x55`).
- Validated byte alignment, framing intervals, and `rdy` handshaking pulses.
