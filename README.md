
# 🚀 Elasticsearch + Kibana Setup Using Docker

This guide explains how to install, configure, and run **Elasticsearch** and **Kibana** using Docker.  
It includes network setup, container commands, password reset steps, enrollment token creation,  
and verification code retrieval.

---

## 🔧 Prerequisites
- Docker installed
- Minimum **4GB RAM**
- Terminal/command-line access

---

# ✅ 1. Create a Docker Network

```bash
docker network create elastic-network
```

Creates a custom network so Elasticsearch and Kibana can communicate internally.

---

# ✅ 2. Pull Elasticsearch Image

```bash
docker pull docker.elastic.co/elasticsearch/elasticsearch:8.19.0
```

Downloads Elasticsearch version **8.19.0** from Elastic’s registry.

---

# ✅ 3. Run Elasticsearch Container

```bash
docker run -d --name es01 --net elastic-network \
-p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
docker.elastic.co/elasticsearch/elasticsearch:8.19.0
```

### **Explanation**
- `-d` → Run in background  
- `--name es01` → Container name  
- `--net elastic-network` → Connect to created network  
- `-p 9200:9200` → REST API port  
- `-p 9300:9300` → Node-to-node communication port  
- `discovery.type=single-node` → Enables single-node cluster mode  
- Image → `elasticsearch:8.19.0`

---

# ✅ 4. View Elasticsearch Logs

```bash
docker logs es01
```

Shows full Elasticsearch logs.

```bash
docker logs es01 | grep passwo
```

Finds the auto-generated `elastic` user password.

---

# ✅ 5. Pull Kibana Image

```bash
docker pull docker.elastic.co/kibana/kibana:8.19.0
```

Downloads Kibana version **8.19.0**.

---

# ✅ 6. Run Kibana Container

```bash
docker run -d --name kib01 --net elastic-network \
-p 5601:5601 docker.elastic.co/kibana/kibana:8.19.0
```

### **Explanation**
- `--name kib01` → Container name  
- `--net elastic-network` → Connects to the same network as Elasticsearch  
- `-p 5601:5601` → Exposes Kibana UI  
- Image → `kibana:8.19.0`

---

# ✅ 7. Reset Elasticsearch Password

```bash
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

Resets the password for user **elastic**.

---

# ✅ 8. Create Kibana Enrollment Token

```bash
docker exec -it es01 /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana
```

Generates the enrollment token required for Kibana to connect to Elasticsearch.

---

# ✅ 9. Get Kibana Verification Code

Use the correct command:

```bash
docker exec -it kib01 /usr/share/kibana/bin/kibana-verification-code
```

Outputs the 6-digit code required by Kibana UI.

---

# 📘 Summary Table

| Command | Purpose |
|--------|---------|
| `docker network create elastic-network` | Create network for ES + Kibana |
| `docker pull elasticsearch:8.19.0` | Download Elasticsearch image |
| `docker run ... es01` | Start Elasticsearch container |
| `docker logs es01` | View Elasticsearch logs |
| `docker logs es01 | grep passwo` | Find auto password |
| `docker pull kibana:8.19.0` | Download Kibana image |
| `docker run ... kib01` | Start Kibana container |
| `docker exec es01 elasticsearch-reset-password` | Reset elastic user password |
| `docker exec es01 elasticsearch-create-enrollment-token` | Create token for Kibana |
| `docker exec kib01 kibana-verification-code` | Get verification code |

---

# 🎉 All Done!
Access Kibana at:

```
http://localhost:5601
```

Login with:  
**Username:** `elastic`  
**Password:** (your new or auto-generated password)  
Paste the enrollment token → Enter verification code → Kibana is ready!
