# ASP.NET Web API Portfolio — Stage 1 (Before Docker)

> **Status:** In Progress
> **Stage:** Environment Setup & Troubleshooting (No Docker yet)

---

## 🎯 Purpose

This stage focuses on preparing a **stable ASP.NET Core Web API (.NET 8)** environment before introducing Docker.

The goal is **not** to build a complex application, but to:

* Ensure the API can run locally
* Resolve real-world environment issues
* Create a solid foundation for Dockerization

---

## 🧱 Project Overview

* Framework: **ASP.NET Core Web API**
* Runtime: **.NET 8 (LTS)**
* Template: Default **WeatherForecast** API
* Endpoint:

```
GET /weatherforecast
```

---

## ⚠️ Issues Encountered & How They Were Solved

### 1️⃣ Default SDK Using .NET 9

**Problem**

* `dotnet new webapi` automatically used **.NET 9**
* Package restore failed due to unstable dependencies

**Solution**

```bash
dotnet new webapi -n WeatherApi -f net8.0
```

**What I Learned**

* The latest installed SDK is always selected by default
* Target framework should be specified explicitly when multiple SDKs exist

---

### 2️⃣ NuGet Restore Failed (NU1100)

**Problem**

* Package restore failed with error:

```
PackageSourceMapping is enabled
nuget.org was not considered
```

**Root Cause**

* Global **NuGet Package Source Mapping** was enabled
* Public NuGet feeds were blocked

**Solution**

* Disabled (`commented out`) the `<packageSourceMapping>` section in `NuGet.Config`

**What I Learned**

* NuGet behavior can be controlled by global configuration
* Restore errors are not always caused by project-level issues

---

### 3️⃣ API Running but Browser Returned 404

**Problem**

* Opening `http://localhost:<port>/` returned **404**

**Explanation**

* ASP.NET Web API does **not** provide a root (`/`) endpoint by default

**Correct Endpoint**

```
http://localhost:<port>/weatherforecast
```

**What I Learned**

* Web APIs expose specific routes, not web pages
* A 404 at the root path is expected behavior

---

## ✅ Current Status

* ✅ API runs successfully on local machine
* ✅ NuGet restore works normally
* ✅ WeatherForecast endpoint is accessible
* ❌ Docker not implemented yet

---

## 🧠 Key Takeaways (Stage 1)

* Difference between **.NET SDK** and **target framework**
* How `dotnet new` selects SDK versions
* Understanding NuGet Package Source Mapping
* Debugging real-world environment issues
* Basic ASP.NET Core Web API routing

---

## 🔜 Next Stage

**Stage 2 — Dockerizing the ASP.NET Web API**

* Create a Dockerfile
* Build a Docker image
* Run the API inside a container
* Access the endpoint via Docker

---

> This repository is built step-by-step to demonstrate real development workflows rather than a copy-paste tutorial.


---

## 🐳 Stage 2 — Docker

### Goal
Run ASP.NET Web API inside a Docker container.

### Steps Done
- Created `Dockerfile`
- Published application in Release mode
- Built Docker image
- Ran container locally

### Docker Commands
```bash
dotnet publish -c Release
docker build -t weatherapi:1 .
docker run -p 8080:8080 weatherapi:1
```

### Test Endpoint
```
GET http://localhost:8080/weatherforecast
```

### Result
- API runs successfully inside Docker
- Endpoint accessible from browser

> Next stage: improve Dockerfile (multi-stage build)

---

## 🐳 Stage 3 — Docker (Multi-stage Build)

### Goal

Create a cleaner and smaller Docker image using multi-stage build.

### Changes

* Build app inside container
* Runtime image contains only published output

### Dockerfile (Multi-stage)

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build /app/publish .
EXPOSE 8080
ENTRYPOINT ["dotnet", "WeatherApi.dll"]
```

### Commands

```bash
docker build -t weatherapi:2 .
docker run -p 8080:8080 weatherapi:2
```

### Result

* Smaller image size
* No SDK included in final image

> Next stage: Docker Compose

---

## 🐳 Stage 4 — Docker Compose

### Goal
Run the API using Docker Compose (service-based setup).

### Files Added
- `docker-compose.yml`

### docker-compose.yml
```yaml
version: "3.9"
services:
  api:
    image: weatherapi:3
    ports:
      - "8080:8080"
```

### Commands
```bash
docker compose up
```

### Result
- API runs via Compose
- Easier start/stop and future multi-service setup

> Next stage: add another service (e.g., database)
