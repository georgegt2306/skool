# 📘 Skool API – Affiliates Compensations & Groups

## 🔹 Descripción

Documentación de APIs internas de Skool para:
- Compensaciones de afiliados
- Listado de grupos del usuario

---

# 🔹 1. Affiliates Compensations

## Endpoint
GET https://api2.skool.com/affiliates/v2/compensations

## Query Params
- offset (int)
- limit (int)
- type (membership)
- group_id (string)
- cancelled (boolean)

## Response
- membership_compensations[]
- has_more

---

# 🔹 2. User Groups

## Endpoint
GET https://api2.skool.com/self/groups

## Query Params

| Parámetro | Tipo | Descripción |
|----------|------|------------|
| offset   | int  | Página |
| limit    | int  | Cantidad |
| prefs    | bool | Preferencias |
| members  | bool | Incluir miembros |

### Ejemplo

GET /self/groups?offset=0&limit=100&prefs=false&members=true

---

## 🔹 Response

### Estructura

{
  "groups": [...],
  "has_more": false,
  "members": [...]
}

---

## 🔹 Campos principales

### groups[]

- id: ID del grupo
- name: nombre interno
- metadata:
  - display_name: nombre visible
  - description: descripción
  - total_members: número de miembros
  - total_posts: publicaciones
  - plan: tipo de plan
  - privacy: privacidad
  - logo_url: logo
  - cover_small_url: portada

---

### members[]

- id: ID del registro
- user_id: ID usuario
- group_id: ID grupo
- role: rol (member, pending, moderator)
- approved_at: fecha aprobación

---

## 🔹 Ejemplo de uso en Python

```python
import requests

url = "https://api2.skool.com/self/groups"

params = {
    "offset": 0,
    "limit": 100,
    "prefs": "false",
    "members": "true"
}

headers = {
    "Cookie": "auth_token=TU_TOKEN",
    "x-aws-waf-token": "TU_WAF",
    "Origin": "https://www.skool.com",
    "Referer": "https://www.skool.com/"
}

response = requests.get(url, params=params, headers=headers)
data = response.json()

for group in data["groups"]:
    print(group["metadata"]["display_name"], group["metadata"]["total_members"])
```

---

## 🔹 Headers (ambas APIs)

- Cookie (auth_token, client_id)
- x-aws-waf-token
- Origin
- Referer

---

## 🔹 Consideraciones

⚠️ APIs internas:
- No oficiales
- Pueden cambiar
- Requieren sesión activa
- Pueden bloquear automatización

---

## 🚀 Ideas de uso

- Dashboard de afiliados
- Dashboard de comunidades
- BI de ingresos y membresías
- Automatización ETL
