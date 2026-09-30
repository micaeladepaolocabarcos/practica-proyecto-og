# Limpieza y Análisis Exploratorio: Producción de Pozos No Convencionales (Vaca Muerta, Argentina)

Práctica del curso Data Science I (Coderhouse). 

El objetivo es construir un flujo de limpieza reproducible con Pandas sobre datos reales y responder preguntas de negocio mediante agrupaciones.

## Origen de los datos

| Campo | Detalle |
|---|---|
| Dataset | Producción de Pozos de Gas y Petróleo No Convencional |
| Fuente | Secretaría de Energía de la Nación, vía [datos.gob.ar](https://datos.gob.ar/dataset/energia-produccion-petroleo-gas-por-pozo-capitulo-iv) |
| Archivo | `produccion-de-pozos-de-gas-y-petroleo-no-convencional.csv` |
| Tamaño | ~145 MB, 421.022 filas × 40 columnas |
| Período | enero 2006 a julio 2026 |
| Unidad de observación | producción de un pozo en un mes |
| Unidades | `prod_pet` y `prod_agua` en m³; `prod_gas` en miles de m³ |

El CSV no está incluido en el repositorio porque supera los 50 MB. Para reproducir el análisis, descargarlo desde el link de la fuente y actualizar la variable `RUTA_CSV` en el notebook.

## Contenido del repositorio

- `U6_Limpieza_y_Análisis_Exploratorio_de_un_Dataset_Real.ipynb`: notebook con diagnóstico, limpieza y agregaciones.
- `README.md`: este archivo.

## Diagnóstico inicial

- Fechas guardadas como texto y valores lógicos como `'t'`/`'f'`.
- `vida_util` (97,9%) y `observaciones` (94,3%) casi vacías; otras 6 columnas con menos del 0,25% de nulos.
- Sin filas duplicadas (ni exactas ni por la clave pozo-año-mes).
- Valores inválidos: 3 producciones negativas, 5.738 profundidades en 0 y 152 profundidades mayores a 8.000 m (la mayoría con 378.939 m).

## Decisiones de limpieza

| Problema | Decisión | Justificación |
|---|---|---|
| `vida_util`, `observaciones` casi vacías | Eliminar columnas | Con más del 94% de nulos no hay base para imputar |
| `habilitado` con un único valor | Eliminar columna | No aporta información |
| Fechas como texto | Convertir a `datetime` | Permite operar con fechas (por ejemplo, filtrar por período o extraer el mes) |
| `rectificado` como `'t'`/`'f'` | Convertir a `bool` | Tipo correcto para un valor lógico |
| Texto con pocos valores repetidos | Convertir a `category` | Ocupa menos memoria y agiliza los cálculos |
| Nulos en `tipopozo`, `tipoextraccion`, `tipoestado` | Imputar con la **moda del propio pozo** | La moda general asignaría el mismo tipo a cualquier pozo; la del pozo respeta su historial. Recupera 610 de 614 filas |
| Nulos no recuperables (incluye `clasificacion`, `subclasificacion`, `sub_tipo_recurso`) | Eliminar filas | Pertenecen a pozos sin ningún mes informado; son el 0,33% del total |
| Producciones negativas | Eliminar filas (3) | Físicamente imposibles |
| Profundidad en 0 o mayor a 8.000 m | Imputar con la **mediana de la formación** | La profundidad depende de la formación objetivo; la mediana es resistente a extremos |
| Duplicados | `drop_duplicates()` preventivo | No hay, pero el paso mantiene el flujo válido ante actualizaciones de la fuente |
| Índice con huecos tras eliminar filas | `reset_index(drop=True)` | Deja un índice continuo sin agregar columnas |

**Resultado:** 419.641 filas × 37 columnas, sin nulos (se eliminó solo el 0,33% de las filas).

## Agregaciones de negocio

1. **Evolución anual de la producción:** el petróleo no convencional pasó de ~15 mil m³ (2006) a ~29,4 millones de m³ (2025); los pozos activos se multiplicaron por más de 20.
2. **Rendimiento por formación:** Vaca Muerta es la más productiva por pozo, tanto en petróleo (~914 m³/mes) como en gas.
3. **Concentración por empresa:** YPF aporta el 58% del petróleo no convencional histórico; las 5 primeras empresas, más del 84%.

## Herramientas

Python, pandas, numpy, Google Colab.
