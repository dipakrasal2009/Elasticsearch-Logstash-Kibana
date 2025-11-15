
# 🚀 Elasticsearch + Kibana Setup Using Docker

This repository contains instructions to run **Elasticsearch (8.x)** and **Kibana (8.x)** using Docker.  
This README adds deeper explanations, a `docker-compose.yml` example, troubleshooting tips, and cleanup commands.

---

## 🔧 Prerequisites
- Docker installed
- Minimum **4GB RAM**
- Terminal/command-line access

---

## 📘 What are Elasticsearch and Kibana?
**Elasticsearch** is a distributed, RESTful search and analytics engine capable of storing, searching, and analyzing large volumes of data in near real-time.  
**Kibana** is a visualization and exploration tool that connects to Elasticsearch to create dashboards, charts, and to query indexed data.

**Use cases:** log analytics, application monitoring, full-text search, metrics storage, security analytics.

---

## ✅ 1. Create a Docker Network

```bash
docker network create elastic-network
```

Creates a custom network so Elasticsearch and Kibana can communicate internally.

---

## ✅ 2. Pull Images

Elasticsearch:
```bash
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.19.0
```

Kibana:
```bash
docker pull docker.elastic.co/kibana/kibana:8.19.0
```

---

## ✅ 3. Run Elasticsearch Container

```bash
docker run -d --name es01 --net elastic-network \
  -p 9200:9200 -p 9300:9300 \
  -e "discovery.type=single-node" \
  docker.elastic.co/elasticsearch/elasticsearch:8.19.0
```

**Explanation**
- `-d` → Run in background  
- `--name es01` → Container name  
- `--net elastic-network` → Connect to created network  
- `-p 9200:9200` → REST API port  
- `-p 9300:9300` → Node-to-node communication port  
- `-e "discovery.type=single-node"` → Enables single-node cluster mode

---

## ✅ 4. View Elasticsearch Logs

```bash
docker logs es01
```

To find the auto-generated password (first startup):
```bash
docker logs es01 | grep password
```

---

## ✅ 5. Run Kibana Container

```bash
docker run -d --name kib01 --net elastic-network \
  -p 5601:5601 docker.elastic.co/kibana/kibana:8.19.0
```

Kibana UI will be available at `http://localhost:5601`.

---

## ✅ 6. Reset Elasticsearch Password

Interactive reset:
```bash
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

Non-interactive reset (example using `--batch` and random password generated locally):
```bash
NEW_PASS="$(openssl rand -base64 12)"
docker exec -i es01 /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic --batch <<EOF
$NEW_PASS
EOF
# Note: confirm command behaviour for your ES version. Sometimes interactive tools require TTY.
```

---

## ✅ 7. Create Kibana Enrollment Token

```bash
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

Copy the token and paste it into Kibana when it asks for an enrollment token.

---

## ✅ 8. Get Kibana Verification Code

If Kibana prompts for a verification code, run:

```bash
docker exec -it kib01 /usr/share/kibana/bin/kibana-verification-code
```

This prints a short numeric code you paste into the Kibana UI to finish enrollment.

**Common mistakes**
- Running `bin/kibana-verification-code` on the host (not in container) — wrong.
- Using the wrong container name (e.g., `kibana` instead of `kib01`) — wrong path.

---

## 🧩 Docker Compose Example

Create a file `docker-compose.yml` with:

```yaml
version: '3.8'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.19.0
    container_name: es01
    environment:
      - discovery.type=single-node
    networks:
      - elastic-network
    ports:
      - "9200:9200"
      - "9300:9300"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata:/usr/share/elasticsearch/data

  kibana:
    image: docker.elastic.co/kibana/kibana:8.19.0
    container_name: kib01
    networks:
      - elastic-network
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch

networks:
  elastic-network:

volumes:
  esdata:
```

Start both:
```bash
docker-compose up -d
```

---

## 🔒 Security & Credentials Notes

- Elastic 8.x enables security by default (TLS + built-in users). On first startup, Elasticsearch prints a generated `elastic` user password in logs.
- Use the reset-password tool to set your own password.
- Enrollment token is time-limited (typically 30 min). If expired, generate again.
- For production, secure configuration, certificates, and appropriate JVM memory settings are required.

---

## 🛠 Troubleshooting

- **Elasticsearch not starting**: Check logs `docker logs es01`. Common causes: insufficient memory, incorrect Java options, file permissions on data directory.
- **Kibana can't connect**: Ensure both containers are on same network and Elasticsearch is healthy (`curl http://localhost:9200 -u elastic`).
- **Verification token not accepted**: Regenerate enrollment token and verification code; ensure background time sync and that codes are used before expiry.

---

## 🧹 Cleanup Commands

Stop and remove containers:
```bash
docker stop kib01 es01
docker rm kib01 es01
```

Remove network:
```bash
docker network rm elastic-network
```

Remove images (if desired):
```bash
docker image rm docker.elastic.co/elasticsearch/elasticsearch:8.19.0 docker.elastic.co/kibana/kibana:8.19.0
```

Remove volumes (data loss!):
```bash
docker volume rm <volume_name>
```

---

## 📚 References & Further Reading
- Official Elasticsearch docs  
- Official Kibana docs

---

## 📌 Summary Table

| Command | Purpose |
|--------|---------|
| `docker network create elastic-network` | Create network for ES + Kibana |
| `docker pull docker.elastic.co/elasticsearch/elasticsearch:8.19.0` | Download Elasticsearch image |
| `docker run ... es01` | Start Elasticsearch container |
| `docker logs es01` | View Elasticsearch logs |
| `docker pull docker.elastic.co/kibana/kibana:8.19.0` | Download Kibana image |
| `docker run ... kib01` | Start Kibana container |
| `docker exec es01 elasticsearch-reset-password` | Reset elastic user password |
| `docker exec es01 elasticsearch-create-enrollment-token -s kibana` | Create token for Kibana |
| `docker exec kib01 /usr/share/kibana/bin/kibana-verification-code` | Get verification code |

---

# 🎉 Ready
The enhanced `README.md` has been created. Download it below.
