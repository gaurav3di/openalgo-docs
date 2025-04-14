# Fast Scalper

## FastScalper: A New Rust-Based Tool for Scalping

**FastScalper** is an innovative desktop application designed specifically for scalpers who need a lightweight, fast, and robust trading tool. Built using Rust, this cross-platform app integrates seamlessly with Open Algo, providing an intuitive interface to place and manage orders efficiently.

This guide will walk you through the installation, configuration, and key features of FastScalper.

***

### Key Features

1. **Cross-Platform Compatibility**\
   FastScalper is compatible with all major operating systems:
   * **Windows**: Supports multiple instances for tracking and managing various instruments.
   * **macOS**: Compatible with both Intel and Apple Silicon processors.
   * **Linux**: Available for distributions like Ubuntu, Debian, and Fedora.
2. **Tiny and Lightweight**\
   A small, portable application optimized for speed, making it ideal for scalpers requiring instant order execution.
3. **Robust Order Management**
   * Supports multiple product types:
     * **CNC**: Cash and Carry (carry-forward trades).
     * **MIS**: Intraday trading for equities and derivatives.
     * **NRML**: Positional trading for derivatives.
   * Easy-to-use controls:
     * **L**: Place a long position.
     * **LX**: Close long positions.
     * **SE**: Place a short position.
     * **SX**: Close short positions.
4. **Voice Alerts**\
   Configurable voice alerts provide audio confirmation for order placement.
5. **Seamless Integration**
   * Connects to Open Algo via an API key for real-time trading.
   * Orders can be monitored through trading terminals (desktop, mobile, or web).
6. **Customizable Settings**
   * API key and host URL configuration.
   * Exchange and product setup options.
   * Enable/disable voice alerts as needed.

***

### Installation

#### Prerequisites

* Ensure Open Algo is installed and running.
* Obtain the FastScalper installer from the **Downloads** section in Open Algo.

#### Steps

1. **Download FastScalper**\
   Navigate to the **Downloads** section and select the appropriate installer for your operating system.
2. **Install FastScalper**\
   Follow the platform-specific installation steps. Launch the application once installed.
3. **Configure FastScalper**
   * Open FastScalper and go to **Settings**.
   * Enter the following details:
     * **API Key**: Obtainable from the Open Algo API Key section.
     * **Host URL**: The URL where Open Algo is running.
     * Configure additional preferences (e.g., exchange, product type, and voice alerts).
4. **Start Trading**\
   Begin placing and managing orders directly through the FastScalper interface.

***

### Usage

1. **Launching Multiple Instances (Windows)**\
   Open several instances to manage multiple instruments simultaneously.
2. **Order Placement**
   * Enter the symbol and quantity.
   * Use dedicated controls for:
     * **L**: Place a long order.
     * **LX**: Close long positions.
     * **SE**: Place a short order.
     * **SX**: Close short positions.
3. **Error Handling**\
   FastScalper prevents invalid actions, such as attempting to close positions when none exist.
4. **Monitoring Orders**
   * View order status in the order book.
   * Monitor trades via trading terminals or web interfaces.

***

### Known Limitations

* **Single Instance Support**: macOS and Linux currently support only one instance at a time.
* **Feature Requests**: Additional features or configurations can be suggested via the Open Algo GitHub repository.
