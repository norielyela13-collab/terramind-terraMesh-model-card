# Ficha del conjunto de datos (*Dataset Card*): TerraMesh

Documento elaborado siguiendo los principios de documentación de conjuntos de datos y la plantilla `datasetcard_template.md` del repositorio `huggingface_hub` de Hugging Face, junto con los principios generales de transparencia de Mitchell et al. (2019). Esta ficha documenta **únicamente el conjunto de datos**; no debe confundirse con la ficha del modelo TerraMind ([`MODEL_CARD.md`](./MODEL_CARD.md)).

**Convención de etiquetas usada en este documento:**

- **[Documentado]** → información publicada por los responsables del conjunto de datos (ficha oficial, repositorio).
- **[Publicado]** → información o resultados presentados en artículos científicos.
- **[Interpretación]** → inferencia razonable, señalada como tal.
- **[Pendiente]** → información no verificable en las fuentes consultadas.

---

## 8.1. Identificación del conjunto de datos

| Campo | Información |
| --- | --- |
| Nombre | **TerraMesh** |
| Organización responsable | `ibm-esa-geospatial` (IBM y ESA Φ-lab), en el marco del proyecto FAST-EO |
| Repositorio oficial | <https://huggingface.co/datasets/ibm-esa-geospatial/TerraMesh> |
| Artículo científico | Blumenstiel, B., Fraccaro, P., Marsocci, V., Jakubik, J., Maurogiovanni, S., Czerkawski, M., … Longépé, N. (2025). *TerraMesh: A Planetary Mosaic of Multimodal Earth Observation Data*. CVPR Workshops 2025. Preprint: <https://arxiv.org/abs/2504.11172> (DOI arXiv: 10.48550/arXiv.2504.11172) |
| Licencia declarada | **CC BY-SA 4.0** (`license: cc-by-sa-4.0` en los metadatos de la ficha) |
| Fecha de lanzamiento | 1 de agosto de 2025, según el registro de noticias de la ficha oficial |
| Fecha de elaboración de esta ficha | 14 de octubre de 2026 |
| Referencias principales | Ficha oficial en Hugging Face; artículo de TerraMesh; artículo de TerraMind (§3 y apéndice §8) |

---

## 8.2. Descripción general

**TerraMesh es un conjunto de datos multimodal a escala planetaria, en formato de datos listos para análisis (*Analysis-Ready Data*, ARD), destinado al preentrenamiento de modelos fundacionales de observación de la Tierra.** Según su ficha oficial, fusiona datos de **Sentinel-1 (radar SAR), Sentinel-2 (óptico), Copernicus DEM (elevación), NDVI (vegetación) y cartografía de cobertura del suelo (LULC)** en más de **9 millones de parches corregistrados** listos para aprendizaje de representaciones a gran escala ([ficha oficial](https://huggingface.co/datasets/ibm-esa-geospatial/TerraMesh)).

Su relación con TerraMind es directa y de dependencia: **TerraMesh fue creado como corpus de preentrenamiento de TerraMind**. El artículo de TerraMesh lo presenta como la contribución principal del trabajo («el mayor conjunto de datos multimodal corregistrado y disponible públicamente hasta la fecha»), y el artículo de TerraMind declara que todas las versiones de TerraMind se preentrenaron sobre TerraMesh o un subconjunto de ella. A la inversa, la ficha de TerraMesh usa a TerraMind como demostración de utilidad: el modelo `TerraMind-1.0-base` se preentrenó con él.

---

## 8.3. Objetivo y contexto de creación

**[Publicado]** El problema que motiva TerraMesh, según sus autores, es que los conjuntos públicos disponibles para preentrenar modelos fundacionales eran **limitados en escala, cobertura geográfica o variedad de sensores**, y que la mayoría enfocaban un solo par de modalidades (por ejemplo, S-1 y S-2) o no exigían alineación espaciotemporal entre ellas, lo que impide aprender representaciones cruzadas fiables entre modalidades (artículo de TerraMesh, §1 y §2).

Los objetivos declarados son:

1. Poner a disposición de la comunidad el **mayor conjunto de datos multimodal corregistrado** disponible públicamente para preentrenamiento en observación de la Tierra.
2. Aportar un **proceso de filtrado y submuestreo** que garantice diversidad, calidad y facilidad de uso en formato ARD.
3. **Demostrar empíricamente** que los modelos preentrenados en TerraMesh mejoran su rendimiento en tareas aguas abajo.

En el contexto de los modelos fundacionales geoespaciales, TerraMesh cumple la función de **corpus común de preentrenamiento**: permite que distintos equipos entrenen modelos sobre los mismos datos alineados, lo que hace comparables los resultados y reduce la necesidad de corpora propietarios.

---

## 8.4. Composición y modalidades

**[Documentado/Publicado]** La ficha oficial organiza el conjunto en **ocho carpetas de modalidades**:

```text
TerraMesh
├── train/   (DEM, LULC, NDVI, S1GRD, S1RTC, S2L1C, S2L2A, S2RGB)
├── val/     (mismas modalidades)
├── train_metadata.parquet
├── val_metadata.parquet
└── terramesh.py
```

Cada muestra espaciotemporal incluye **siete de las ocho modalidades alineadas**, porque cada muestra contiene **solo una versión de Sentinel-1** (GRD *o* RTC), dado que los conjuntos fuente solo aportan un producto S-1 cada uno (ficha oficial; artículo de TerraMesh, Tabla 2). Por eso la ficha oficial habla de «siete modalidades alineadas» en sus ejemplos, mientras que el resumen del artículo habla de «ocho modalidades espaciotemporalmente alineadas»: ambas cifras son coherentes (ocho en total, siete por muestra).

| Modalidad | Tipo | Escala | Rango típico | Bandas | Descripción |
| --- | --- | --- | --- | --- | --- |
| Sentinel-2 L1C | Óptica | DN | 0 – 10000 | 13 | Reflectancia de nivel superior a la atmósfera |
| Sentinel-2 L2A | Óptica | DN | 0 – 10000 | 12 | Reflectancia de nivel inferior a la atmósfera |
| Sentinel-2 RGB | Óptica | RGB | 0 – 255 | 3 | Producto RGB generado a partir de S-2 L2A |
| Sentinel-1 GRD | SAR | dB | −50 – +10 | 2 (VV, VH) | Retrodispersión calibrada, rango detectado en tierra |
| Sentinel-1 RTC | SAR | dB | −50 – +10 | 2 (VV, VH) | Retrodispersión corregida radiométricamente por terreno |
| NDVI | Índice | Índice | −1 – +1 | 1 | Índice de diferencia normalizada de vegetación |
| Copernicus DEM | Elevación | Metro | −400 – 8800 | 1 | Modelo digital basado en TanDEM-X |
| LULC | Anotación | Clases | 0 – 9 | 1 | Etiquetas de uso y cobertura del suelo de ESRI |

*Fuente: artículo de TerraMesh, Tabla 2; ficha oficial.*

**Diferencia frente a las modalidades de TerraMind.** **[Interpretación documentada]** Las **ocho carpetas de TerraMesh** (tabla anterior) son las modalidades que efectivamente forman parte del conjunto de datos. TerraMind, como modelo, añade en su preentrenamiento elementos que **no están en estas carpetas**: descripciones textuales (*captions*) generadas sintéticamente con LLaVA-Next a partir de RGB y etiquetas de Overture Maps, y coordenadas geográficas discretizadas (artículo de TerraMind, §3). Además, el artículo de TerraMind habla de «nueve modalidades geoespaciales» en el preentrenamiento; **[Pendiente]** no se encontró en las fuentes una enumeración única que explique exactamente esa cifra frente a las ocho carpetas del repositorio de datos, por lo que no se afirma aquí cuáles son esas nueve.

---

## 8.5. Cobertura geográfica y temporal

### Cobertura espacial

**[Publicado/Documentado]**

- **Distribución global**, con muestras en los cinco continentes; el conjunto incluye plenamente el hemisferio sur y al menos algunos ejemplos de geografías casi inexistentes en conjuntos menores, como la Antártida o islas remotas (artículo de TerraMesh, §3.3).
- **La distribución NO es homogénea:** las regiones muy homogéneas (desierto del Sahara, tundra) están **submuestreadas por diseño**; las zonas urbanas están sobrerrepresentadas: el área construida alcanza el **4 %** del conjunto, frente a menos del 1 % de la superficie terrestre global (artículo de TerraMesh, §3.3).
- La cobertura combina **MajorTOM-Core** (casi toda la superficie terrestre con un solo timestamp) y **SSL4EO-S12 v1.1** (cobertura geográfica menor pero multitemporal, centrada en ciudades).

**[Interpretación]** No debe asumirse una representación homogénea de todos los países ni de todas las regiones: el propio artículo de TerraMesh demuestra mediante mapa de calor que la densidad de muestras varía notablemente de una celda de un grado a otra, y el submuestreo de zonas áridas es intencional.

### Cobertura temporal

**[Publicado/Documentado]**

- El conjunto **cubre ocho años**, con la mayor parte de las adquisiciones **entre 2019 y 2023** (artículo de TerraMesh, §3.3, usando la marca temporal de Sentinel-2).
- **85 %** de las marcas temporales de S-1 y S-2 están alineadas dentro de una **ventana de una semana**; el **1 %** de las muestras presenta una diferencia temporal **superior a un año** por disponibilidad en la fuente MajorTOM (ficha oficial).
- Cada muestra conserva marcas temporales explícitas (`S2_time`, `S1_time` en los metadatos; la capa LULC se asocia al año ESRI más cercano a la fecha S-2).

**[Pendiente]** No se encontró información verificable sobre este aspecto en las fuentes consultadas: un rango de fechas exacto (fecha inicial y final) declarado por los responsables; la documentación lo expresa como «ocho años, mayoría 2019–2023».

---

## 8.6. Fuentes de información

**[Documentado]** Según la ficha oficial (sección *Acknowledgements*) y los artículos:

| Fuente | Producto utilizado | Licencia de la fuente |
| --- | --- | --- |
| **Sentinel-1 / Sentinel-2 (Copernicus)** | Imágenes GRD, RTC, L1C, L2A y RGB derivado | Incluidas vía SSL4EO-S12 y MajorTOM-Core |
| **SSL4EO-S12 v1.1** (Wang et al., 2023; Blumenstiel et al., 2025) | Datos S-1 y S-2 multitemporales | CC BY 4.0 |
| **MajorTOM-Core** (Francis & Czerkawski, 2024) | Cobertura casi global de S-1/S-2/DEM | CC BY-SA 4.0 |
| **ESRI / Impact Observatory / Microsoft** | Mapas anuales de uso y cobertura del suelo (LULC) | CC BY 4.0 |
| **Copernicus WorldDEM-30** (© DLR e.V. 2010-2014; © Airbus Defence and Space GmbH 2014-2018, facilitado por Copernicus/UE/ESA) | Modelo digital de elevación de 30 m | Todos los derechos reservados (ver condiciones en la ficha oficial) |
| **SEnSeI v2** (Francis, 2024) | Máscaras de nube y hielo usadas para augmentar LULC | Modelo de terceros, enlazado desde la ficha |
| **NDVI** | Calculado a partir de bandas de Sentinel-2 L2A | Derivado propio |

---

## 8.7. Estructura y organización

**[Documentado]**

- **Divisiones:** dos divisiones de nivel superior, `train/` y `val/`, cada una con una carpeta por modalidad.
- **Archivos de partición:** `train_metadata.parquet` y `val_metadata.parquet`, con columnas `tar`, `zarr`, `sample_id`, `split`, `center_lon`, `center_lat`, `crs`, `bounds`, `geometry`, `S2_time`, `S1_time`, `cloud_cover`.
- **Contenedores:** cada carpeta de modalidad contiene hasta **889 archivos shard (`.tar`)** de hasta **10 240 muestras** cada uno, leíbles con WebDataset. Los nombres distinguen el origen: `majortom_{split}_{id}.tar` para muestras de MajorTOM-Core y `ssl4eos12_*` para las de SSL4EO-S12.
- **Formato de muestra:** cada muestra es un archivo **Zarr Zip** (`.zarr.zip`, versión de Zarr 2) cargable con `zarr` o `xarray.load_zarr()`, con metadatos alineados (dimensiones `band`, `time`, `y`, `x`; coordenadas; CRS UTM; marcas temporales; `cloud_mask` para modalidades ópticas y LULC).
- **Mecanismo de acceso:** archivo `terramesh.py` incluido en el repositorio, con la función `build_terramesh_dataset()` (basada en WebDataset), que permite **transmisión (*streaming*) directa** desde Hugging Face o lectura local, selección de modalidades, transformaciones con `albumentations` y devolución opcional de metadatos.
- **Descarga:** mediante la CLI de Hugging Face (`hf download ibm-esa-geospatial/TerraMesh --repo-type dataset`), con opción `--include` para descargar solo un split o una modalidad.

---

## 8.8. Características técnicas

Todas las cifras siguientes proceden de la ficha oficial o del artículo de TerraMesh; no se han inferido ni estimado valores.

| Característica | Valor documentado | Fuente |
| --- | --- | --- |
| Número de muestras de entrenamiento | **9 089 536** | Artículo de TerraMesh, §3.3 |
| Número de muestras de validación | **89 088** | Artículo de TerraMesh, §3.3 |
| Número total de muestras (visor de Hugging Face, consulta del 14/10/2026) | ≈ **9,02 millones** de filas (≈ 8,93 M *train*; ≈ 89,1 mil *validación*) | Visor del repositorio en Hugging Face |
| Parches individuales (todas las modalidades consideradas) | **64 millones** | Artículo de TerraMesh |
| Tamaño total | **16 TB comprimido** (el artículo indica que una versión en TIF ocuparía 41 TB, factor de compresión 2,9) / «el conjunto completo requiere **17 TB** de almacenamiento» | Artículo de TerraMesh §3.3 / ficha oficial |
| Resolución espacial | **10 m**, sobre una cuadrícula basada en Sentinel-2 (DEM de origen 30 m remuestreado; LULC ESRI de 10 m) | Artículo de TerraMesh, §3.1 |
| Tamaño de parche | **264 × 264 píxeles** (S-1 y S-2); los ejemplos de *fine-tuning* recortan a 224 × 224 | Artículo de TerraMind §3; ficha de TerraMesh |
| Resolución temporal | Sin serie temporal por muestra: un timestamp por modalidad; cobertura de ocho años, mayoría 2019–2023; 85 % de pares S-1/S-2 alineados dentro de una semana | Artículo de TerraMesh; ficha oficial |
| Resolución espectral | S-2 L1C: 13 bandas; S-2 L2A: 12 bandas; S-2 RGB: 3 bandas; S-1 GRD/RTC: 2 bandas (VV, VH); NDVI: 1; DEM: 1; LULC: 1 | Artículo de TerraMesh, Tabla 2 |
| Formatos | Zarr Zip (xarray Dataset) dentro de archivos tar para WebDataset; metadatos en Parquet | Artículo de TerraMesh, §3.2; ficha oficial |
| Tipos de datos almacenados | S-2 y DEM en enteros de 16 bits; S-1 y NDVI en flotantes de 16 bits; LULC y S-2 RGB en 8 bits | Artículo de TerraMesh, §3.2 |
| Metadatos por muestra | Latitud/longitud (EPSG:4326), coordenadas por píxel en la zona UTM correspondiente, marcas temporales, máscara de nube SEnSeI v2 (clases: tierra, agua, nieve, nube fina, nube gruesa, sombra de nube, sin dato) y, en los Parquet, cobertura de nube y geometría | Artículo de TerraMesh, §3.3; ficha oficial |
| Modalidades | 8 carpetas; 7 modalidades alineadas por muestra | Ficha oficial; artículo de TerraMesh |
| Librerías declaradas | `webdataset`, `datasets`, `pandas`, `polars` (metadatos de la ficha) | Ficha oficial |

**Discrepancia documentada entre fuentes:** el artículo (agosto de 2025) reporta 9 089 536 muestras de entrenamiento; el visor actual (octubre de 2026) muestra ≈ 8,93 millones. **[Interpretación]** La explicación plausible es la actualización de la ficha del **20 de agosto de 2026**, que indica la eliminación de las muestras de entrenamiento que intersectaban el área de validación («menos del 2 % de las muestras de entrenamiento») tras añadirse un búfer de 5 km a las celdas de validación. Esta relación causal **no está declarada explícitamente** en la ficha y se presenta solo como interpretación.

---

## 8.9. Procesamiento y preparación

**[Publicado/Documentado]** El artículo de TerraMesh (§3.2) describe un pipeline de procesamiento con los siguientes pasos:

1. **Fusión de fuentes base:** unión de SSL4EO-S12 v1.1 (multitemporal, urbano-centrado) con un subconjunto de MajorTOM-Core (cobertura casi global), ambos con productos S-2 L1C, S-2 L2A y S-1 GRD/RTC, lo que simplifica la integración.
2. **Alineación espaciotemporal:** las modalidades se reproyectan y alinean a una cuadrícula común basada en Sentinel-2 a 10 m, y las adquisiciones se emparejan temporalmente (la mayoría dentro de una semana).
3. **Muestreo y filtrado:** submuestreo de MajorTOM-Core por clases de cobertura y ecorregiones; incorporación de las muestras urbanocentradas de SSL4EO-S12; reducción deliberada de regiones homogéneas (desiertos, tundra).
4. **Procesamiento por producto:** S-1 GRD con eliminación de ruido térmico y calibración; S-1 RTC con corrección radiométrica por terreno; generación de S-2 RGB a partir de L2A con normalización de percentiles (2 % y 98 %, mapeo a 8 bits); cálculo de NDVI a partir de S-2 L2A; remuestreado bilineal del Copernicus DEM de 30 m a la cuadrícula S-2; descarga del LULC ESRI del año más cercano al timestamp S-2 con reproyección correspondiente.
5. **Alineación temática del LULC:** se sustituye la clase «nube» de ESRI por «sin dato» y se incorporan las predicciones de nube y nieve/hielo del modelo SEnSeI v2 para mejorar la correspondencia entre imagen y etiqueta.
6. **Embalaje:** guardado de cada parche como Zarr Zip (xarray), barajado y almacenado en archivos tar para WebDataset, con estrategias de precisión reducida y compresión que reducen el conjunto de 41 TB a 16 TB.

**Decisión metodológica relevante:** los autores descartaron pseudoetiquetas de DynamicWorld (LULC) y ETH-GCHM (altura de dosel) por generalización deficiente entre años y versiones de procesamiento de Sentinel-2, documentando predicciones falsas concretas; en cambio, SEnSeI v2 generalizó bien a todos los años y fuentes (artículo de TerraMesh, §3.1).

---

## 8.10. Etiquetas y anotaciones

**[Documentado]** TerraMesh **no es un conjunto de datos etiquetado para tareas de clasificación o segmentación supervisada** en el sentido habitual. Lo que contiene es:

- **La capa LULC como modalidad**, con etiquetas de uso y cobertura del suelo de ESRI (rango de clases 0–9; el artículo describe «nueve clases de uso»), descrita como una modalidad temática más, co-registrada con las imágenes. La ficha declara que se usa con máscaras de nube/hielo augmentadas de SEnSeI v2.
- **Máscaras de nube** (metadatos) con siete clases: tierra, agua, nieve, nube fina, nube gruesa, sombra de nube y sin dato.
- **Metadatos geoespaciales**: coordenadas, CRS, límites, geometría, marcas temporales y cobertura de nube.

**[Pendiente]** No se encontró información verificable sobre este aspecto en las fuentes consultadas: **no hay etiquetas de anotación humana de segmentación o clasificación** para tareas aguas abajo dentro de TerraMesh. Esos conjuntos de referencia (Sen1Floods11, Burn Scars, PASTIS, MADOS, etc.) son conjuntos **distintos y externos**, utilizados para evaluar modelos preentrenados en TerraMesh, no partes de él.

---

## 8.11. Usos previstos

**[Documentado/Interpretación]** De acuerdo con la documentación disponible, los usos apropiados de TerraMesh son:

1. **Preentrenamiento de modelos fundacionales de observación de la Tierra** mediante objetivos de aprendizaje auto-supervisado o de correlación entre modalidades (uso declarado: se creó precisamente para ello y sirvió para preentrenar TerraMind).
2. **Investigación comparativa** sobre el efecto de escala, diversidad de modalidades y cobertura geográfica en el rendimiento aguas abajo (el artículo de TerraMesh aporta experimentos de ablación en esa dirección).
3. **Aprendizaje de representaciones multimodales** con datos ópticos, SAR, topográficos y temáticos alineados, aprovechando la estructura de *shards* y el soporte de streaming.
4. **Benchmarking de estrategias de fusión** entre modalidades alineadas (fusión temprana vs. tardía, entradas multimodales vs. unimodales).

La ficha oficial lo etiqueta como *pre-training*, *multimodal* y *Earth observation*, con tarea de «extracción de características de imagen».

**Uso no directo:** no está diseñado como conjunto de referencia para medir la exactitud de un sistema productivo; no contiene particiones de test con etiquetas de rendimiento (la división `val` es de validación, y desde agosto de 2026 incluye un búfer de 5 km para evitar fuga de datos con el entrenamiento).

---

## 8.12. Limitaciones y sesgos

### Problemas documentados por los responsables

**[Publicado/Documentado]**

1. **Representatividad espacial desigual por diseño.** Regiones homogéneas (Sahara, tundra) submuestreadas; zonas urbanas sobrerrepresentadas (4 % de área construida frente a <1 % global); el mapa de calor de la ficha muestra densidad de muestras desigual entre celdas de un grado (artículo de TerraMesh, §3.3; ficha oficial). Los autores lo presentan como una decisión deliberada para la diversidad temática, no como un defecto.
2. **Representatividad temporal concentrada.** Ocho años cubiertos, con concentración entre 2019 y 2023; un 1 % de muestras con desfase temporal mayor a un año entre modalidades (ficha oficial).
3. **Disponibilidad asimétrica entre modalidades.** Cada muestra tiene **solo una versión de Sentinel-1** (GRD o RTC), por lo que no es posible comparar GRD y RTC sobre la misma muestra exacta.
4. **Problema de alineación temporal corregido.** La ficha registra que el **5 de marzo de 2026** se corrigió un problema de alineación temporal en los datos S-1 GRD derivado de un error del conjunto fuente SSL4EO-S12 (incidencia pública en el repositorio de SSL4EO-S12 v1.1). Trabajos anteriores basados en versiones previas podrían haberse visto afectados.
5. **Fuga de datos entre entrenamiento y validación mitigada recientemente.** El **20 de agosto de 2026** se añadió un búfer de 5 km alrededor de las celdas de validación de MajorTOM y se eliminaron las muestras de entrenamiento que las intersectaban (<2 % del entrenamiento). Versiones anteriores del conjunto no incorporaban esta medida.
6. **Dependencia de fuentes externas y sus licencias.** Copernicus WorldDEM-30 se distribuye «todos los derechos reservados», y LULC procede de un tercero (ESRI/Impact Observatory): reutilizaciones integrales deben respetar cada licencia subyacente, no solo la CC BY-SA 4.0 del conjunto.
7. **Problemas de entorno de lectura.** La ficha advierte que el conjunto se creó con `zarr==2.18.0` y `numcodecs==0.15.1`; Zarr 3.0 tiene incompatibilidades hacia atrás y Zarr 2.18 es incompatible con NumCodecs ≥ 0.16.

### Riesgos potenciales que requerirían evaluación

**[Interpretación — no declarados como incidentes por los responsables]**

- **Vacíos de información para estudios territoriales locales.** Para un país o zona concreta, el número de muestras puede ser escaso o estar concentrado en épocas y tipos de paisaje particulares; cualquier estudio local debe verificar primero la cobertura efectiva usando `train_metadata.parquet` / `val_metadata.parquet` y el mapa de distribución.
- **Diferencias entre sensores y versiones de procesamiento.** El propio artículo documenta que el cambio de la cadena de procesamiento de Sentinel-2 en 2021 alteró significativamente los valores de reflectancia; trabajar con datos de distintas épocas exige comprobar la versión de procesamiento.
- **Clases temáticas con distribución sesgada.** Las proporciones publicadas (34,7 % pastizales, 25,0 % bosques, 4 % áreas construidas) difieren de las de SSL4EO-S12 (28,6 % y 14,2 %), lo que condiciona los modelos entrenados con él.
- **Consecuencias para estudios territoriales.** Usar estas capas (especialmente LULC) como si fueran catastro, cartografía base oficial o evidencia de titularidad extrapolaría su propósito previsto.
- **Limitaciones de acceso.** El tamaño total (16–17 TB) y los límites de descarga de Hugging Face condicionan el acceso completo; existe soporte de *streaming* como alternativa.

---

## 8.13. Licencia y acceso

**[Documentado]**

- **Licencia de TerraMesh: Creative Commons Attribution-ShareAlike 4.0 (CC BY-SA 4.0)**, declarada tanto en los metadatos de la ficha como en el artículo y en la sección *License* de la ficha oficial. Los autores la describen como «permisiva» y apta para adopción académica e industrial.
- **Licencias de las fuentes internas** (distintas de la del conjunto): SSL4EO-S12 v1.1 CC BY 4.0; MajorTOM-Core CC BY-SA 4.0; LULC ESRI/Impact Observatory/Microsoft CC BY 4.0; Copernicus WorldDEM-30 © DLR y © Airbus, «todos los derechos reservados», facilitado por Copernicus/UE/ESA.
- **Acceso:** público y gratuito desde Hugging Face, con CLI (`hf download`), *streaming* por URLs `resolve`/`raw` o descarga selectiva por `--include`. Se recomienda iniciar sesión en Hugging Face para evitar límites de descarga y limitar trabajadores (`--max-worker 4`).
- **Soporte:** pestaña de discusiones del repositorio, etiquetando a `@blumenstiel`.

**Confusión a evitar:** **la licencia CC BY-SA 4.0 de TerraMesh NO es la licencia del modelo TerraMind**, que es Apache 2.0 (código y pesos). Son componentes distintos con condiciones distintas; ver [`MODEL_CARD.md`](./MODEL_CARD.md), sección 7.12.

---

## 8.14. Consideraciones éticas

**[Interpretación fundamentada en la documentación]**

1. **Datos de observación a escala global.** TerraMesh contiene imágenes satelitales de todo el planeta. Aunque procede de misiones públicas (Copernicus) y no de capturas individuales, su uso masivo permite caracterizar actividades y lugares; los responsables no declaran restricciones específicas de privacidad en la ficha, pero conviene tratar derivados de alta resolución con responsabilidad.
2. **Atribución obligatoria.** La licencia CC BY-SA 4.0 exige dar crédito a los autores y a las fuentes originales, y las licencias de las capas internas añaden sus propios requisitos de atribución; reutilizar parcialmente el conjunto obliga a respetar ambas capas.
3. **Derivados con efecto territorial.** Los modelos entrenados con TerraMesh heredan sus sesgos de cobertura (urbano sobrerrepresentado, regiones áridas submuestreadas, concentración 2019–2023). Aplicar esos derivados a decisiones sobre suelo, ordenamiento o recursos naturales sin validación local puede perjudicar precisamente a las zonas menos representadas.
4. **Efecto de las correcciones de la versión.** Las correcciones de 2026 (alineación temporal y búfer de validación) implican que resultados obtenidos con versiones anteriores podrían no ser reproducibles con la versión actual; citar la fecha de descarga es una buena práctica de reproducibilidad.
5. **Supervisión humana.** Ningún uso de este conjunto sustituye la revisión por especialistas en el territorio de que se trate.

---

## 8.15. Autor de la ficha

- **Nombre:** Washington Noriel Yela Portocarrero.
- **Programa académico:** Ingeniería Catastral y Geodesia.
- **Institución:** Universidad Distrital Francisco José de Caldas.
- **Asignatura:** Inteligencia Artificial para el Territorio.
- **Fecha:** 14 de octubre de 2026.

**Aclaración:** el autor de esta ficha es quien **documenta** el conjunto de datos con fines académicos; no es el creador original de TerraMesh. Los responsables de TerraMesh son el equipo de IBM Research – Europe, la ESA Φ-lab y el Forschungszentrum Jülich, en el marco del proyecto FAST-EO (autores principales: Blumenstiel, Fraccaro, Marsocci, Jakubik, Maurogiovanni, Czerkawski, Sedona, Cavallaro, Brunschwiler, Bernabe-Moreno y Longépé).

---

## 8.16. Referencias

La lista completa y normalizada está en [`REFERENCES.md`](./REFERENCES.md). Las fuentes específicas de esta ficha son:

1. IBM ESA Geospatial. (s. f.). *TerraMesh* [Conjunto de datos]. Hugging Face. <https://huggingface.co/datasets/ibm-esa-geospatial/TerraMesh>
2. Blumenstiel, B., Fraccaro, P., Marsocci, V., Jakubik, J., Maurogiovanni, S., Czerkawski, M., … Longépé, N. (2025). *TerraMesh: A Planetary Mosaic of Multimodal Earth Observation Data*. CVPR Workshops 2025. <https://arxiv.org/abs/2504.11172>
3. Jakubik, J., Yang, F., Blumenstiel, B., Scheurer, E., Sedona, R., Maurogiovanni, S., … Longépé, N. (2025). *TerraMind: Large-Scale Generative Multimodality for Earth Observation*. ICCV 2025. <https://arxiv.org/abs/2504.11171>
4. Mitchell, M., Wu, S., Zaldivar, A., Barnes, P., Vasserman, L., Hutchinson, B., Spitzer, E., Raji, I. D., & Gebru, T. (2019). *Model Cards for Model Reporting*. FAT* '19, 220–229. <https://doi.org/10.1145/3287560.3287596>
5. Hugging Face. (s. f.). *Dataset card template* [Plantilla]. huggingface_hub. <https://github.com/huggingface/huggingface_hub/tree/main/src/huggingface_hub/templates>
6. Fuentes de datos citadas por la ficha oficial: SSL4EO-S12 v1.1, MajorTOM-Core, ESRI/Impact Observatory, Copernicus WorldDEM-30 y SEnSeI v2 (enlaces en la sección 8.6 y en `REFERENCES.md`).
