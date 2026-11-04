---
aliases:
  - n8n
tags:
  - learning
  - dev/ai/automatizations
date: 2026-09-21
---
**Sources**: [n8n docs](https://docs.n8n.io/)

**Related:** [[Artificial Intelligence]], [[Docker]]

---

## Description

Write here...

---

## Key concepts

Write here...

---

## Details

### Environment Variables

#### WEBHOOK_URL

URL where OAuth systems like Google's will call to authorize the usage of its API services.

---

## Examples

Write here...

---

## Utils

### `Docker` Execution

```bash title:bash
docker volume create n8n_data

docker run --name n8n -p 5678:5678 -e GENERIC_TIMEZONE="America/Mexico_City" -e TZ="America/Mexico_City" -e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true -e N8N_RUNNERS_ENABLED=true -e WEBHOOK_URL="https://my-domain.com" -v n8n_data:/home/node/.n8n -d n8nio/n8n
```

---

## Claude Sessions
