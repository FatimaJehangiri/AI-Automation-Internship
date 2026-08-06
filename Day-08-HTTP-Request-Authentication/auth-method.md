# Authentication Method Used

## API

Open-Meteo Weather API

## Authentication

**None**

The Open-Meteo Weather API is a public REST API and does not require authentication for the endpoint used in this project.

No credentials such as:

- API Key
- Bearer Token
- Basic Authentication
- OAuth

are required to access the weather forecast data.

This made it suitable for learning HTTP Request integration and JSON response handling without additional authentication configuration.

## Why No Authentication?

The selected endpoint is publicly accessible and allows users to retrieve weather forecast data by providing only the required query parameters, such as latitude, longitude, timezone, and forecast duration.

This simplifies API integration while demonstrating the core functionality of the HTTP Request node in n8n.
