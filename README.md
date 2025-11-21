# Pipeline ELT Compras Databricks 🏪

> Pipeline **ELT end-to-end** para procesamiento de órdenes de compra (Presencial y Online) usando **Databricks** y **Delta Lake** con arquitectura escalable y validaciones de calidad.

[![Python](https://img.shields.io/badge/Python-3.9+-blue?logo=python)](https://www.python.org/)
[![Databricks](https://img.shields.io/badge/Databricks-Latest-red?logo=databricks)](https://databricks.com/)
[![Delta Lake](https://img.shields.io/badge/Delta%20Lake-Enabled-green)](https://delta.io/)
[![Status](https://img.shields.io/badge/Status-Production%20Ready-brightgreen)]()
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 📋 Tabla de Contenidos

- [Descripción del Proyecto](#descripción-del-proyecto)
- [Arquitectura ELT](#arquitectura-elt)
- [Tecnologías](#tecnologías)
- [Requisitos](#requisitos)
- [Instalación](#instalación)
- [Estructura del Proyecto](#estructura-del-proyecto)
- [Notebooks Principales](#notebooks-principales)
- [Flujo de Datos](#flujo-de-datos)
- [Validaciones de Calidad](#validaciones-de-calidad)
- [Automatización con Jobs](#automatización-con-jobs)
- [Resultados & Métricas](#resultados--métricas)
- [Troubleshooting](#troubleshooting)
- [Contacto](#contacto)

---

## 📌 Descripción del Proyecto

Este proyecto implementa un **pipeline ELT completo** que procesa datos de compras desde múltiples canales (Presencial y Online) mediante una arquitectura de 3 capas en Databricks:

✅ **Extrae** datos de CSV (compras presencial) y JSON (compras online)  
✅ **Carga** en capa Bronze con trazabilidad completa  
✅ **Transforma** con limpieza, validación y enriquecimiento de datos  
✅ **Valida** calidad en cada capa con reglas automáticas  
✅ **Genera** tablas analíticas en capa Gold con joins y agregaciones  
✅ **Automatiza** mediante Jobs con notificaciones por email  

### 🎯 Caso de Uso

Procesamiento de **órdenes de compra** de una plataforma de e-commerce (estilo Linio) que captura:
- **Compras Presencial**: Tiendas físicas con datos en CSV
- **Compras Online**: Plataforma digital con datos en JSON
- **Detalles de Compra**: Productos, categorías, precios (múltiples CSV)

Resultado: Tabla de hechos analítica (`gold_fact_compras`) con información consolidada para BI/reportes.

---

## 🏗️ Arquitectura ELT

<img width="2400" height="1600" alt="image" src="https://github.com/user-attachments/assets/b9fb05da-d3f4-4036-a7d8-cb54176c4130" />

```

┌──────────────────────────────────────────────────────────────────┐
│                        FUENTES DE DATOS                          │
│  ┌─────────────────────┐  ┌──────────────────────────────────┐  │
│  │  Presencial.csv     │  │  Online.json + Detalles/*.csv    │  │
│  │  (Tiendas físicas)  │  │  (Plataforma Online)             │  │
│  └─────────────────────┘  └──────────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌──────────────────────────────────────────────────────────────────┐
│                     🟫 CAPA BRONZE (Raw)                         │
│  01_bronze_layer → bronze_compras, bronze_detalles              │
│  - Ingesta sin transformación (solo strings)                     │
│  - Renombre de columnas (snake_case)                             │
│  - Trazabilidad (tipo_compra, fecha_carga)                       │
│  01_bronze_calidad → log_calidad_datos                           │
│  - Validaciones: no nulos, duplicados, formato                   │
│  - Parámetros de control → estado, detalle (JSON)               │
└──────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌──────────────────────────────────────────────────────────────────┐
│                     🟩 CAPA SILVER (Clean)                       │
│  02_silver_layer → silver_compras, silver_detalles              │
│  - Conversión de tipos de datos                                  │
│  - Limpieza de espacios y formato de texto                       │
│  - Transformaciones complejas:                                   │
│    • Estado (1→"Creado", 2→"En Curso", etc.)                   │
│    • Extracción cliente_id + num_documento                      │
│    • Tipo de documento (DNI, RUC10, RUC20)                      │
│    • Nombre cliente (concat nombres + apellidos)                │
│    • Días abierto + grupo de días                               │
│  02_silver_calidad → log_calidad_datos                           │
│  - Validaciones: relaciones entre fechas, rangos                │
└──────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌──────────────────────────────────────────────────────────────────┐
│                     🟨 CAPA GOLD (Analytics)                     │
│  03_gold_layer → gold_fact_compras                               │
│  - Join silver_compras + silver_detalles (match por factura)    │
│  - Tabla de hechos con todas las dimensiones                     │
│  - Listo para BI (Power BI, Tableau, Looker)                    │
└──────────────────────────────────────────────────────────────────┘
                                  │
                                  ▼
┌──────────────────────────────────────────────────────────────────┐
│                   📊 VISUALIZACIÓN & BI                          │
│  Power BI | Tableau | Databricks SQL Analytics                  │
└──────────────────────────────────────────────────────────────────┘
```

---

## 🛠️ Tecnologías

| Componente | Herramienta | Versión | Detalle |
|-----------|-----------|---------|---------|
| **Plataforma** | Databricks | Latest | Enterprise/Community |
| **Motor de Datos** | Apache Spark | 3.4.x | Distributed Computing |
| **Storage** | Delta Lake | Latest | ACID Transactions |
| **Cloud** | Microsoft Azure | - | Infrastructure |
| **Lenguaje** | Python | 3.9+ | PySpark |
| **Orquestación** | Databricks Workflows | - | Jobs & Tasks |
| **Notificaciones** | Email Alerts | - | Success/Error/Quality |
| **Testing** | pytest | Latest | Unit & Integration |
| **BI** | Power BI / Tableau | Latest | Visualización |

---

## 📦 Requisitos Previos

- ✅ Cuenta activa en **Databricks** (Community o Enterprise)
- ✅ Acceso a **Azure Storage** o almacenamiento local
- ✅ Datasets descargados (`dataset.zip` con archivos CSV/JSON)
- ✅ Notebooks de email descargados (`Correos.dbc`)
- ✅ Python 3.9+
- ✅ Git
- ✅ pip (gestor de paquetes)

---

## 🚀 Instalación & Configuración

### 1️⃣ Preparación en Databricks

```bash
# A) Crear esquema en Databricks
CREATE SCHEMA linio;

# B) Crear volumes para datos
CREATE VOLUME linio.compras;
CREATE VOLUME linio.detalles;

# C) Cargar archivos del dataset.zip
# - Presencial.csv → /Volumes/linio/compras/
# - Online.json → /Volumes/linio/compras/
# - *.csv (detalles) → /Volumes/linio/detalles/

# D) Importar notebooks de email
# Descargar Correos.dbc → Import en Databricks
```

### 2️⃣ Clonar el Repositorio

```bash
git clone https://github.com/tu-usuario/elt-compras-databricks.git
cd elt-compras-databricks
```

### 3️⃣ Instalar Dependencias (Local)

```bash
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### 4️⃣ Configurar Credenciales

Crea `.env` en la raíz:

```env
# Databricks
DATABRICKS_HOST=https://your-workspace.cloud.databricks.com
DATABRICKS_TOKEN=dapiXXXXXXXXXXXXXXXX

# Proyecto ELT
SCHEMA_NAME=linio
VOLUME_COMPRAS=linio.compras
VOLUME_DETALLES=linio.detalles

# Email notifications
EMAIL_FROM=your-email@example.com
EMAIL_SMTP=smtp.gmail.com
EMAIL_PORT=587
EMAIL_USER=your-email@example.com
EMAIL_PASSWORD=your-app-password
EMAIL_TO=destinatario@example.com
```

### 5️⃣ En Databricks - Importar Notebooks

Opción A: Manualmente
```
Workspace → Import → Seleccionar archivos de /notebooks
```

Opción B: CLI
```bash
databricks workspace import_dir ./notebooks /Users/tu-usuario/elt-compras
```

---

## 📁 Estructura del Proyecto

```
pipeline-elt-compras-databricks/
├── 📁 notebooks/
│   ├── 📁 bronze_layer/
│   │   ├── 01_bronze_layer.ipynb          # Ingestión de datos raw
│   │   └── 01_bronze_calidad.ipynb        # Validación calidad Bronze
│   ├── 📁 silver_layer/
│   │   ├── 02_silver_layer.ipynb          # Transformaciones y limpieza
│   │   └── 02_silver_calidad.ipynb        # Validación calidad Silver
│   ├── 📁 gold_layer/
│   │   └── 03_gold_layer.ipynb            # Consolidación analytics
│   └── 📁 notifications/
│       ├── enviar_correo_exitoso.ipynb    # Notificación éxito
│       ├── enviar_correo_error.ipynb      # Notificación error
│       └── enviar_correo_calidad.ipynb    # Notificación calidad
├── 📁 data/
│   ├── 📁 raw/
│       ├── 📁 compras/
│       │   ├── Presencial.csv             # Compras presenciales
│       │   └── Online.json                # Compras online
│       └── 📁 detalles/
│           ├── detalle_001.csv
│           ├── detalle_002.csv
│           └── ...
├── 📁 docs/
│   └── architecture_diagram.png
│ 
├── 📁 assets/
│   └── 📁 screenshots/
│       ├── databricks_job.png
│       └── delta_tables.png
│  
├── .gitignore
├── requirements.txt
├── README.md
└── LICENSE
```

---

## 📖 Notebooks Principales

### 🟫 Bronze Layer

#### `01_bronze_layer.ipynb`
**Objetivo**: Ingesta raw de datos sin transformación

```python
# Flujo:
1. Leer Presencial.csv → df_compras_presencial (strings)
2. Renombrar columnas a snake_case
3. Agregar tipo_compra="Presencial", fecha_carga=now()
4. Leer Online.json → df_compras_online (strings con StructType)
5. Renombrar columnas a snake_case
6. Agregar tipo_compra="Online", fecha_carga=now()
7. UNION df_compras_presencial + df_compras_online → df_compras
8. Leer masivamente *.csv de detalles → df_detalles
9. Renombrar columnas, agregar nombre_archivo, fecha_carga
10. Crear/poblar tablas delta:
    - bronze_compras (con try...except)
    - bronze_detalles (con try...except)
```

**Entradas**: CSV presencial, JSON online, CSV detalles (volumes)  
**Salidas**: `bronze_compras`, `bronze_detalles` tables  

---

#### `01_bronze_calidad.ipynb`
**Objetivo**: Validar calidad de datos en capa bronze

```python
# Reglas para bronze_compras:
- [factura] ≠ null
- [factura] ≥ 7 caracteres
- [factura] sin duplicados
- [fecha_orden] ≠ null

# Reglas para bronze_detalles:
- [factura] ≠ null
- [producto] ≠ null

# Salida: log_calidad_datos (append mode)
# Parámetros exportados al siguiente notebook:
- estado: "Validacion Exitosa" | "Falla Critica"
- detalle: JSON con registros fallidos (si aplica)
```

---

### 🟩 Silver Layer

#### `02_silver_layer.ipynb`
**Objetivo**: Transformación, limpieza y enriquecimiento

**Parámetro**: `fecha_carga` (default: '2025-06-16')

```python
# SILVER_COMPRAS - Transformaciones:
- venta_id, estado → integer
- fecha_orden, fecha_entrega, fecha_envio → date
- factura, tipo_cliente → UPPER + TRIM
- nombres, apellidos → INITCAP + TRIM
- vendedor, departamento, metodo_pago → TRIM
- estado: 1→"Creado", 2→"En Curso", 3→"Programado", 4→"Cancelado", 5→"Entregado"
- cliente_id: split por "-" → cliente_id (int), num_documento (string)
- num_documento: pad left con 0 si < 8 chars
- tipo_documento: "DNI" (8 chars) | "RUC10" (11, inicia con 10) | "RUC20" (11, inicia con 20)
- nombre_cliente: concat(nombres, apellidos)
- dias_abierto: DATEDIFF(fecha_carga, fecha_orden) si estado IN ("Creado", "En Curso", "Programado")
- grupo_dias_abierto: "[0-3 días]" | "[4-7 días]" | "[más de 8 días]"

# SILVER_DETALLES - Transformaciones:
- detalle_id, unidades, oferta_id → integer
- precio_unitario → double
- factura → UPPER + TRIM
- categoria, subcategoria, producto → TRIM
- subtotal: unidades * precio_unitario
- tienda: extract antes del "." en nombre_archivo
```

---

#### `02_silver_calidad.ipynb`
**Objetivo**: Validar calidad después de transformaciones

```python
# Reglas para silver_compras:
- Si [estado]="Entregado" → [fecha_envio] ≠ null
- Si [estado]="Entregado" → [fecha_envio] ≥ [fecha_orden]

# Reglas para silver_detalles:
- [subtotal] > 0

# Salida: log_calidad_datos (append mode)
# Parámetros: estado, detalle (si aplica)
```

---

### 🟨 Gold Layer

#### `03_gold_layer.ipynb`
**Objetivo**: Crear tabla de hechos analítica

```python
# Flujo:
1. Leer silver_compras, silver_detalles
2. INNER JOIN por [factura]
3. Seleccionar columnas finales (20+ columnas)
4. Agregar fecha_carga (trazabilidad)
5. Crear/poblar gold_fact_compras (con try...except)

# Tabla resultante:
gold_fact_compras (1 fila por detalle de compra, con info de orden)
Ideal para: dashboards, reportes, análisis OLAP
```

---

## 🔄 Flujo de Datos Completo

```
EJECUCIÓN MANUAL (para testing):
1. Run 00_setup.ipynb (crear schema + volumes)
2. Run 01_bronze_layer.ipynb → bronze_compras, bronze_detalles
3. Run 01_bronze_calidad.ipynb → validaciones + parámetros
4. IF estado="Validacion Exitosa":
     Run 02_silver_layer.ipynb → silver_compras, silver_detalles
   ELSE:
     Stop (enviar email de error)
5. Run 02_silver_calidad.ipynb → validaciones
6. IF estado="Validacion Exitosa":
     Run 03_gold_layer.ipynb → gold_fact_compras
   ELSE:
     Stop (enviar email de alert)
7. Run send_success.ipynb → email notificación

EJECUCIÓN AUTOMATIZADA (con Job):
Databricks Workflow → Todas las tareas arriba + condicionales + email
(Ver detalles en siguiente sección)
```

---

## ✅ Validaciones de Calidad

### Bronze Layer Checks
```
✓ No nulls en columnas clave (factura, fecha_orden)
✓ Formato mínimo (ej: factura ≥ 7 chars)
✓ Detección de duplicados
```

### Silver Layer Checks
```
✓ Integridad referencial (fechas coherentes)
✓ Rangos válidos (ej: subtotal > 0)
```

### Gold Layer
```
✓ Completitud del join (100% de registros coinciden)
✓ Sin nulos en columnas analíticas clave
```

---

## 🔧 Automatización con Databricks Jobs

**Nombre del Job**: `Pipeline ELT Compras`

### Estructura del Workflow

```
START
  │
  ├─→ [Task] 00_Setup
  │   └─→ Status: Success ✓
  │
  ├─→ [Task] 01_Bronze_Layer
  │   └─→ Status: Success ✓
  │
  ├─→ [Task] 01_Bronze_Calidad
  │   ├─→ IF estado="Validacion Exitosa": Continue
  │   └─→ ELSE: Send Email Error → END
  │
  ├─→ [Task] 02_Silver_Layer
  │   └─→ Status: Success ✓
  │
  ├─→ [Task] 02_Silver_Calidad
  │   ├─→ IF estado="Validacion Exitosa": Continue
  │   └─→ ELSE: Send Email Quality Alert → END
  │
  ├─→ [Task] 03_Gold_Layer
  │   └─→ Status: Success ✓
  │
  └─→ [Task] Send_Success_Email
      └─→ END (Success)
```

### Configuración del Job

```bash
# Crear job con CLI:
databricks jobs create --json '{
  "name": "Pipeline ELT Compras",
  "tasks": [
    {"task_key": "setup", "notebook_task": {"notebook_path": "/Repos/..."}},
    {"task_key": "bronze_layer", "depends_on": [{"task_key": "setup"}], ...},
    ...
  ],
  "schedule": {"quartz_cron_expression": "0 2 * * ? *"},  # 2 AM daily
  "timeout_seconds": 3600
}'

# Ejecutar manualmente:
databricks jobs run-now --job-id 12345

# Ver ejecuciones:
databricks runs list --job-id 12345
```

### Email Notifications

Todos los tasks envían emails con parámetro `send_to`:

```python
# En cada task de email:
dbutils.widgets.text("send_to", "admin@company.com")
send_to = dbutils.widgets.get("send_to")

# Ejemplo: send_success.ipynb
smtplib.send(
  subject="✅ Pipeline ELT Compras - Ejecución Exitosa",
  body=f"Tablas actualizadas: bronze, silver, gold. Registros procesados: X"
)
```

---

## 📊 Resultados & Métricas

### Performance (Estimado con 50M registros)

| Métrica | Valor | Observación |
|---------|-------|-----------|
| **Ingesta Bronze** | ~2 min | CSV presencial + JSON online |
| **Transformación Silver** | ~4 min | Limpieza y tipo conversión |
| **Validación Calidad** | ~1.5 min | Reglas complejas |
| **Join & Gold** | ~3 min | 1 fact table de 100M+ filas |
| **Tiempo Total** | ~10.5 min | End-to-end pipeline |
| **Email Notification** | ~30 seg | Async |

### Calidad de Datos (Ejemplo)

```
📊 REPORTE DE CALIDAD - Ejecución 2025-06-16
═════════════════════════════════════════════

BRONZE LAYER:
✅ bronze_compras: 2,500,000 registros
   - Duplicados detectados: 250 (0.01%) → Removidos
   - Nulos [factura]: 0 ✓
   - Nulos [fecha_orden]: 0 ✓
   
✅ bronze_detalles: 5,000,000 registros
   - Nulos [factura]: 0 ✓
   - Nulos [producto]: 0 ✓

SILVER LAYER:
✅ silver_compras: 2,500,000 registros
   - Inconsistencia fecha_envio: 15 registros (estado=Entregado pero fecha_envio=null)
   - Regla fallida: [fecha_envio] ≥ [fecha_orden]: 3 registros
   - Estado: ⚠️ VALIDACION CON ADVERTENCIAS

✅ silver_detalles: 5,000,000 registros
   - Subtotal < 0: 0 registros ✓

GOLD LAYER:
✅ gold_fact_compras: 5,000,000 registros
   - Join completitud: 100% ✓
   - Ready for BI ✅
```

## 📚 Recursos Útiles

- 🔗 [Databricks Documentation](https://docs.databricks.com/)
- 🔗 [Delta Lake Guide](https://delta.io/learn/delta-lake-tutorial/)
- 🔗 [PySpark SQL API](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/)
- 🔗 [Databricks Workflows](https://docs.databricks.com/workflows/)
- 🔗 [Data Quality Best Practices](https://docs.databricks.com/delta/tutorial/)

---

## 🤝 Contribuciones

Las contribuciones son bienvenidas:

1. Fork el proyecto
2. Crear rama feature (`git checkout -b feature/MejorValidacion`)
3. Commit (`git commit -m 'Add nueva regla de calidad'`)
4. Push (`git push origin feature/MejorValidacion`)
5. Pull Request

---

## 📄 Licencia

MIT License - Ver [LICENSE](LICENSE)

---

## 👤 Contacto & Links

- 💼 **LinkedIn**: [Tu Perfil](https://www.linkedin.com/in/geiby-maribel-sosa-chamba-300831b7/)
- 🌐 **Portfolio**: [Tu Web](https://portfolio-data-engineer.lovable.app/)
- 📧 **Email**: geibymari@gmail.com
- 📹 **Video del Proyecto**: [LinkedIn Post](https://www.linkedin.com/posts/geiby-maribel-sosa-chamba-300831b7_dataengineering-databricks-elt-activity-7378228733519876096-FNgJ?utm_source=share&utm_medium=member_desktop&rcm=ACoAABjci3ABc4Wz2UXPphN0W0ty6TIVmuuEbsI)

---

## 🔔 Changelog

**v1.0.0** (2025-11-17)
- ✅ Pipeline ELT completo (Bronze → Silver → Gold)
- ✅ Validaciones de calidad en 3 capas
- ✅ Automatización con Databricks Jobs
- ✅ Notificaciones por email
- ✅ Documentación completa

---

**Última actualización**: Noviembre 2025  
**Versión**: 1.0.0  
**Estado**: ✅ Production Ready

⭐ **Si te fue útil, dale una estrella en GitHub!** ⭐
