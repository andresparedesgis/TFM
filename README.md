# 🗺️ SIG de Movilidad Urbana en la Comunidad de Madrid

**Trabajo Final de Máster — Master GIS Online**  
**Autor:** Andrés Paredes Fernández · [andres.paredes@esri.es](mailto:andres.paredes@esri.es)  
**Colaboración:** Esri España

---

## 📋 Descripción del proyecto

Sistema de Información Geográfica que integra la **red viaria**, los **servicios de movilidad** y las **Zonas de Bajas Emisiones (ZBE)** de la Comunidad de Madrid, ofreciendo herramientas de análisis y aplicaciones interactivas para técnicos GIS, gestores y ciudadanos.

El sistema permite, entre otras cosas, calcular rutas teniendo en cuenta las restricciones de acceso según la **etiqueta ambiental DGT** del vehículo (0, ECO, C, B o SIN etiqueta), localizar aparcamientos, puntos de recarga eléctrica y gasolineras, y reportar incidencias en tiempo real desde dispositivos móviles.

---

## 📁 Contenido del repositorio

```
📦 TFM-SIG-Movilidad-Madrid
├── 📄 Memoria/
│   └── Trabajo_TFM_APF.pdf     # Memoria académica completa del TFM
├── 🌐 Web/
│   ├── index.html                          # Página principal del proyecto
│   └── tecnico.html                        # Página de documentación técnica
├── 🔧 Releases/
│   ├── Proyecto.Zip               # Proyecto completo para ArcGIS Pro
└── 📄 README.md
```

---

## 🏗️ Arquitectura del sistema

### Geodatabase — `GDB_Movilidad`

La geodatabase inicial está estructurada en **5 feature datasets** temáticos:

| Feature Dataset | Contenido principal |
|---|---|
| `Red_Viaria` | Carreteras (Superficie, Pasos Elevados, Pasos Subterráneos) + Network Dataset |
| `ZBE` | Límites ZBE, cámaras, puntos de control, viarios restringidos |
| `Estacionamientos` | Zonas SER (azul, naranja, verde, rojo), aparcamientos públicos |
| `Seguridad_Vial` | Radares de velocidad fija |
| `Servicios` | Gasolineras, recarga eléctrica, sitios públicos, elementos de interés |

---

## ⚙️ Automatización

### ModelBuilder

**Modelo 1 — Infraestructura y distribución de capas**
- Parte 1: Crea la GDB y los 6 feature datasets mediante `Create File Geodatabase` + `Create Feature Dataset`.
- Parte 2: Itera sobre las capas raw con `Iterate Feature Classes`, las renombra, reproyecta y distribuye automáticamente al dataset correspondiente mediante lógica condicional.

**Modelo 2 — Análisis de cobertura de estacionamientos**
- Flujo: `Merge → Buffer → Dissolve → Clip → Minimum Bounding Geometry → Erase`
- Salida adicional: ráster de `Kernel Density` con la concentración de servicios de aparcamiento.

### Notebooks Python (ArcPy)

**Notebook 1 — Red viaria**
- Calcula velocidades por tipo de vía (`fclass`), tiempos de conducción y a pie, y jerarquía de red.
- Divide la capa en tres sublayers: `Superficie`, `Pasos_Elevados`, `Pasos_Subterraneos`.
- Ejecuta `Integrate` y `CreateNetworkDataset` para generar `ND_Madrid_Notebook`.

**Notebook 2 — Análisis de rutas con evaluación ZBE**
- Calcula la ruta óptima origen–destino (Ruta 1, sin restricciones).
- Evalúa si la ruta cruza zonas prohibidas para la etiqueta del vehículo mediante `Intersect`.
- Si la ruta no es válida, calcula una ruta alternativa (Ruta 2) usando `Polygon Barriers`.
- Campo de resultado: `Color_ZBE` → `VERDE` (acceso permitido) / `ROJO` (acceso restringido).
- 
🔑 Lógica de restricciones ZBE

| Etiqueta | ZBE General | ZBEDEP Distrito Centro | ZBEDEP Plaza Elíptica |
|---|---|---|---|
| **0 emisiones** | ✅ Acceso libre | ✅ Acceso libre | ✅ Acceso libre |
| **ECO** | ✅ Acceso libre | ✅ Acceso libre | ✅ Acceso libre |
| **C** | ✅ Acceso libre | ❌ Restringido | ❌ Restringido |
| **B** | ✅ Acceso libre | ❌ Restringido | ❌ Restringido |
| **SIN etiqueta** | ❌ Restringido | ❌ Restringido | ❌ Restringido |

---

### Experience Builder

Aplicación web de página única con cuatro componentes principales:

1. **Widget de mapa** — Selección de origen y destino mediante clic sobre el mapa.
2. **Listado elementos de Interes** — Listado de los elementos de interes repartidos por Madrid.
3. **Visualización en tiempo real de los elementos del mapa** — Visualizacion de los elementos que se puedan añadir o actualizar con el uso de ArcGIS QuickCaptures y Field Maps en el mapa en tiempo real.
4. **Panel de resultados** — Muestra la ruta sobre el mapa (verde/roja) e información textual.

### Aplicaciones móviles

| Aplicación | Perfil | Función |
|---|---|---|
| **Field Maps** | Técnicos GIS, emergencias | Edición detallada de gasolineras, recargas, radares, incidencias |
| **ArcGIS QuickCapture** | Ciudadanía general | Reporte rápido de radares, advertencias y accidentes |

---

## 🛠️ Tecnologías utilizadas

- **ArcGIS Pro** — Geodatabase, ModelBuilder, Network Analyst, ArcPy
- **ArcGIS Online** — Feature Layers, Tile Layers, Portal de la organización
- **ArcGIS Notebooks** — Python (arcpy, arcpy.na, ArcGIS API for Python)
- **Experience Builder** — Aplicación web de consulta y análisis
- **Field Maps + Field Maps Designer** — Captura móvil con formularios enriquecidos
- **ArcGIS QuickCapture** — Reporte ciudadano ultrarrápido
- **Datos abiertos** — [datos.madrid.es](https://datos.madrid.es) · Geoportal Comunidad de Madrid · OpenStreetMap

---

## 📊 Fuentes de datos

| Capa | Fuente |
|---|---|
| Red viaria | OpenStreetMap (procesado con ArcGIS Pro) |
| ZBE y puntos de control | Portal de datos abiertos del Ayuntamiento de Madrid |
| Estacionamientos (SER) | Portal de datos abiertos del Ayuntamiento de Madrid |
| Radares de velocidad | Portal de datos abiertos del Ayuntamiento de Madrid |
| Gasolineras y recargas | Portal de datos abiertos del Ayuntamiento de Madrid |
| Elementos turísticos | Portal de datos abiertos del Ayuntamiento de Madrid |

Todos los datos son de acceso abierto y libre reproducción, garantizando la replicabilidad del proyecto.

---

## 👥 Perfiles de usuario

| Perfil de usuario                                    | Aplicaciones y herramientas                | Casos de uso principales                                                                                                                                              | Permisos                                                                |
| ---------------------------------------------------- | ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **Técnicos GIS, Gestores y Servicios de Emergencia** | ArcGIS Pro, Field Maps, Experience Builder | Análisis espaciales avanzados, validación y actualización de datos en campo, gestión de incidencias, supervisión de infraestructuras y coordinación de emergencias    | Acceso completo a todos los servicios, incluyendo edición y publicación |
| **Ciudadanía general y turistas**                    | Experience Builder, QuickCapture           | Consulta de rutas, visualización de ZBE, localización de gasolineras, puntos de recarga y aparcamientos, descubrimiento de puntos de interés y reporte de incidencias | Acceso a servicios públicos abiertos y envío de incidencias             |

---

## 📄 Documentación

- 📘 **Memoria completa** → `Memoria/Trabajo_TFM_APF.pdf`
- 🌐 **Web del proyecto** → `Web/index.html` (apertura directa en navegador)
- 🔧 **Documentación técnica** → `Web/tecnico.html`

---

## 📬 Contacto

**Andrés Paredes Fernández**  
Master GIS Online  
✉️ [andres.paredes@esri.es](mailto:andres.paredes@esri.es)
