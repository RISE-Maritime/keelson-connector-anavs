# Keelson Connector ANavS

A high-performance connector for ANavS GNSS/INS devices that interfaces with their binary protocol format. This connector can output data at up to 125Hz, significantly higher than the ~10Hz limitation of NMEA formats.

## Overview

The ANavS connector reads binary protocol data from ANavS GNSS/INS devices and publishes it via the Keelson/Zenoh messaging system. It supports both direct TCP connections to ANavS devices and reading from stdin for integration with tools like SOCAT.

### Key Features

- **High-frequency data output**: Up to 125Hz binary format support
- **Multiple input modes**: Direct TCP connection or stdin piping
- **Flexible publishing**: Selective data type publishing (position, velocity, acceleration, attitude)
- **Real-time performance**: Optimized for low-latency maritime and robotic applications
- **Container support**: Docker containerization for easy deployment

## Installation

### Prerequisites

```bash
sudo apt install socat  # For TCP/UDP bridging
```

### Python Environment Setup

1. Clone the repository
2. Set up a Python virtual environment (Python >= 3.11)
3. Install dependencies:

```bash
# Core dependencies
pip install -r requirements.txt

# Development dependencies (includes testing and linting tools)
pip install -r requirements_dev.txt
```

### Docker Setup

The `Dockerfile` builds a container with the connector. Use `docker-compose.anavs.yml` for containerized deployment.

```bash
# Build the container
docker build -t keelson-connector-anavs .

# Run with docker-compose
docker-compose -f docker-compose.anavs.yml up
```

## Usage

### Command Line Options

```bash
./bin/main [OPTIONS]
```

**Core Options:**
- `-r, --realm`: Realm/domain identifier (default: "rise")
- `-e, --entity-id`: Unique entity identifier (required)
- `-s, --source-id`: Source identifier (default: "anavs/0")
- `-l, --log-level`: Log level (10=DEBUG, 20=INFO, 30=WARN, 40=ERROR, 50=CRITICAL)

**Input Options:**
- `--input-mode`: Input mode - "tcp" (direct connection) or "stdin" (pipe input)
- `--anavs-host`: ANavS device IP address (default: "192.168.1.124")
- `--anavs-port`: ANavS device port (default: 6001)

**Publishing Options:**
- `--publish`: Data types to publish - choices: ["all", "raw", "location_fix", "ecef_position", "velocity", "acceleration", "attitude", "utc_time", "gps_timing", "result_code", "satellites_used", "accuracy", "timing", "status"]

**Advanced Options:**
- `--connect`: Custom Zenoh endpoints (for non-multicast networks)
- `-f, --frame-id`: Frame identifier for coordinate systems

### Quick Start Examples

#### Direct TCP Connection
```bash
# Connect directly to ANavS device
./bin/main --log-level 20 -r rise -e usv -s ins/0/anavs --publish all

# Publish only position and attitude data
./bin/main --log-level 20 -r rise -e usv -s ins/0/anavs --publish location_fix --publish attitude
```

#### SOCAT Piping (Recommended for Production)
```bash
# TCP connection via SOCAT
socat TCP:192.168.1.124:6001 STDOUT | ./bin/main --log-level 20 -r rise -e usv -s ins/0/anavs --publish all --input-mode stdin

# UDP connection via SOCAT
socat UDP4-RECV:8500,reuseaddr STDOUT | ./bin/main --log-level 20 -r rise -e ssrs18 -s rutx --publish all --input-mode stdin
```

#### Serial/USB Connection
```bash
# Serial connection via SOCAT
socat /dev/ttyUSB0,b115200 STDOUT | ./bin/main --log-level 20 -r rise -e usv -s ins/0/anavs --publish all --input-mode stdin
```

## Published Data Types

The connector publishes various data types to Zenoh topics:

### Core Navigation Data
- **location_fix**: GPS position (lat/lon/altitude) - Foxglove LocationFix format
- **velocity**: NED velocity vector (m/s) - Decomposed3DVector format
- **acceleration**: Body frame acceleration (m/s²) - Decomposed3DVector format
- **attitude**: Euler angles (heading, pitch, roll in degrees) - TimestampedFloat format

### Raw and Timing Data
- **raw**: Raw binary protocol data - TimestampedBytes format
- **utc_time**: UTC timestamp from GPS
- **gps_timing**: GPS week and time-of-week

### Quality and Status Information
- **result_code**: ANavS solution quality indicator
- **satellites_used**: Number of satellites in solution
- **accuracy**: Position and velocity accuracy estimates
- **status**: Device status information

## Development

### Project Structure

```
keelson-connector-anavs/
├── bin/
│   ├── main                    # Main executable
│   └── terminal_inputs.py      # Command line argument parsing
├── experimental/               # Jupyter notebooks and examples
│   ├── NMEA.ipynb             # NMEA format examples
│   ├── Post-processing-GNSS.ipynb  # GNSS post-processing examples
│   └── nmea.json              # Sample data
├── tests/                      # Unit tests
│   ├── test_gngst.py          # GNGST message tests
│   ├── test_gnzda.py          # GNZDA message tests
│   ├── test_pashr.py          # PASHR message tests
│   └── test_rot.py            # ROT message tests
├── docker-compose.anavs.yml    # Docker composition for deployment
├── Dockerfile                  # Container build configuration
├── requirements.txt            # Core Python dependencies
├── requirements_dev.txt        # Development dependencies
└── ANavS-Reference-Guide-v2.17.pdf  # Protocol documentation
```

### Running Tests

```bash
# Run all tests
python -m pytest tests/

# Run specific test file
python -m pytest tests/test_gngst.py

# Run with coverage
python -m pytest --cov=bin tests/
```

### Code Quality

```bash
# Format code
black bin/ tests/

# Lint code
pylint bin/ tests/
```

## Zenoh Router Setup

For development, you'll need a Zenoh router running. Here's a quick setup:

1. **Start Zenoh router:**
   ```bash
   # Using Docker (recommended)
   docker run --rm -p 7447:7447/tcp -p 7447:7447/udp -p 8000:8000/tcp eclipse/zenoh:latest

   # Check router status
   curl http://localhost:8000/@/router/local
   ```

2. **Test connectivity:**
   ```bash
   # Install Zenoh CLI tools
   pip install zenoh-cli

   # Test publishing
   z_pub -k "test/topic" -v "Hello Zenoh"

   # Test subscribing (in another terminal)
   z_sub -k "test/topic"
   ```

## Integration Examples

### Maritime Applications
```bash
# Vessel navigation system
socat TCP:192.168.1.124:6001 STDOUT | ./bin/main -r maritime -e vessel_001 -s ins/primary --publish location_fix --publish velocity --publish attitude --input-mode stdin
```

### Robotics Applications
```bash
# High-frequency robot localization
socat TCP:192.168.1.124:6001 STDOUT | ./bin/main -r robotics -e robot_01 -s gnss/ins --publish all --input-mode stdin
```

### Research and Development
```bash
# Full data logging for analysis
socat TCP:192.168.1.124:6001 STDOUT | ./bin/main -r research -e testbed_01 -s anavs/lab --publish all --log-level 10 --input-mode stdin
```

## Performance Notes

- **Optimized parsing**: Uses pre-compiled binary parsers for minimal CPU overhead
- **Pre-computed keys**: Zenoh topic keys are pre-compiled for faster publishing
- **Immediate publishing**: No buffering delays - data is published as soon as it's parsed
- **Memory efficient**: Circular buffer management prevents memory leaks during long runs

## Troubleshooting

### Common Issues

1. **Connection refused**: Check ANavS device IP and port settings
2. **No data received**: Verify binary protocol is enabled on ANavS device
3. **High CPU usage**: Reduce log level or limit published data types
4. **Zenoh connection issues**: Verify router is running and reachable

### Debug Commands

```bash
# Check device connectivity
telnet 192.168.1.124 6001

# Monitor raw data stream
socat TCP:192.168.1.124:6001 STDOUT | hexdump -C

# Test with minimal logging
./bin/main --log-level 40 -r test -e debug -s test --publish raw --input-mode tcp
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add/update tests as needed
5. Run code quality checks
6. Submit a pull request

## License

[Add license information here]

## References

- [ANavS Binary Protocol Reference Guide](ANavS-Reference-Guide-v2.17.pdf)
- [Keelson Framework Documentation](https://github.com/RISE-Maritime/keelson)
- [Zenoh Documentation](https://zenoh.io/docs/)
- [Zenoh CLI Tools](https://github.com/RISE-Maritime/zenoh-cli)
