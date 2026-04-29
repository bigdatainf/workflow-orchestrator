# TMDB Data Pipeline with N8N, RabbitMQ and MinIO

## Overview

This project implements a simple event-driven data pipeline using:

- **TheMovieDB (TMDB) API** as data source
- **RabbitMQ** for message queuing
- **N8N** for workflow orchestration
- **MinIO (S3-compatible)** as a Data Lake storage layer

The goal is to demonstrate how modern data architectures can be built using loosely coupled components.


## Architecture
TMDB API → N8N (Producer) → RabbitMQ → N8N (Consumer) → MinIO


### Components

- **Producer Workflow (N8N)**
  - Calls TMDB API
  - Extracts movie data
  - Publishes each movie as a message to RabbitMQ

- **RabbitMQ**
  - Acts as a message broker
  - Decouples data ingestion from processing

- **Consumer Workflow (N8N)**
  - Listens to RabbitMQ queue
  - Parses incoming messages
  - Transforms data into structured datasets
  - Stores results in MinIO

- **MinIO (Data Lake)**
  - Stores CSV files
  - Organizes data into logical buckets


## Data Model

The pipeline generates three datasets:

### 1. Movies

One file per movie:

```bash
tmdb-movies/movies_<movie_id>.csv
```


Fields:
- movie_id
- title
- popularity
- vote_average


### 2. Genres

One file per movie, containing multiple rows:

```bash
tmdb-genres/genres_<movie_id>.csv
```


Fields:
- movie_id
- genre_id

Note:
A movie can have multiple genres, so this dataset represents a **one-to-many relationship**.


### 3. Datetimes

One file per movie:

```bash
tmdb-datetimes/datetimes_<movie_id>.csv
```


Fields:
- movie_id
- date
- year
- month
- day


## Docker Deployment

All services are deployed using Docker Compose.

### Services

- N8N (workflow engine)
- RabbitMQ (message broker)
- MinIO (object storage)

### Run the system

```bash
docker compose up -d
```

### Access Points

- N8N:	http://localhost:5678
- RabbitMQ:	http://localhost:15672
- MinIO UI:	http://localhost:9001

### Default Credentials

#### RabbitMQ

- user: guest
- password: guest

#### MinIO

- user: minio
- password: minio123

#### MinIO Configuration

Before running the workflows, create the following buckets:

- tmdb-movies
- tmdb-genres
- tmdb-datetimes

### S3 (MinIO) Settings in N8N

When configuring the S3 node in N8N:

- Endpoint: http://localhost:9000
- Access Key: minio
- Secret Key: minio123
- Region: us-east-1
- Force Path Style: enabled

### Workflows

#### Producer Workflow

##### Steps:

1. Trigger (manual or scheduled)
2. HTTP request to TMDB API
3. Extract movie list
4. Split into individual items
5. Publish each movie to RabbitMQ queue

#### Consumer Workflow

##### Steps:

1. RabbitMQ Trigger (queue listener)
2. Parse message content (JSON)
3. Branch into three parallel transformations:
    - Movies
    - Genres
    - Datetimes
4. Convert JSON to CSV
5. Upload files to MinIO
