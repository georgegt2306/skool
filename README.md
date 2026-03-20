# 📘 Skool Internal API Documentation

Documentación técnica de endpoints internos de Skool identificados mediante tráfico autenticado del navegador.

> ⚠️ **Importante**
> - Estos endpoints aparentan ser **internos/no oficiales**.
> - Requieren una **sesión autenticada activa**.
> - Pueden cambiar sin previo aviso.
> - Se recomienda no hardcodear cookies ni tokens sensibles en código fuente.

---

## Índice

1. [Autenticación y headers](#autenticación-y-headers)
2. [GET /affiliates/v2/compensations](#1-get-affiliatesv2compensations)
3. [GET /self/groups](#2-get-selfgroups)
4. [Paginación](#paginación)
5. [Códigos de respuesta esperados](#códigos-de-respuesta-esperados)
6. [Ejemplos de consumo](#ejemplos-de-consumo)
7. [Notas de implementación](#notas-de-implementación)

---

## Autenticación y headers

Ambos endpoints usan el mismo esquema observado en tus pruebas.

### Headers requeridos

| Header | Requerido | Descripción |
|---|---:|---|
| `Cookie` | Sí | Incluye al menos `auth_token`, `client_id` y `aws-waf-token` |
| `x-aws-waf-token` | Sí | Token de protección AWS WAF |
| `Origin` | Sí | Debe enviarse como `https://www.skool.com` |
| `Referer` | Sí | Debe enviarse como `https://www.skool.com/` |

### Ejemplo

```http
Cookie: auth_token=YOUR_AUTH_TOKEN; aws-waf-token=YOUR_AWS_WAF_TOKEN; client_id=YOUR_CLIENT_ID
Origin: https://www.skool.com
Referer: https://www.skool.com/
x-aws-waf-token: YOUR_AWS_WAF_TOKEN
```

### Observaciones

- `auth_token` autentica la sesión del usuario.
- `client_id` parece identificar la sesión o cliente web.
- `aws-waf-token` y `x-aws-waf-token` ayudan a pasar la capa de protección WAF.
- Si alguno expira, la petición puede fallar aunque la URL sea correcta.

---

# 1. GET `/affiliates/v2/compensations`

Obtiene compensaciones por membresía asociadas a afiliados de un grupo.

## Endpoint

```http
GET https://api2.skool.com/affiliates/v2/compensations
```

## Query params

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---:|---|
| `offset` | integer | Sí | Desplazamiento o página lógica de resultados |
| `limit` | integer | Sí | Número máximo de registros por respuesta |
| `type` | string | Sí | Tipo de compensación. Valor observado: `membership` |
| `group_id` | string | Sí | ID del grupo |
| `cancelled` | boolean | No | Filtra miembros cancelados |

## Ejemplo de request

```http
GET /affiliates/v2/compensations?offset=1&limit=30&type=membership&group_id=76fd40a7ca93463a9fc4d568548d789a&cancelled=false
```

## Response body

```json
{
  "membership_compensations": [
    {
      "member_id": "0b21a72cef8b4f73aa18f75c8473f932",
      "total": 1180,
      "group_member_role": "member",
      "is_membership_free_trial": false,
      "referral_status": "Active",
      "user_first_name": "Elias",
      "user_last_name": "Kardossli",
      "username": "elias-kardossli-1623",
      "picture_profile": "https://assets.skool.com/...",
      "picture_bubble": "https://assets.skool.com/..."
    }
  ],
  "has_more": false
}
```

## Campos de respuesta

### `membership_compensations[]`

| Campo | Tipo | Descripción |
|---|---|---|
| `member_id` | string | ID del miembro referido |
| `total` | number | Total acumulado asociado al miembro |
| `group_member_role` | string | Rol del miembro dentro del grupo |
| `is_membership_free_trial` | boolean | Indica si la membresía está en prueba gratuita |
| `referral_status` | string | Estado del referral, por ejemplo `Active` o `Cancelling` |
| `user_first_name` | string | Nombre del usuario |
| `user_last_name` | string | Apellido del usuario |
| `username` | string | Username público o slug del usuario |
| `picture_profile` | string (URL) | Imagen de perfil principal |
| `picture_bubble` | string (URL) | Imagen reducida / miniatura |

### `has_more`

| Campo | Tipo | Descripción |
|---|---|---|
| `has_more` | boolean | Indica si existen más resultados paginables |

## Interpretación funcional

Este endpoint es útil para:

- ranking de afiliados por ingreso generado,
- panel de comisiones,
- análisis de referrals activos,
- automatización de reportes de membresías.

---

# 2. GET `/self/groups`

Devuelve los grupos asociados al usuario autenticado y, opcionalmente, información de miembros.

## Endpoint

```http
GET https://api2.skool.com/self/groups
```

## Query params

| Parámetro | Tipo | Requerido | Descripción |
|---|---|---:|---|
| `offset` | integer | Sí | Desplazamiento o página lógica |
| `limit` | integer | Sí | Cantidad máxima de grupos a retornar |
| `prefs` | boolean | No | Incluye o excluye preferencias |
| `members` | boolean | No | Incluye arreglo `members` en la respuesta |

## Ejemplo de request

```http
GET /self/groups?offset=0&limit=100&prefs=false&members=true
```

## Response body resumido

```json
{
  "groups": [
    {
      "id": "76fd40a7ca93463a9fc4d568548d789a",
      "name": "academiaofm",
      "metadata": {
        "display_name": "Academia OFM",
        "description": "Academia líder...",
        "plan": "pro",
        "privacy": 1,
        "total_members": 302,
        "total_posts": 996
      },
      "created_at": "2024-05-18T11:44:58.548475Z",
      "updated_at": "2026-03-20T16:16:51.700661Z"
    }
  ],
  "has_more": false,
  "members": [
    {
      "id": "c3047bf87b00491ba17256b70d982426",
      "user_id": "bf91ec5254dc460289a797a6e141b4d9",
      "group_id": "76fd40a7ca93463a9fc4d568548d789a",
      "role": "group-moderator",
      "approved_at": "2025-11-11T08:20:53.036066Z"
    }
  ]
}
```

## Campos principales de `groups[]`

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | string | ID del grupo |
| `name` | string | Nombre interno o slug del grupo |
| `metadata` | object | Metadatos extensos del grupo |
| `created_at` | string (datetime) | Fecha de creación |
| `updated_at` | string (datetime) | Fecha de actualización |

## Campos relevantes dentro de `groups[].metadata`

| Campo | Tipo | Descripción |
|---|---|---|
| `display_name` | string | Nombre visible del grupo |
| `description` | string | Descripción pública/interna del grupo |
| `color` | string | Color principal en formato hex |
| `cover_small_url` | string (URL) | Imagen de portada reducida |
| `logo_url` | string (URL) | Logo principal |
| `logo_big_url` | string (URL) | Logo ampliado |
| `favicon_url` | string (URL) | Favicon del grupo |
| `initials` | string | Iniciales del grupo |
| `created_by` | string | ID del creador |
| `owner` | string (JSON serializado) | Información serializada del propietario |
| `member` | string (JSON serializado) | Información serializada de la membresía del usuario actual |
| `membership` | integer | Indicador de membresía |
| `membership_model` | integer | Modelo de membresía |
| `display_price` | string (JSON serializado) | Precio visible y ciclo de cobro |
| `plan` | string | Plan del grupo, por ejemplo `hobby` o `pro` |
| `privacy` | integer | Nivel de privacidad |
| `tabs` | string (JSON serializado) | Tabs y visibilidad de módulos |
| `levels` | string (JSON serializado) | Niveles/gamificación del grupo |
| `labels` | string | IDs de etiquetas separadas por coma |
| `links` | string (JSON serializado) | Links adicionales del grupo |
| `num_courses` | integer | Cantidad de cursos |
| `num_modules` | integer | Cantidad de módulos |
| `total_admins` | integer | Total de administradores |
| `total_members` | integer | Total de miembros |
| `total_online_members` | integer | Miembros online |
| `total_posts` | integer | Total de publicaciones |
| `total_rules` | integer | Total de reglas |
| `payment_status` | string | Estado de pago del grupo o del tenant |
| `billing_email` | string | Email de facturación |
| `invoice_name` | string | Razón social de facturación |
| `invoice_tax_id` | string | Identificación fiscal |
| `invoice_address` | string | Dirección fiscal |
| `afl_percent` | integer | Porcentaje de afiliación |
| `recurring_interval` | string | Intervalo de cobro, por ejemplo `month` |

## Campos principales de `members[]`

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | string | ID del registro de membresía |
| `metadata` | object | Metadatos asociados a la membresía |
| `created_at` | string (datetime) | Fecha de creación |
| `updated_at` | string (datetime) | Última actualización |
| `user_id` | string | ID del usuario |
| `group_id` | string | ID del grupo |
| `role` | string | Rol del usuario en el grupo (`pending`, `member`, `group-moderator`, etc.) |
| `approved_at` | string (datetime) | Fecha de aprobación |
| `last_offline` | string (datetime) | Última desconexión observada |

## Interpretación funcional

Este endpoint sirve para:

- listar los grupos a los que pertenece el usuario,
- obtener metadatos operativos de cada comunidad,
- cruzar la membresía del usuario con cada grupo,
- construir dashboards de comunidades, planes, miembros y actividad.

---

## Paginación

Ambos endpoints exponen `has_more`.

### Estrategia recomendada

1. Ejecutar consulta con `offset` inicial.
2. Procesar resultados.
3. Revisar `has_more`.
4. Si `has_more = true`, incrementar `offset` y repetir.

### Pseudocódigo

```text
offset = 0
has_more = true

while has_more:
    response = request(offset=offset, limit=100)
    procesar(response)
    has_more = response["has_more"]
    offset += 1
```

> Nota: según la API, `offset` podría comportarse como índice de página y no como índice absoluto. Conviene validarlo empíricamente.

---

## Códigos de respuesta esperados

Como documentación inferida, estos son los casos más probables:

| Código | Significado |
|---:|---|
| `200 OK` | Request exitosa |
| `401 Unauthorized` | Sesión inválida o expirada |
| `403 Forbidden` | Token WAF inválido o bloqueo por seguridad |
| `429 Too Many Requests` | Demasiadas solicitudes |
| `5xx` | Error interno del servicio |

---

## Ejemplos de consumo

## Python – compensaciones

```python
import requests

url = "https://api2.skool.com/affiliates/v2/compensations"

params = {
    "offset": 1,
    "limit": 30,
    "type": "membership",
    "group_id": "76fd40a7ca93463a9fc4d568548d789a",
    "cancelled": "false"
}

headers = {
    "Cookie": "auth_token=TU_TOKEN; aws-waf-token=TU_WAF_TOKEN; client_id=TU_CLIENT_ID",
    "Origin": "https://www.skool.com",
    "Referer": "https://www.skool.com/",
    "x-aws-waf-token": "TU_WAF_TOKEN"
}

response = requests.get(url, params=params, headers=headers, timeout=30)
response.raise_for_status()

data = response.json()

for item in data.get("membership_compensations", []):
    print(item["user_first_name"], item["user_last_name"], item["total"])
```

## Python – grupos

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
    "Cookie": "auth_token=TU_TOKEN; aws-waf-token=TU_WAF_TOKEN; client_id=TU_CLIENT_ID",
    "Origin": "https://www.skool.com",
    "Referer": "https://www.skool.com/",
    "x-aws-waf-token": "TU_WAF_TOKEN"
}

response = requests.get(url, params=params, headers=headers, timeout=30)
response.raise_for_status()

data = response.json()

for group in data.get("groups", []):
    metadata = group.get("metadata", {})
    print(
        group.get("id"),
        metadata.get("display_name"),
        metadata.get("plan"),
        metadata.get("total_members")
    )
```

## cURL – compensaciones

```bash
curl --request GET   --url 'https://api2.skool.com/affiliates/v2/compensations?offset=1&limit=30&type=membership&group_id=76fd40a7ca93463a9fc4d568548d789a&cancelled=false'   --header 'Cookie: auth_token=TU_TOKEN; aws-waf-token=TU_WAF_TOKEN; client_id=TU_CLIENT_ID'   --header 'Origin: https://www.skool.com'   --header 'Referer: https://www.skool.com/'   --header 'x-aws-waf-token: TU_WAF_TOKEN'
```

## cURL – grupos

```bash
curl --request GET   --url 'https://api2.skool.com/self/groups?offset=0&limit=100&prefs=false&members=true'   --header 'Cookie: auth_token=TU_TOKEN; aws-waf-token=TU_WAF_TOKEN; client_id=TU_CLIENT_ID'   --header 'Origin: https://www.skool.com'   --header 'Referer: https://www.skool.com/'   --header 'x-aws-waf-token: TU_WAF_TOKEN'
```

---

## Notas de implementación

### 1) Campos JSON serializados

En `GET /self/groups`, varios campos dentro de `metadata` no vienen como objetos JSON nativos, sino como **strings serializados**, por ejemplo:

- `member`
- `owner`
- `tabs`
- `levels`
- `links`
- `display_price`
- `lp_attachments_data`
- `survey`
- `payment_card`
- `retention_video`

Esto implica que, si quieres trabajarlos estructuradamente, debes parsearlos después de leer la respuesta.

### Ejemplo de parseo

```python
import json

raw_owner = group["metadata"].get("owner")
owner = json.loads(raw_owner) if raw_owner else None
```

### 2) Seguridad

- Nunca subas tokens reales al repositorio.
- Usa variables de entorno para cookies/tokens.
- Rota credenciales si las expusiste en un chat, script o commit.

### 3) Automatización

Para ETL o dashboards se recomienda:

- guardar snapshots por fecha,
- normalizar `groups`, `members`, `owners` y `compensations`,
- registrar expiración de tokens,
- implementar reintentos ante `429` y `5xx`.

---

## Estructura sugerida de tablas

### `skool_groups`
- `id`
- `name`
- `display_name`
- `description`
- `plan`
- `privacy`
- `total_members`
- `total_posts`
- `num_courses`
- `num_modules`
- `created_at`
- `updated_at`

### `skool_group_memberships`
- `id`
- `group_id`
- `user_id`
- `role`
- `approved_at`
- `last_offline`
- `created_at`
- `updated_at`

### `skool_affiliate_compensations`
- `member_id`
- `group_id`
- `first_name`
- `last_name`
- `username`
- `group_member_role`
- `referral_status`
- `is_membership_free_trial`
- `total`
- `picture_profile`
- `picture_bubble`
- `snapshot_at`

---

## Disclaimer

Esta documentación fue armada con base en ejemplos reales de request/response compartidos manualmente. Al tratarse de endpoints internos, la estructura final puede variar según la cuenta, permisos, flags o cambios del frontend de Skool.