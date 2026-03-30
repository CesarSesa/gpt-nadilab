# Our Updated Unified DB — Nadistudio Scaffold

> **Database Schema Reference**  
> **Project:** Castle Nadistudio v6.3.1  
> **Updated:** 2026-03-27  
> **Total Tables:** 40+ | **Core Tenant Tables:** 23 FKs → `tenants`

---

## CONVENCIONES

- **PK** = Primary Key
- **FK** = Foreign Key → `table.column`
- **CHECK** = Restricciones de valores permitidos
- **Default** = Valor por defecto
- **RLS** = Row Level Security habilitado (todas las tablas)

---

## 🏛️ CORE — El Núcleo del Sistema

### `tenants`
> Tiendas/clientes del SaaS. Centro de gravedad: 23 tablas referencian aquí.

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `name` | text | — | Nombre del negocio |
| `slug` | text | UNIQUE | Subdominio: {slug}.nadistudio.cl |
| `admin_email` | text | nullable | — |
| `admin_code` | text | nullable | Código de acceso temporal |
| `settings` | jsonb | {} | Config variada del tenant |
| `is_active` | boolean | true | — |
| `plan` | text | 'free' | free \| starter \| pro \| enterprise |
| `rubro_type` | varchar | 'retail-clothing' | **CHECK:** retail-clothing \| real-estate \| hardware \| pharmacy \| food |
| `config_json` | jsonb | {ui:{}, limits:{}, capabilities:[]} | Capacidades y límites |
| `theme_config` | jsonb | nullable | Colores, tema visual |
| `custom_domain` | text | nullable, UNIQUE | Dominio propio |
| `monthly_token_limit` | int | 1000000 | Límite IA (~$2 USD) |
| `contact_phone` | text | nullable | — |
| `tenant_rut` | text | nullable | RUT fiscal |
| `business_legal_name` | text | nullable | Razón social |
| `social_links` | jsonb | {tiktok:null, ...} | Redes sociales |
| `legal_pages` | jsonb | {faq:null, tos:null, privacy:null} | Páginas legales |
| `admin_password_hash` | text | nullable | bcrypt hash |
| `password_version` | int | 1 | 1=legacy, 2=hashed |
| `captadora_enabled` | boolean | false | Captadora de propiedades |
| `telegram_bot_token` | text | nullable | Bot dedicado del tenant |
| `telegram_bot_username` | text | nullable | — |
| `activation_code` | text | nullable, UNIQUE | Código de activación INMO-XXXXXX |
| `created_at` | timestamptz | now() | — |

**Tablas que referencian `tenants.id`:**
`products`, `properties`, `product_drafts`, `property_drafts`, `categories`, `suppliers`, `clients`, `users`, `sales`, `customers`, `expenses`, `cash_registers`, `cash_movements`, `telegram_users`, `active_sessions`, `session_photos`, `media_queue`, `entity_images`, `photo_albums`, `operations_history`, `tenant_features`, `system_logs`, `property_inquiries`, `rubro_configs`, `business_context`, `tenant_storage_usage`, `admin_login_attempts`, `captadora_submissions`

---

### `users`
> Usuarios internos del tenant (admin/employee)

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `name` | text | — | — |
| `email` | text | nullable, UNIQUE | — |
| `pin` | text | nullable | PIN numérico |
| `role` | text | 'employee' | **CHECK:** admin \| employee |
| `telegram_user_id` | bigint | nullable, UNIQUE | ID Telegram |
| `is_active` | boolean | true | — |
| `created_at` | timestamptz | now() | — |

**Referenciada por:** `sales.seller_id`, `cash_registers.opened_by/closed_by`, `cash_movements.created_by`, `expenses.created_by`

---

### `telegram_users`
> Usuarios de Telegram vinculados a un tenant

| Columna | Tipo | Default | Descripción |
|---------|------|---------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `telegram_user_id` | bigint | — | ID único de Telegram |
| `tenant_id` | uuid | FK → tenants.id | — |
| `role` | text | 'user' | user \| admin (contextual) |
| `telegram_username` | text | nullable | @username |
| `activated_at` | timestamptz | now() | — |

---

## 👕 VERTICAL: RETAIL (Ropa)

### `products`
> Productos del catálogo retail

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `name` | text | — | Nombre del producto |
| `description` | text | nullable | — |
| `slug` | text | nullable | URL-friendly |
| `sale_price` | int | 0 | Precio venta |
| `cost_price` | int | 0 | Precio costo |
| `offer_price` | int | nullable | Precio oferta |
| `stock_quantity` | int | 0 | Stock actual |
| `min_stock` | int | 2 | Stock mínimo alerta |
| `category` | text | nullable | Categoría textual |
| `brand` | text | nullable | Marca |
| `size` | text | nullable | Talla |
| `color` | text | nullable | Color |
| `style` | text | nullable | Estilo |
| `sku` | text | nullable | Código SKU |
| `supplier` | text | nullable | Proveedor (legacy) |
| `is_active` | boolean | true | — |
| `is_featured` | boolean | false | Destacado |
| `is_visible_in_store` | boolean | true | Visible en tienda |
| `category_id` | uuid | nullable, FK → categories.id | — |
| `supplier_id` | uuid | nullable, FK → suppliers.id | — |
| `images` | text[] | {} | Array URLs imágenes |
| `status` | varchar | 'active' | active \| archived |
| `deleted_at` | timestamp | nullable | Soft delete |
| `created_at` | timestamptz | now() | — |
| `updated_at` | timestamptz | now() | — |

---

### `categories`
> Categorías de productos

| Columna | Tipo | Default | Descripción |
|---------|------|---------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `name` | text | — | — |
| `slug` | text | — | — |
| `description` | text | nullable | — |
| `sort_order` | int | 0 | Orden visual |
| `is_active` | boolean | true | — |
| `created_at` | timestamptz | now() | — |

---

### `suppliers`
> Proveedores de productos

| Columna | Tipo | Default | Descripción |
|---------|------|---------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `name` | text | — | Nombre proveedor |
| `contact_name` | text | nullable | Contacto |
| `phone` | text | nullable | — |
| `email` | text | nullable | — |
| `notes` | text | nullable | — |
| `is_active` | boolean | true | — |
| `created_at` | timestamptz | now() | — |
| `updated_at` | timestamptz | now() | — |

---

### `product_drafts`
> Pipeline: fotos → productos (M1-M9)

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `created_by_telegram_id` | text | nullable | Quién envió las fotos |
| `created_by_telegram_username` | text | nullable | — |
| `batch_id` | text | — | Agrupación de fotos |
| `status` | text | 'pending' | **CHECK:** pending \| editing \| ready \| pending_analysis \| auto_detected \| in_review \| ready_for_approval \| approved \| batch_reviewed \| ready_to_publish \| published \| discarded \| archived |
| **DETECTED (por IA)** |
| `detected_name` | text | nullable | — |
| `detected_category` | text | nullable | — |
| `detected_brand` | text | nullable | — |
| `detected_color` | text | nullable | — |
| `detected_color_secondary` | text | nullable | — |
| `detected_size` | text | nullable | — |
| `detected_quantity` | int | 1 | — |
| `detected_condition` | text | nullable | — |
| `detected_has_tags` | boolean | nullable | — |
| `detected_notes` | text | nullable | — |
| `detected_price` | int | nullable | — |
| `detected_description` | text | nullable | — |
| **FINAL (editado por humano)** |
| `final_name` | text | nullable | — |
| `final_price` | int | nullable | — |
| `final_stock` | int | nullable | — |
| `final_supplier` | text | nullable | — |
| `final_sku` | text | nullable | — |
| `final_description` | text | nullable | — |
| **METADATA** |
| `confidence_score` | numeric | nullable | 0-1 confianza IA |
| `photos_analysis` | text | nullable | Raw analysis IA |
| `image_urls` | text[] | nullable | URLs fotos |
| `product_group_key` | text | nullable | Agrupación visual |
| `published_product_id` | uuid | nullable, FK → products.id | Producto publicado |
| **TIMESTAMPS** |
| `auto_detected_at` | timestamptz | nullable | Cuando IA procesó |
| `reviewed_at` | timestamptz | nullable | Cuando admin revisó |
| `approved_at` | timestamptz | nullable | Aprobado para publicar |
| `archived_at` | timestamptz | nullable | — |
| `archived_reason` | text | nullable | — |
| `archived_source` | text | nullable | De dónde se archivó |
| `created_at` | timestamptz | now() | — |
| `updated_at` | timestamptz | now() | — |

---

## 🏠 VERTICAL: REAL-ESTATE (Inmobiliaria)

### `properties`
> Propiedades publicadas

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `slug` | text | — | URL única |
| `title` | text | — | Título publicación |
| `property_type` | text | — | **CHECK:** house \| apartment \| office \| land |
| `operation_type` | text | — | **CHECK:** sale \| rent |
| `commune` | text | — | Comuna |
| `region` | text | 'Metropolitana' | Región |
| `address` | text | nullable | Dirección exacta |
| `price` | int | nullable, CHECK ≥0 | Precio en moneda original |
| `price_currency` | text | 'CLP' | **CHECK:** CLP \| UF \| USD |
| `price_clp` | int | nullable | Precio en CLP calculado |
| `common_expenses` | int | nullable | Gastos comunes |
| `bedrooms` | int | nullable, CHECK ≥0 | Dormitorios |
| `bathrooms` | int | nullable, CHECK ≥0 | Baños |
| `total_area` | int | nullable, CHECK ≥0 | m² terreno |
| `built_area` | int | nullable, CHECK ≥0 | m² construidos |
| `year_built` | int | nullable | Año construcción |
| `orientation` | text[] | nullable | Norte, Sur, etc. |
| `amenities` | text[] | nullable | Amenidades |
| `security_features` | text[] | nullable | Seguridad |
| `parking_count` | int | 0 | Estacionamientos |
| `parking_types` | text[] | nullable | Cubierto, descubierto |
| `storage_count` | int | 0 | Bodegas |
| `status` | text | 'draft' | **CHECK:** draft \| published \| archived \| reserved |
| `featured` | boolean | false | Destacada |
| `description` | text | nullable | Descripción larga |
| `description_short` | text | nullable | Resumen |
| `views_count` | int | 0 | Vistas página |
| `contact_clicks` | int | 0 | Clicks "contactar" |
| `deleted_at` | timestamptz | nullable | Soft delete |
| `created_at` | timestamptz | now() | — |
| `updated_at` | timestamptz | now() | — |

---

### `property_drafts`
> Pipeline inmobiliario (más complejo que retail)

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| **ORIGEN** |
| `created_by_telegram_id` | bigint | nullable | Telegram |
| `created_by_telegram_username` | text | nullable | — |
| `whatsapp_sender` | text | nullable | WhatsApp origen |
| `whatsapp_message_id` | text | nullable | — |
| `raw_input_text` | text | nullable | Texto crudo recibido |
| `raw_image_urls` | text[] | nullable | Fotos originales |
| `source` | text | 'manual' | manual \| telegram \| whatsapp |
| `source_metadata` | jsonb | {} | Metadata origen |
| **DETECTED (IA)** |
| `detected_property_type` | text | nullable | — |
| `detected_operation` | text | nullable | sale \| rent |
| `detected_bedrooms`, `detected_bathrooms` | int | nullable | — |
| `detected_amenities` | text[] | nullable | — |
| `detected_price` | numeric | nullable | — |
| `detected_currency` | text | nullable | CLP \| UF \| USD |
| `detected_price_clp` | int | nullable | — |
| `detected_commune`, `detected_region`, `detected_address` | text | nullable | — |
| `detected_year_built` | int | nullable | — |
| `detected_condition` | text | nullable | — |
| `detected_total_area`, `detected_built_area` | int | nullable | m² |
| `detected_parking_count` | int | nullable | — |
| `detected_orientation` | text[] | nullable | — |
| `detected_parking_types` | text[] | nullable | — |
| `detected_security_features` | text[] | nullable | — |
| `detected_common_expenses` | int | nullable | Gastos comunes |
| **DESCRIPCIONES IA** |
| `description_tecnica` | text | nullable | Descripción técnica |
| `description_puntos_fuertes` | text | nullable | Puntos fuertes |
| `description_plusvalia` | text | nullable | Plusvalía |
| `description_full` | text | nullable | Descripción completa |
| `amenities_description` | text | nullable | Amenidades detalladas |
| `security_description` | text | nullable | Seguridad detallada |
| **SUGERENCIAS** |
| `suggested_title` | text | nullable | Título sugerido |
| `suggested_slug` | text | nullable | URL sugerida |
| **VALIDACIÓN** |
| `confidence_score` | numeric | nullable, CHECK 0-1 | Confianza IA |
| `missing_data` | text[] | nullable | Datos faltantes |
| `missing_critical` | text[] | nullable | Datos críticos faltantes |
| `edited_data` | jsonb | {} | Ediciones humanas |
| **WORKFLOW** |
| `status` | text | 'pending_analysis' | **CHECK:** draft \| pending_analysis \| auto_detected \| in_review \| ready_for_approval \| approved \| rejected \| archived \| error |
| `published_property_id` | uuid | nullable, FK → properties.id | Propiedad publicada |
| **NOTIFICACIONES** |
| `email_sent` | boolean | false | Email admin enviado |
| `email_sent_at` | timestamptz | nullable | — |
| `reminder_sent` | boolean | false | Recordatorio enviado |
| **TIMESTAMPS** |
| `auto_detected_at` | timestamptz | nullable | IA procesó |
| `reviewed_at` | timestamptz | nullable | Admin revisó |
| `reviewed_by` | uuid | nullable | Quién revisó |
| `approved_at` | timestamptz | nullable | Aprobado |
| `approved_by` | uuid | nullable | Quién aprobó |
| `rejected_at` | timestamptz | nullable | Rechazado |
| `rejected_reason` | text | nullable | Por qué |
| `archived_at` | timestamptz | nullable | Archivado |
| `archived_reason` | text | nullable | — |
| `archived_source` | text | nullable | Origen del archivo |
| `created_at` | timestamptz | now() | — |
| `updated_at` | timestamptz | now() | — |

---

### `clients`
> CRM: Leads y clientes inmobiliarios

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `name` | text | — | Nombre |
| `phone` | text | nullable | Teléfono |
| `email` | text | nullable | Email |
| `status` | text | 'new' | **CHECK:** new \| contacted \| visit_scheduled \| negotiating \| closed_won \| closed_lost |
| `source` | text | 'manual' | **CHECK:** telegram \| whatsapp \| web \| phone \| manual \| referral |
| `budget_min`, `budget_max` | int | nullable | Rango presupuesto |
| `preferred_location` | text | nullable | Zona preferida |
| `property_type` | text | nullable | **CHECK:** house \| apartment \| office \| land \| commercial |
| `notes` | text | nullable | Notas |
| `assigned_to` | uuid | nullable, FK → auth.users | Agente asignado |
| `last_contact_at` | timestamptz | nullable | Último contacto |
| `deleted_at` | timestamp | nullable | Soft delete |
| `created_at` | timestamptz | now() | — |
| `updated_at` | timestamptz | now() | — |

---

### `property_inquiries`
> Consultas sobre propiedades específicas

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `property_id` | uuid | FK → properties.id | Propiedad consultada |
| `client_id` | uuid | nullable, FK → clients.id | Cliente (si existe) |
| `contact_name` | text | nullable | Nombre quien consulta |
| `contact_phone` | text | nullable | — |
| `contact_email` | text | nullable | — |
| `contact_telegram_id` | bigint | nullable | — |
| `message` | text | nullable | Mensaje |
| `source` | text | 'telegram' | **CHECK:** telegram \| whatsapp \| web \| phone |
| `status` | text | 'new' | **CHECK:** new \| contacted \| visit_scheduled \| negotiating \| closed_won \| closed_lost \| converted |
| `assigned_to` | uuid | nullable | Agente asignado |
| `notes` | text | nullable | — |
| `created_at` | timestamptz | now() | — |
| `updated_at` | timestamptz | now() | — |

---

## 💰 RETAIL: Punto de Venta

### `customers`
> Clientes finales retail (compradores)

| Columna | Tipo | Default | Descripción |
|---------|------|---------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `name` | text | — | — |
| `phone`, `email`, `instagram` | text | nullable | — |
| `preferred_size`, `preferred_color` | text | nullable | Preferencias |
| `notes` | text | nullable | — |
| `total_purchases` | int | 0 | # compras |
| `total_purchase_amount` | int | 0 | $ total |
| `last_purchase` | date | nullable | — |
| `created_at` | timestamptz | now() | — |
| `updated_at` | timestamptz | now() | — |

---

### `sales`
> Ventas con items desglosados

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `customer_id` | uuid | nullable, FK → customers.id | Cliente |
| `seller_id` | uuid | nullable, FK → users.id | Vendedor |
| `subtotal` | int | — | Antes descuentos |
| `discount` | int | 0 | Descuento |
| `total_amount` | int | 0 | Total final |
| `tax` | int | 0 | Impuestos |
| `payment_method` | text | nullable | **CHECK:** efectivo \| tarjeta \| transferencia \| otro |
| `item_count` | int | 0 | # items |
| `status` | text | 'completed' | completed \| ... |
| `operation_id` | text | nullable | ID operación legible |
| `sale_timestamp` | timestamptz | now() | — |
| `date`, `sale_date` | date | CURRENT_DATE | — |
| `notes` | text | nullable | — |
| `created_at` | timestamptz | now() | — |
| `updated_at` | timestamptz | now() | — |

---

### `sale_items`
> Items individuales de una venta

| Columna | Tipo | Default | Descripción |
|---------|------|---------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `sale_id` | uuid | FK → sales.id | — |
| `product_id` | uuid | FK → products.id | — |
| `quantity` | int | 1 | — |
| `unit_price` | int | — | Precio unitario |
| `total` | int | — | quantity × price |
| `tenant_id` | uuid | FK → tenants.id | — |
| `created_at` | timestamptz | now() | — |

---

### `expenses`
> Gastos del negocio

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `description` | text | — | — |
| `amount` | int | — | — |
| `date` | date | CURRENT_DATE | — |
| `category` | text | 'otros' | **CHECK:** mercaderia \| local \| servicios \| marketing \| otros |
| `created_by` | uuid | nullable, FK → users.id | — |
| `created_at` | timestamptz | now() | — |
| `updated_at` | timestamptz | now() | — |

---

### `cash_registers`
> Caja diaria (apertura/cierre)

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `date` | date | CURRENT_DATE | — |
| `opened_at` | timestamptz | nullable | Apertura |
| `opening_amount` | int | 0 | Fondo inicial |
| `opened_by` | uuid | nullable, FK → users.id | — |
| `closed_at` | timestamptz | nullable | Cierre |
| `closing_amount` | int | nullable | Monto esperado |
| `actual_amount` | int | nullable | Monto real contado |
| `difference` | int | nullable | Diferencia |
| `closed_by` | uuid | nullable, FK → users.id | — |
| `closing_notes` | text | nullable | — |
| `status` | text | 'abierta' | **CHECK:** abierta \| cerrada |
| `created_at` | timestamptz | now() | — |
| `updated_at` | timestamptz | now() | — |

---

### `cash_movements`
> Movimientos de caja (ventas, gastos, ajustes)

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `cash_register_id` | uuid | nullable, FK → cash_registers.id | Caja |
| `type` | text | nullable | **CHECK:** venta \| gasto \| retiro \| ajuste \| apertura |
| `amount` | int | — | Monto |
| `description` | text | nullable | — |
| `sale_id` | uuid | nullable, FK → sales.id | Si es venta |
| `expense_id` | uuid | nullable, FK → expenses.id | Si es gasto |
| `created_by` | uuid | nullable, FK → users.id | — |
| `created_at` | timestamptz | now() | — |
| `updated_at` | timestamptz | now() | — |

---

## 🤖 BOT WORKFLOW

### `active_sessions`
> Sesiones conversacionales activas

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `user_id` | bigint | — | Telegram user ID |
| `chat_id` | bigint | — | Telegram chat ID |
| `session_type` | text | — | Tipo de flujo |
| `step` | text | — | Paso actual |
| `data` | jsonb | {} | Datos temporales |
| `expires_at` | timestamptz | — | Expiración |
| `rubro_context` | text | 'retail-clothing' | **CHECK:** retail-clothing \| real-estate \| generic |
| `created_at` | timestamptz | now() | — |
| `updated_at` | timestamptz | now() | — |

**Referenciada por:** `session_photos.session_id`

---

### `session_photos`
> Fotos en sesión activa (temporal)

| Columna | Tipo | Default | Descripción |
|---------|------|---------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `session_id` | uuid | nullable, FK → active_sessions.id | — |
| `file_id` | text | — | Telegram file ID |
| `file_url` | text | nullable | URL temporal |
| `created_at` | timestamptz | now() | — |

---

## 📸 SISTEMA DE MEDIOS (Yamato Media System)

### `media_queue`
> Cola de recepción universal — TODA entrada pasa por aquí

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `source_type` | text | — | **CHECK:** telegram \| whatsapp \| web \| api |
| `source_message_id` | text | nullable | ID mensaje origen |
| `source_chat_id` | bigint | nullable | Chat origen |
| `source_user_id` | bigint | nullable | Usuario origen |
| `source_media_group_id` | text | nullable | Grupo de fotos Telegram |
| `content_type` | text | — | **CHECK:** text_only \| photos_text \| voice \| document \| mixed |
| `raw_text` | text | nullable | Texto recibido |
| `raw_caption` | text | nullable | Caption fotos |
| `status` | text | 'pending' | **CHECK:** pending \| accumulating \| processing \| completed \| failed \| expired |
| `processing_attempts` | int | 0 | Intentos |
| `error_message` | text | nullable | — |
| `rubro_context` | text | nullable | **CHECK:** retail-clothing \| real-estate \| generic \| auto-detect |
| `detected_intent` | text | nullable | Intento detectado |
| `file_id` | text | nullable | Archivo principal |
| `file_url` | text | nullable | URL archivo |
| `mime_type` | text | nullable | — |
| `file_size` | int | nullable | Bytes |
| `received_at` | timestamptz | now() | — |
| `processing_started_at` | timestamptz | nullable | — |
| `completed_at` | timestamptz | nullable | — |
| `expires_at` | timestamptz | now() + 24h | Expiración |
| `created_at` | timestamptz | now() | — |

**Referenciada por:** `media_queue_photos.queue_id`, `entity_images.source_queue_id`

---

### `media_queue_photos`
> Fotos individuales vinculadas a media_queue

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `queue_id` | uuid | FK → media_queue.id | — |
| `sort_order` | int | — | Orden preservado |
| `file_id` | text | — | Telegram file ID (~24h válido) |
| `file_url` | text | nullable | — |
| `storage_path` | text | nullable | `{tenant_id}/temp/{queue_id}/{sort_order}.jpg` |
| `width`, `height` | int | nullable | Dimensiones |
| `file_size` | int | nullable | Bytes |
| `mime_type` | text | nullable | — |
| `status` | text | 'pending' | **CHECK:** pending \| uploaded \| processed \| error |
| `error_message` | text | nullable | — |
| `created_at` | timestamptz | now() | — |

---

### `entity_images`
> Imágenes publicadas (productos/propiedades)

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `entity_type` | text | — | **CHECK:** product \| property \| draft_product \| draft_property |
| `entity_id` | uuid | — | ID de la entidad |
| `storage_path` | text | — | Ruta en Supabase Storage |
| `public_url` | text | — | URL pública |
| `file_size_bytes` | int | nullable | — |
| `mime_type` | text | nullable | — |
| `status` | text | 'active' | **CHECK:** active \| archived \| orphan \| processing \| error |
| `sort_order` | int | 0 | 0 = imagen principal |
| `source_queue_id` | uuid | nullable, FK → media_queue.id | Trazabilidad |
| `processed_by` | text | nullable | kimi-vision-v1, manual-upload, etc. |
| `metadata` | jsonb | {} | EXIF, detección IA, confianza |
| `created_at` | timestamptz | now() | — |
| `updated_at` | timestamptz | now() | — |

---

### `photo_albums` (Legacy)
> Álbumes de fotos (antiguo, aún presente)

| Columna | Tipo | Default | Descripción |
|---------|------|---------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `media_group_id` | text | — | Telegram media group |
| `tenant_id` | uuid | FK → tenants.id | — |
| `user_id`, `chat_id` | bigint | — | Telegram IDs |
| `photos_count`, `expected_count` | int | 0 / nullable | — |
| `status` | text | 'collecting' | — |
| `expires_at` | timestamptz | now() + 10min | — |
| `created_at` | timestamptz | now() | — |

---

### `photo_album_items` (Legacy)

| Columna | Tipo | Default | Descripción |
|---------|------|---------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `album_id` | uuid | FK → photo_albums.id | — |
| `file_id`, `file_url`, `mime_type`, `size` | | | — |
| `created_at` | timestamptz | now() | — |

---

## ⚙️ CONFIGURACIÓN & FEATURE FLAGS

### `tenant_features`
> Feature flags por tenant

| Columna | Tipo | Default | Descripción |
|---------|------|---------|-------------|
| `tenant_id` | uuid | PK, FK → tenants.id | — |
| `voice_input` | boolean | false | Entrada por voz |
| `photo_analysis` | boolean | true | Análisis de fotos |
| `batch_upload` | boolean | true | Subida batch |
| `rectification` | boolean | true | Rectificación ventas |
| `drafts` | boolean | true | Sistema de drafts |
| `analytics` | boolean | false | Estadísticas |
| `whatsapp_integration` | boolean | false | WhatsApp bot |
| `max_products` | int | 1000 | Límite productos |
| `max_properties` | int | 500 | Límite propiedades |
| `max_photos_per_batch` | int | 25 | Fotos por batch |
| `max_team_members` | int | 5 | Usuarios permitidos |
| `updated_at` | timestamptz | now() | — |

---

### `rubro_configs`
> Config específica por rubro/tenant

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `rubro_type` | text | — | **CHECK:** retail-clothing \| real-estate |
| `config` | jsonb | {} | Config arbitraria |
| `created_at` | timestamptz | now() | — |
| `updated_at` | timestamptz | now() | — |

---

## 📝 LOGGING, AUDIT & MEMORIA

### `system_logs`
> Logs operacionales del sistema

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | nullable, FK → tenants.id | — |
| `session_id` | text | nullable | — |
| `user_id` | bigint | nullable | Telegram ID |
| `level` | text | — | **CHECK:** debug \| info \| warn \| error \| critical |
| `component` | text | — | **CHECK:** bot \| processor \| storage \| ai \| auth \| webhook \| db \| system |
| `operation` | text | — | Operación |
| `message` | text | — | Mensaje |
| `metadata` | jsonb | {} | Datos extras |
| `error_stack` | text | nullable | Stack trace |
| `error_message` | text | nullable | — |
| `duration_ms` | int | nullable | Tiempo ejecución |
| `environment` | text | 'unknown' | prod \| dev |
| `vercel_deployment` | text | nullable | Deploy ID |
| `created_at` | timestamptz | now() | — |

---

### `operations_history`
> Historial de operaciones (audit trail)

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `user_id` | bigint | nullable | Quién ejecutó |
| `operation_type` | text | — | **CHECK:** SELL \| RESTOCK \| CREATE \| UPDATE \| DELETE \| RECTIFY \| UPLOAD \| PUBLISH \| ARCHIVE |
| `entity_type` | text | — | **CHECK:** product \| property \| sale \| expense \| draft \| inquiry |
| `entity_id` | uuid | nullable | ID afectado |
| `previous_state` | jsonb | nullable | Estado anterior |
| `new_state` | jsonb | nullable | Estado nuevo |
| `delta` | jsonb | nullable | Diferencia |
| `source` | text | 'telegram' | **CHECK:** telegram \| whatsapp \| web \| api |
| `ip_address` | inet | nullable | — |
| `user_agent` | text | nullable | — |
| `created_at` | timestamptz | now() | — |
| `processed_at` | timestamptz | nullable | — |

---

### `project_logs`
> Memoria del proyecto (decisiones, blockers)

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `created_at` | timestamptz | now() | — |
| `project` | text | — | Nombre proyecto |
| `phase` | text | nullable | Fase |
| `task` | text | — | Tarea |
| `reason` | text | nullable | Razón cambio |
| `docs_referenced` | text[] | nullable | Docs consultados |
| `scripts_executed` | text[] | nullable | Scripts corridos |
| `decisions` | jsonb | nullable | Decisiones técnicas |
| `blockers` | text | nullable | Problemas encontrados |
| `blocker_resolution` | text | nullable | Cómo se resolvió |
| `status` | text | 'completed' | **CHECK:** completed \| in_progress \| blocked \| abandoned |
| `next_steps` | text | nullable | — |
| `time_spent_minutes` | int | nullable | — |
| `execution_order` | int | nullable | Orden ejecución |
| `tags` | text[] | nullable | — |
| `notes` | text | nullable | — |
| `created_by` | text | 'kimi-session' | — |

---

## 🔐 ADMIN & SEGURIDAD

### `system_admins`
> Superadmins del sistema

| Columna | Tipo | Default | Descripción |
|---------|------|---------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `email` | text | UNIQUE | — |
| `password_hash` | text | — | bcrypt |
| `name` | text | — | — |
| `created_at` | timestamptz | now() | — |
| `last_login_at` | timestamptz | nullable | — |

---

### `admin_login_attempts`
> Intentos de login (detección intrusiones)

| Columna | Tipo | Default | Descripción |
|---------|------|---------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | nullable, FK → tenants.id | — |
| `email` | text | nullable | — |
| `ip_address` | text | nullable | — |
| `user_agent` | text | nullable | — |
| `success` | boolean | false | — |
| `attempted_at` | timestamptz | now() | — |

---

## 🎯 UTILIDADES

### `user_tenants`
> Relación muchos-a-muchos auth.users ↔ tenants

| Columna | Tipo | Default | Descripción |
|---------|------|---------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `user_id` | uuid | FK → auth.users.id | — |
| `tenant_id` | uuid | FK → tenants.id | — |
| `role` | text | 'staff' | staff \| admin |

---

### `operation_counters`
> Numeración de operaciones legibles

| Columna | Tipo | Default | Descripción |
|---------|------|---------|-------------|
| `tenant_id` | uuid | PK, FK → tenants.id | — |
| `operation_type` | text | PK | Tipo operación |
| `last_number` | int | 0 | Último número usado |

---

### `tenant_storage_usage`
> Tracking uso de storage para billing

| Columna | Tipo | Default | Descripción |
|---------|------|---------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `bytes_used` | bigint | 0 | Total bytes |
| `objects_count` | int | 0 | # archivos |
| `property_images_bytes/count` | bigint/int | 0 | Desglose |
| `product_images_bytes/count` | bigint/int | 0 | Desglose |
| `nadistudio_bytes/count` | bigint/int | 0 | Desglose |
| `month_year` | text | YYYY-MM | Periodo |
| `last_updated` | timestamptz | now() | — |
| `created_at` | timestamptz | now() | — |

---

### `business_context`
> Vector DB para contexto de IA (pgvector)

| Columna | Tipo | Default / CHECK | Descripción |
|---------|------|-----------------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `content_type` | text | — | **CHECK:** transaction \| product_info \| insight \| conversation \| error_knowledge \| analogy \| decision |
| `title` | text | nullable | — |
| `content` | text | — | Contenido textual |
| `embedding` | vector | nullable | Embedding pgvector |
| `metadata` | jsonb | {} | — |
| `last_queried_at` | timestamptz | nullable | — |
| `query_count` | int | 0 | Veces consultado |
| `created_at` | timestamptz | now() | — |
| `updated_at` | timestamptz | now() | — |

---

### `captadora_submissions`
> Submissions del formulario captadoras

| Columna | Tipo | Default | Descripción |
|---------|------|---------|-------------|
| `id` | uuid | gen_random_uuid() | PK |
| `tenant_id` | uuid | FK → tenants.id | — |
| `captadora_code` | text | — | Código captadora |
| `captadora_pin` | text | — | PIN verificación |
| `captadora_name` | text | — | Nombre |
| `is_active` | boolean | true | — |
| `submission_code` | text | nullable, UNIQUE | Código único |
| `status` | text | 'active' | — |
| `tipo`, `operacion` | text | nullable | — |
| `precio_estimado` | numeric | nullable | — |
| `moneda` | text | 'UF' | — |
| `comuna`, `direccion` | text | nullable | — |
| `notas` | text | nullable | — |
| `photos` | jsonb | [] | Array fotos |
| `photo_count` | int | 0 | — |
| `draft_id` | uuid | nullable | FK a draft |
| `kimi_analysis` | jsonb | nullable | Análisis IA |
| `created_at` | timestamptz | now() | — |
| `submitted_at` | timestamptz | nullable | — |

---

## 📊 ÍNDICES POR FUNCIONALIDAD

### Multi-tenancy (SIEMPRE filtrar por tenant_id)
- Todas las tablas tienen `tenant_id` como FK a `tenants.id`
- RLS policies filtran automáticamente por tenant

### Búsquedas frecuentes
- `tenants.slug` — UNIQUE (subdominio)
- `tenants.activation_code` — UNIQUE
- `products.tenant_id + status + is_active`
- `properties.tenant_id + status + featured`
- `product_drafts.tenant_id + status + batch_id`
- `property_drafts.tenant_id + status`
- `media_queue.tenant_id + status + expires_at`
- `entity_images.entity_type + entity_id + sort_order`

### Trazabilidad
- `entity_images.source_queue_id` → `media_queue.id`
- `product_drafts.published_product_id` → `products.id`
- `property_drafts.published_property_id` → `properties.id`

---

## 🔄 FLUJOS DE DATOS CLAVE

```
[TELEGRAM/WEB] 
      ↓
[media_queue] ← Acumulación de fotos (accumulating)
      ↓
[media_queue_photos] ← Fotos individuales ordenadas
      ↓
[Kimi Vision API] ← Procesamiento IA
      ↓
[product_drafts / property_drafts] ← Drafts con datos detectados
      ↓
[Revisión humana] ← UI de aprobación
      ↓
[products / properties] ← Publicación
      ↓
[entity_images] ← Imágenes vinculadas + trazabilidad
```

---

*Documento generado: 2026-03-27 | Para consultas de esquema, buscar aquí primero.*
