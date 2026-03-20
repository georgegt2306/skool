# 📘 Skool API – Affiliates Compensations

## 🔹 Descripción

Este endpoint permite obtener las **compensaciones por membresía de afiliados** dentro de un grupo específico en Skool.

> ⚠️ Nota: Esta es una API interna (no oficial), basada en cookies de sesión.

---

## 🔹 Endpoint

GET https://api2.skool.com/affiliates/v2/compensations

---

## 🔹 Query Params

| Parámetro     | Tipo    | Requerido | Descripción |
|--------------|--------|----------|------------|
| offset       | int    | ✅       | Número de página |
| limit        | int    | ✅       | Cantidad de registros |
| type         | string | ✅       | Tipo (membership) |
| group_id     | string | ✅       | ID del grupo |
| cancelled    | boolean| ❌       | Filtrar cancelados |

Ejemplo:
GET /affiliates/v2/compensations?offset=1&limit=30&type=membership&group_id=XXX&cancelled=false

---

## 🔹 Headers requeridos

| Header            | Descripción |
|------------------|------------|
| Cookie           | Contiene auth_token y client_id |
| x-aws-waf-token  | Token de seguridad AWS |
| Origin           | https://www.skool.com |
| Referer          | https://www.skool.com/ |

---

## 🔹 Autenticación

- Basada en cookies de sesión
- Requiere:
  - auth_token
  - client_id
  - aws-waf-token

---

## 🔹 Response

Estructura:

{
  "membership_compensations": [...],
  "has_more": false
}

---

## 🔹 Campos

### membership_compensations[]

- member_id: ID del miembro
- total: Total generado
- group_member_role: Rol en el grupo
- is_membership_free_trial: Está en prueba
- referral_status: Estado (Active, Cancelling)
- user_first_name: Nombre
- user_last_name: Apellido
- username: Username
- picture_profile: URL imagen
- picture_bubble: URL miniatura

---

### has_more

- has_more: Indica si hay más páginas

---

## 🔹 Ejemplo de respuesta

{
  "membership_compensations": [
    {
      "member_id": "abc123",
      "total": 1180,
      "referral_status": "Active",
      "user_first_name": "Elias",
      "user_last_name": "Kardossli"
    }
  ],
  "has_more": false
}

---

## 🔹 Paginación

- offset → página
- limit → cantidad
- has_more → continuar

---

## 🔹 Ejemplo en Python

import requests

url = "https://api2.skool.com/affiliates/v2/compensations"

params = {
    "offset": 1,
    "limit": 30,
    "type": "membership",
    "group_id": "TU_GROUP_ID",
    "cancelled": "false"
}

headers = {
    "Cookie": "auth_token=TU_TOKEN",
    "x-aws-waf-token": "TU_WAF",
    "Origin": "https://www.skool.com",
    "Referer": "https://www.skool.com/"
}

response = requests.get(url, params=params, headers=headers)
data = response.json()

for user in data["membership_compensations"]:
    print(user["user_first_name"], user["total"])

---

## 🔹 Consideraciones

⚠️ Esta API:
- No es pública
- Puede cambiar sin aviso
- Depende de sesión activa
- Puede bloquear automatización
