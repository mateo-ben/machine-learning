# Reto 1 · Perfiles de gestión municipal de residuos sólidos en Ecuador

Proyecto de **aprendizaje no supervisado** (K-Means, agrupamiento jerárquico Ward y PCA) para descubrir perfiles de gestión de residuos en los GAD municipales del Ecuador.

**Equipo:** Mateo Benites · Mateo Pastrano · Saul Casanova
**Curso:** Aprendizaje Automático Inicial · PUCE · Periodo 2026-2

## Pregunta

¿Qué perfiles de gestión de residuos aparecen en los registros municipales del Ecuador y cómo comunicar sus diferencias para orientar propuestas de mejora?

## Fuente de los datos

**Fuente:** INEC – AME, *Estadística de Información Ambiental Económica en GAD Municipales: Gestión de Residuos Sólidos*, sección "Gestión Integral de Residuos Sólidos" del portal oficial: https://www.ecuadorencifras.gob.ec/gad-municipales/

- **Edición:** 2024
- **Archivo original:** `BDD_GIRS_2024_VF.csv` (221 filas × 213 columnas)
- **Fecha de descarga:** 09/11/2026

El archivo de datos de este repositorio **no es la base original**. Es una tabla derivada con identificadores, variables descriptivas y 5 características calculadas a partir del original. Los nombres no llevan tildes ni eñes para evitar problemas de codificación.

## Estructura

```
reto1/
├── datos/
│   └── girs2024_caracteristicas.csv
├── Graficos_informe_Reto1.xlsx
├── README.md
└── Reto1_GIRS_notebook.ipynb
```

| Archivo | Contenido |
|---|---|
| `datos/girs2024_caracteristicas.csv` | Base preparada para el análisis (descrita abajo) |
| `Reto1_GIRS_notebook.ipynb` | Notebook de Colab: auditoría, preparación, K-Means, Ward, PCA, perfiles y conclusiones. Genera la Figura 3 del informe (`figura3_pca.png`) |
| `Graficos_informe_Reto1.xlsx` | Datos y gráficos de las Figuras 1 y 2 del informe, elaborados con los valores obtenidos en el notebook |
| `Informe_Reto1.docx` | Informe divulgativo dirigido a técnicos municipales |

## Datos: `datos/girs2024_caracteristicas.csv`

221 filas (una por cantón) y 12 columnas. Está separado por comas, en UTF-8 y con decimales con punto.

| Columna | Tipo | Descripción | Unidad / valores | Origen (códigos INEC) |
|---|---|---|---|---|
| `CantonId` | Identificador | Código del cantón (leer como texto) | — | `CantonId` |
| `NombreCanton` | Identificador | Nombre del cantón, sin tildes | — | `NombreCantón` |
| `rgnatu` | Descriptiva | Región natural | 1 Sierra, 2 Costa, 3 Amazonía, 4 Galápagos | `rgnatu` |
| `Pob` | Descriptiva | Población urbana proyectada 2022 (no se usa como denominador) | habitantes | `Pob` |
| `MSF` | Descriptiva | ¿Tiene separación en la fuente? | 0 = no, 1 = sí | `MSF` |
| `log_crr` | **Característica** | Escala: residuos recolectados en logaritmo | log10 de t/mes | log10(`CRR`) |
| `cob_barrido` | **Característica** | Cobertura del servicio de barrido | proporción 0–1 | `VB` / (`MR4221` + `MR4222`), tope 1 |
| `prop_diferenciada` | **Característica** | Parte de la recolección que es diferenciada | proporción 0–1 | `DIF` / `MR453`, tope 1 |
| `prop_recuperada` | **Característica** | Inorgánicos recuperados respecto a lo recolectado | proporción | (`RI` / 1000) / `CRR` |
| `ppc` | **Característica** | Producción per cápita urbana | kg/hab/día | `MR4321` |
| `trata_organicos` | Descriptiva | ¿Reporta tratamiento de orgánicos? | 0 = no, 1 = sí | 1 si `TROT` > 0 |
| `motivo_exclusion` | Descriptiva | Por qué falta `cob_barrido` | texto | `MR421`, `MR422` |

Solo las **5 características** entran al agrupamiento. Las demás columnas sirven para identificar y describir los perfiles.

### Valores faltantes

`cob_barrido` está vacía en 6 cantones porque no se puede calcular:

- **Faltante:** Montecristi, Rocafuerte, Patate y Quero no definieron la longitud de calles a barrer.
- **No aplica:** Gualaquiza y Paquisha no tienen servicio de barrido.

No se rellenan, así que el análisis usa **215 cantones**. En cambio, los ceros de `prop_diferenciada` y `prop_recuperada` son ceros reales: el municipio reportó que no realiza esa actividad.

## Cómo cargar los datos

```python
import pandas as pd

URL = "https://raw.githubusercontent.com/mateo-ben/machine-learning/main/reto1/datos/girs2024_caracteristicas.csv"
df = pd.read_csv(URL, dtype={"CantonId": str})
```

## Resultados

Con K-Means (k = 5, `n_init=50`, semilla 0) se identificaron cinco perfiles: *Pequeños con servicio básico completo* (76 cantones), *Recolección diferenciada* (52), *Urbanos de mayor escala sin separación* (49), *Barrido limitado* (33) y *Recuperación activa de inorgánicos* (5). El detalle está en el notebook y en el informe.

> Análisis exploratorio con fines académicos. Los perfiles **no son un ranking** ni una clasificación oficial de municipios.
