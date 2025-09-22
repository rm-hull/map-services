# map-services

This project provides a suite of Dockerized microservices for geospatial data, mapping, and related information. It includes a MapProxy instance for serving map tiles, various APIs for specific datasets (e.g., company data, points of interest, weather overlays), and scheduled jobs for data import and maintenance.

## Features

*   **MapProxy:** Serves map tiles, supporting WMTS, WMS, and TMS protocols, with on-the-fly reprojection and caching.
*   **Geospatial APIs:**
    *   **Company Data API:** Provides access to company information.
    *   **Code-Point Data API:** (Implicitly from import)
    *   **Points of Interest (POI) API:** Serves POI data.
    *   **Postcode Polygons API:** Provides geographical data for postcodes.
    *   **UK Weather Overlays API:** Delivers weather data, likely from the Met Office.
    *   **Street Manager Relay API:** Related to street management data.
*   **Data Import & Management:** Scheduled jobs for importing large datasets like Companies House data and Code-Point data.
*   **Monitoring:** `du-exporter` for disk usage metrics.
*   **Scheduled Tasks:** A `cron` service to automate data updates and other maintenance.
*   **External Access:** Configured for integration with Traefik and Cloudflare Tunnel for secure external exposure of services.

## Prerequisites

Before running this project, ensure you have the following installed:

*   [Docker](https://www.docker.com/get-started)
*   [Docker Compose](https://docs.docker.com/compose/install/)

You will also need API keys for certain services:

*   **OS DataHub API Key:** Required for the MapProxy service to access Ordnance Survey map tiles.
*   **Met Office DataHub API Key:** Required for the UK Weather Overlays service.
*   **Unsplash Access Key:** Required for the POI server.

## Setup

1.  **Clone the repository:**

    ```bash
    git clone https://github.com/your-repo/map-services.git
    cd map-services
    ```

2.  **Configure Environment Variables:**
    Copy the example environment file and fill in your API keys:

    ```bash
    cp .env.example .env
    ```
    Edit the `.env` file and replace the placeholder values with your actual API keys:

    ```ini
    OS_DATAHUB_API_KEY="YOUR_OS_DATAHUB_API_KEY"
    METOFFICE_DATAHUB_API_KEY="YOUR_METOFFICE_DATAHUB_API_KEY"
    UNSPLASH_ACCESS_KEY="YOUR_UNSPLASH_ACCESS_KEY"
    ```

3.  **Data Directories:**
    The project expects certain data to be present in the `data/` directory. Some services will import data into these directories, while others expect pre-existing data (e.g., `poi_uk.gpkg`).

## Usage

To start all the services defined in `docker-compose.yml`, run:

```bash
docker compose up -d
```

This will:
*   Start the `cron` scheduler.
*   Generate the `mapproxy.yaml` configuration.
*   Launch the MapProxy server and all other API services.
*   Start the `du-exporter`.

### Accessing Services

Most services are configured with Traefik and Cloudflare Tunnel labels, indicating they are intended to be accessed via a reverse proxy. The example configuration uses `api.homelab.destructuring-bind.org` as the base hostname.

*   **MapProxy WMTS:** `https://api.homelab.destructuring-bind.org/mapproxy/wmts/wmts/...`
*   **UK Weather Overlays API:** `https://api.homelab.destructuring-bind.org/v1/metoffice/datahub/...`
*   **Company Data API:** `https://api.homelab.destructuring-bind.org/v1/company-data/...`
*   **POI Server API:** `https://api.homelab.destructuring-bind.org/v1/geods-poi/...`
*   **Postcode Polygons API:** `https://api.homelab.destructuring-bind.org/v1/postcode/...`
*   **Street Manager Relay API:** `https://api.homelab.destructuring-bind.org/v1/street-manager-relay/...`

### Scheduled Tasks

The `cron` service runs tasks defined in the `crontab` file.
*   **Company Data Import:** Runs monthly to update company data.
*   **Code-Point Import:** Runs monthly to update Code-Point data.

You can inspect the `crontab` file for exact schedules.

## Services Overview

| Service Name             | Description                                                              | Data Directory                 |
| :----------------------- | :----------------------------------------------------------------------- | :----------------------------- |
| `cron`                   | Schedules periodic tasks (e.g., data imports).                           | N/A                            |
| `du-exporter`            | Exports disk usage metrics for monitoring.                               | `data/` (read-only)            |
| `mapproxy-config-generator` | Generates `mapproxy.yaml` from template and environment variables.       | N/A                            |
| `mapproxy`               | Map tile proxy server, serving various map layers.                       | `data/mapproxy/cache_data`     |
| `uk-weather-overlays`    | API for UK weather overlay data.                                         | `data/metoffice-datahub`       |
| `company-data`           | API for company data.                                                    | `data/company-data`            |
| `company-data-import`    | Imports company data from external sources (e.g., Companies House).      | `data/company-data`            |
| `code-point-import`      | Imports Code-Point data from external sources (e.g., OS.uk).             | `data/company-data`            |
| `poi-server`             | API for Points of Interest (POI).                                        | `data/geods-poi`               |
| `postcode-polygons`      | API for postcode polygon data.                                           | N/A                            |
| `street-manager-relay`   | API for street management data.                                          | `data/street-manager-relay`    |

## Configuration

*   **`mapproxy.template.yaml`**: This file is a template for the MapProxy configuration. It uses `envsubst` to inject environment variables (like `OS_DATAHUB_API_KEY`) into the final `mapproxy.yaml` used by the `mapproxy` service.
*   **`crontab`**: Defines the schedule for various data import and maintenance jobs.

## Data

The `data/` directory is central to this project, storing various datasets used by the services:

*   `company-data/`: Stores company-related data, including the database for the `company-data` API.
*   `geods-poi/`: Contains the `poi_uk.gpkg` file for the POI server.
*   `mapproxy/`: Stores MapProxy cache data.
*   `metoffice-datahub/`: Stores Met Office weather data for the `uk-weather-overlays` service.
*   `street-manager-relay/`: Stores data for the `street-manager-relay` service.

## License

This project is licensed under the terms specified in the `LICENSE.md` file.