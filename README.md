# Ground Telemetry Station 

A system for connecting to MAVLink drones and checking their status. More information can be found in each of the projects displayed above as git submodules.

- **Frontend** - Web UI for creating datalinks and viewing telemetry (Alpine.js)
- **Backend** - REST API + WebSocket server (Python/FastAPI)
- **Driver** - MAVLink/gRPC bridge (C++)

## Project deployment with Docker 

The easiest way to run the system is with Docker Compose, which includes an ArduPilot SITL simulator:

```bash
docker-compose up
```

This starts all services:
- Frontend: http://localhost:8085
- Backend API: http://localhost:8000
- Driver gRPC: localhost:50051
- ArduPilot SITL simulator: localhost:5760

### Connecting to the Simulator

1. Open http://localhost:8085
2. Enter `simulator` as the URI (not `127.0.0.1`) to connect to the right Docker instance.
3. Enter `5760` as the port
4. Click "Create Datalink"
5. Click "Create Datastream" to start receiving telemetry. Drone updates sent by the backend will be displayed in the log box and the Drone data will start to be updated.

## Local Development
For local development without Docker, run each service separately:

### 1. Start the ArduPilot Simulator

```bash
sim_vehicle.py -v ArduCopter --no-mavproxy
```

### 2. Start the Driver

```bash
cd mavlink-driver
./build.sh
./build/mavlink_driver
```

### 3. Start the Backend

```bash
cd ground-telemetry-station-backend
./launch_server.sh
```

### 4. Start the Frontend

```bash
cd ground-telemetry-station-frontend
python3 -m http.server 8085
```

Then open http://localhost:8085 and connect to `127.0.0.1:5760`.

## Architecture

```
┌─────────────────┐      HTTP/WS       ┌─────────────────┐
│    Frontend     │◄──────────────────►│     Backend     │
│   (Port 8085)   │                    │   (Port 8000)   │
└─────────────────┘                    └────────┬────────┘
                                                │ gRPC
                                                ▼
                                       ┌─────────────────┐
                                       │     Driver      │
                                       │  (Port 50051)   │
                                       └────────┬────────┘
                                                │ MAVLink
                                                ▼
                                       ┌─────────────────┐
                                       │   Simulator /   │
                                       │   Real Drone    │
                                       │   (Port 5760)   │
                                       └─────────────────┘
```
