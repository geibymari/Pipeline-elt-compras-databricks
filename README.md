# pipeline-elt-compras-databricks
Pipeline ELT end-to-end para análisis de compras multicanal usando Databricks, Delta Lake y arquitectura Medallion (Bronze-Silver-Gold).
<img width="103" height="20" alt="image" src="https://github.com/user-attachments/assets/81079b1e-2540-4d5b-b1e3-dd1a54b13154" />
<img width="129" height="20" alt="image" src="https://github.com/user-attachments/assets/c2780a6a-6195-4155-9f8c-16c8be671504" />
<img width="122" height="20" alt="image" src="https://github.com/user-attachments/assets/8f522984-a510-456a-9369-83e9b7728d4b" />
<img width="152" height="20" alt="image" src="https://github.com/user-attachments/assets/73aed439-43c5-4918-945b-db53e8ca9c64" />
<img width="82" height="20" alt="image" src="https://github.com/user-attachments/assets/1714e6cd-6bc9-4cf3-a250-11782f719c7b" />

### 📋 Tabla de Contenidos
* Descripción del Proyecto
* Arquitectura-ELT
* Tecnologías
* Requisitos
* Instalación
* Estructura del Proyecto
* Notebooks Principales
* Flujo de Datos
* Validaciones de Calidad
* Automatización con Jobs
* Resultados & Métricas
* Troubleshooting
* Contacto

### 📌 Descripción del Proyecto
Este proyecto implementa un pipeline ELT completo que procesa datos de compras desde múltiples canales (Presencial y Online) mediante una arquitectura de 3 capas en Databricks:

✅ Extrae datos de CSV (compras presencial) y JSON (compras online)

✅ Carga en capa Bronze con trazabilidad completa

✅ Transforma con limpieza, validación y enriquecimiento de datos

✅ Valida calidad en cada capa con reglas automáticas

✅ Genera tablas analíticas en capa Gold con joins y agregaciones

✅ Automatiza mediante Jobs con notificaciones por email

#### 🎯 Caso de Uso

Procesamiento de órdenes de compra de una plataforma de e-commerce (estilo Linio) que captura:

* Compras Presencial: Tiendas físicas con datos en CSV

* Compras Online: Plataforma digital con datos en JSON

* Detalles de Compra: Productos, categorías, precios (múltiples CSV)

Resultado: Tabla de hechos analítica (gold_fact_compras) con información consolidada para BI/reportes.

### 🏗️ Arquitectura ELT

┌──────────────────────────────────────────────────────────────────┐
│                       
                        FUENTES DE DATOS                          │
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
