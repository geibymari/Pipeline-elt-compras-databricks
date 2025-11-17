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
