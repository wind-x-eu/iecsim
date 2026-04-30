# IEC61850 Simulator

Brief usage guide for the simulator executable.

## Arguments
- `--bind <ip>`: Bind address. Default is `0.0.0.0`.
- `--port <n>`: MMS port. Default is `102`.
- `--interval-ms <n>`: Update interval per turbine (100..60000). Default is `1000`.
- `--turbines <n>` or `-n <n>`: Number of turbines (1..256). Default is `1` or `TURBINES` env var.
- `--reduced-model`: Use reduced IEC 61850 model.
- `--minimal-model`: Use minimal IEC 61850 model.
- `--disable-cswi`: Omit CSWI control LN.
- `--deny-data-list`: Deny MMS data list browsing.
- `--block-cswi-reads`: Reject CSWI reads.
- `--ld-prefix <text>`: Logical device name prefix. Default `WPPSLD`.
- `--ld-suffix <text>`: Logical device name suffix. Default empty.
- `--ld-index-start <n>`: Start index for LD naming. Default `0`.
- `--ld-names <csv>`: Explicit LD names (comma separated).
- `--brcb-name <name>`: Create a single BRCB with this name.
- `--wtur-status-fc MX`: Enable WTUR status values under MX (use `MX`).
- `--wtur-status-co`: Expose WTUR status under CO.
- `--wtur-status-cf`: Expose WTUR status under CF.
- `--wtur-status-dc`: Expose WTUR status under DC.

## Environment variables
- `TURBINES`: Default turbine count when `--turbines` is not used.
- `REDUCED_MODEL`: If `1|true|yes|on`, enables reduced model.
- `MINIMAL_MODEL`: If `1|true|yes|on`, enables minimal model.

## Docker
Pre-built images are available in two registries:
- Private: `harbor.swms-cloud.com/windx/iecsim:latest` and `harbor.swms-cloud.com/windx/iecsimweb:latest`
- Public alternative: `ghcr.io/wind-x-eu/iecsim:latest` and `ghcr.io/wind-x-eu/iecsimweb:latest`

Build the simulator image from the repository root:
```
docker build -f IEC61850Simulator/Dockerfile -t iecsim:latest .
```

Run the simulator container and publish MMS on port 102:
```
docker run --rm -p 102:10202 -e TURBINES=3 -e MINIMAL_MODEL=1 iecsim:latest
```

Build the website image:
```
docker build -f IECSimulatorWebsite/Dockerfile -t iecsimweb:latest .
```

Run the website container:
```
docker run --rm -p 8080:8080 iecsimweb:latest
```

## Docker Compose
`docker-compose.yml` defaults to Harbor images and includes commented GHCR alternatives.
To use GHCR, replace the `image` values for `iecsim` and `iecsimweb` with:
- `ghcr.io/wind-x-eu/iecsim:latest`
- `ghcr.io/wind-x-eu/iecsimweb:latest`

Example `docker-compose.yml` snippet (GHCR only):
```yaml
services:
  iecsim:
    image: ghcr.io/wind-x-eu/iecsim:latest
    platform: linux/amd64
    ports:
      - "102:10202"
      - "10202:10202"
    environment:
      - TURBINES=3
      - MINIMAL_MODEL=1
    restart: unless-stopped

  iecsimweb:
    image: ghcr.io/wind-x-eu/iecsimweb:latest
    platform: linux/amd64
    ports:
      - "8080:8080"
    depends_on:
      - iecsim
    restart: unless-stopped
```

Start simulator + website together from the repository root:
```
docker compose up -d
```

View logs:
```
docker compose logs -f
```

Stop and remove containers:
```
docker compose down
```

## Examples
Start with 3 turbines on default port:
```
dotnet run --project IEC61850Simulator\IEC61850Simulator.csproj -- --turbines 3
```

Bind to localhost and use port 8102:
```
dotnet run --project IEC61850Simulator\IEC61850Simulator.csproj -- --bind 127.0.0.1 --port 8102
```

Use a reduced model and faster update interval:
```
dotnet run --project IEC61850Simulator\IEC61850Simulator.csproj -- --reduced-model --interval-ms 250
```

Explicit LD names:
```
dotnet run --project IEC61850Simulator\IEC61850Simulator.csproj -- --ld-names WPPS,WPPSLD0,WPPSLD1
```
