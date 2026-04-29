# AISwitch_Demo

A web-based security analysis platform with support for Palo Alto Prisma AIRS and custom API endpoints.

## Features

- **Multi-Engine Support**: Switch between Palo Alto Prisma AIRS and custom API endpoints
- **Real-time Chat Interface**: Interactive threat analysis and security queries
- **Runtime Security Analysis**: Analyze threats, anomalies, and policy violations
- **Configurable Endpoints**: Support for custom API integrations
- **Modern UI**: Dark theme with real-time visual feedback

## Getting Started

### Requirements

- Web browser with modern JavaScript support
- API credentials (either Palo Alto Prisma AIRS or custom API endpoint)

### Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/SHI-Lab-Halik/AISwitch_Demo.git
   ```

2. Open the application in your web browser

### Configuration

#### Prisma AIRS Setup
- API Key: Your Prisma Cloud authentication token
- Tenant ID: Your Prisma Cloud tenant identifier
- Region: Deployment region (us, eu, ap)

#### Custom API Setup
- Endpoint URL: Your custom API endpoint
- API Key/Token: Authentication credentials
- Auth Header: Custom authorization header name

## Usage

1. Select your preferred analysis engine
2. Enter your API credentials in the configuration panel
3. Click **SAVE CONFIG**
4. Type your security query and press Enter or click Send
5. Receive analysis results from the selected engine

## Architecture

The application provides two analysis modes:

- **Prisma AIRS Mode**: Direct integration with Palo Alto Prisma AI Runtime Security
- **Custom API Mode**: Generic REST API endpoint for third-party security tools

## Support

For issues or questions, please refer to the project documentation.
