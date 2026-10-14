# Ficha del modelo (*Model Card*): TerraMind

Documento elaborado según los principios de Mitchell, Wu, Zaldivar, Barnes, Vasserman, Hutchinson, Spitzer, Raji y Gebru (2019), *Model Cards for Model Reporting*, y tomando como guía de estructura la plantilla `modelcard_template.md` del repositorio de Hugging Face `huggingface_hub`. Cada afirmación técnica se acompaña de la fuente que la respalda. Cuando la información no pudo verificarse, se indica de forma explícita.

**Convención de etiquetas usada en este documento:**

- **[Documentado]** → información publicada por los desarrolladores (repositorio oficial, fichas de Hugging Face, documentación).
- **[Publicado]** → resultados o hallazgos presentados en artículos científicos revisados.
- **[Interpretación]** → inferencia técnica razonable, señalada como tal, no afirmada por las fuentes.
- **[Pendiente]** → información no verificable en las fuentes consultadas.
- **[Experimentación propia requerida]** → resultados que solo podrían obtenerse ejecutando experimentos que no se realizaron en esta actividad.

---

## 7.1. Identificación del modelo

| Campo | Información |
| --- | --- |
| Nombre | TerraMind (familia de modelos **TerraMind 1.0**) |
| Desarrolladores | IBM (IBM Research – Europe), Agencia Espacial Europea (ESA Φ-lab) y Forschungszentrum Jülich, en el marco del proyecto FAST-EO |
| Organizaciones responsables | IBM, ESA Φ-lab, Forschungszentrum Jülich. El artículo lista además afiliaciones de ETH Zurich, NASA IMPACT y University of Iceland de algunos autores |
| Repositorio oficial de código | <https://github.com/IBM/terramind> |
| Publicación de pesos | Organización `ibm-esa-geospatial` en Hugging Face: <https://huggingface.co/ibm-esa-geospatial/TerraMind-1.0-base> |
| Versión documentada | Familia TerraMind 1.0. Fichas disponibles: `TerraMind-1.0-tiny`, `TerraMind-1.0-small`, `TerraMind-1.0-base`, `TerraMind-1.0-large`. En los artículos se reportan sobre todo las variantes **TerraMindv1-B** (base), **TerraMindv1-L** (large) y **TerraMindv1-B-single** (versión monomodal/escala única) |
| Variante utilizada en este proyecto | **Pendiente de confirmación.** La actividad no especifica la variante concreta; por ello esta ficha documenta la familia TerraMind 1.0 y utiliza los resultados de `v1-B` y `v1-L` como referencia, indicando siempre a qué variante corresponde cada cifra |
| Fecha de elaboración de la ficha | 14 de octubre de 2026 |
| Fuentes oficiales | Repositorio GitHub de IBM/terramind; fichas de Hugging Face de las cuatro variantes; documentación de TerraTorch; artículo Jakubik et al. (2025) |

**Referencia bibliográfica principal:** Jakubik, J., Yang, F., Blumenstiel, B., Scheurer, E., Sedona, R., Maurogiovanni, S., … Longépé, N. (2025). *TerraMind: Large-Scale Generative Multimodality for Earth Observation*. IEEE/CVF International Conference on Computer Vision (ICCV) 2025. Preprint: <https://arxiv.org/abs/2504.11171> (v5, actualizada el 17 de junio de 2026).

---

## 7.2. Descripción general

**TerraMind es un modelo fundacional generativo y multimodal para la observación de la Tierra**, presentado por sus autores como el primer modelo *any-to-any* de esta naturaleza: puede recibir como entrada subconjuntos variables de modalidades geoespaciales y devolver tanto representaciones (embebidos) como otras modalidades generadas artificialmente ([artículo, resumen](https://arxiv.org/abs/2504.11171); [repositorio oficial](https://github.com/IBM/terramind)). El prefijo *fundacional* indica que no está diseñado para una única tarea concreta, sino que se preentrena de forma masiva y no supervisada sobre datos heterogéneos para después adaptarse (*fine-tuning*) a múltiples tareas aguas abajo con relativamente pocos datos etiquetados.

Su carácter **multimodal** significa que no trabaja con un solo tipo de dato, sino con varias fuentes de información geoespacial simultáneas: imágenes ópticas de Sentinel-2 (niveles L1C y L2A, más una versión RGB), imágenes de radar sintético de apertura de Sentinel-1 (productos GRD y RTC), modelos digitales de elevación (DEM), índices de vegetación NDVI, capas de uso y cobertura del suelo (LULC), y también datos secuenciales como coordenadas geográficas y texto en forma de descripciones (*captions*) ([artículo, §1 y §3](https://arxiv.org/abs/2504.11171)). El artículo informa de que el preentrenamiento empleó **nueve modalidades geoespaciales** y que, a nivel de implementación, se usaron **seis modalidades a nivel de píxel y ocho a nivel de token** durante el preentrenamiento (§4.2).

**Papel en la observación de la Tierra.** TerraMind busca resolver un problema concreto de este campo: los modelos de aprendizaje automático tradicionales para teledetección se entrenan para una única tarea y una única combinación de sensores, mientras que en observación de la Tierra conviven fuentes muy heterogéneas (óptica, radar, topografía, cartografía temática) que rara vez están alineadas entre sí. TerraMind aprende, mediante reconstrucción de tokens enmascarados, correlaciones **entre modalidades** (por ejemplo, qué aspecto tendría una escena de radar vista una escena óptica) y **dentro de una misma modalidad** a dos niveles de abstracción (token y píxel), generando representaciones latentes que después pueden aprovecharse en muchas tareas ([artículo, §1 y §4.2](https://arxiv.org/abs/2504.11171)).

**Diferencia frente a un modelo de una sola tarea.** Según la documentación disponible, un modelo diseñado para una única tarea (por ejemplo, segmentación de inundaciones) devuelve una salida predeterminada para esa tarea y requiere reentrenarse para otra. TerraMind, en cambio, se usa de tres formas documentadas: (i) como codificador (*backbone*) que produce embebidos para cualquier cabeza de red aguas abajo; (ii) como generador *any-to-any* de modalidades; y (iii) con la técnica *Thinking-in-Modalities* (TiM), que genera modalidades artificiales intermedias para mejorar la tarea final ([ficha oficial del modelo](https://huggingface.co/ibm-esa-geospatial/TerraMind-1.0-base); [repositorio oficial](https://github.com/IBM/terramind)).

---

## 7.3. Arquitectura y funcionamiento

### Enfoque arquitectónico

**[Documentado]** TerraMind utiliza una **arquitectura de transformador simétrica codificador–decodificador**, propuesta originalmente en el modelo 4M (Mizrahi et al., 2023; es la referencia [53] del artículo) y adaptada al dominio geoespacial, que procesa secuencias de tokens multimodales y admite además entradas a nivel de píxel ([artículo, §4.2](https://arxiv.org/abs/2504.11171); [ficha del modelo](https://huggingface.co/ibm-esa-geospatial/TerraMind-1.0-base)).

Sus rasgos documentados son:

- **Representación de doble escala (*dual-scale*):** el modelo trabaja simultáneamente con *tokens* discretos (información contextual de alto nivel) y con *patches* de píxel (información espacial fina). Ambos representan la misma área de 16 × 16 píxeles: el token la codifica como un entero discreto y el *patch* conserva los valores flotantes del sensor ([artículo, §4.2](https://arxiv.org/abs/2504.11171)).
- **Fusión temprana (*early fusion*):** los tokens y los *patches* se integran en el espacio de entrada antes de la predicción, a diferencia de enfoques de fusión tardía que procesan cada modalidad por separado y combinan las salidas al final. El artículo reporta que la fusión a nivel de token supera a la fusión tardía en tareas de mapeo de aguas (Tabla 2 del artículo).
- **Proyecciones lineales por *patch*:** las entradas de píxel (imágenes satelitales y DEM) se proyectan con proyecciones lineales aprendidas por *patch* de 16 × 16, análogas a las de un Vision Transformer (ViT).
- **Tokenizers específicos de cada modalidad:** previamente al preentrenamiento del modelo se entrenan tokenizers unimodales. Para modalidades tipo imagen se usan autoencoders con cuantización de escala finita (**FSQ**) en el cuello de botella, con codificador ViT, decodificador UNet con paso de difusión y vocabulario de 16K códigos (4K para LULC, modalidad más simple). Para datos secuenciales se emplea un tokenizador WordPiece adaptado y tokens especiales de coordenadas (latitud y longitud redondeadas al cuarto de grado más cercano) ([artículo, §4.1 y §9.1](https://arxiv.org/abs/2504.11171)).
- **Objetivo de preentrenamiento:** reconstrucción de tokens enmascarados en espacio de token, muestreando entradas y objetivos de una distribución de Dirichlet. Este diseño obliga al modelo a funcionar con la ausencia total de modalidades, algo habitual en observación de la Tierra ([artículo, §4.2](https://arxiv.org/abs/2504.11171)).
- **Decodificador de difusión:** permite devolver los tokens generados al espacio de imagen o de texto, lo que sustenta la capacidad de generación *any-to-any* y la generación encadenada (una modalidad generada alimenta la siguiente).

### Modalidades que puede manejar

**[Documentado]** Según la ficha oficial, las **entradas en bruto** admitidas por el modelo son: `S2L2A`, `S2L1C`, `S1GRD`, `S1RTC`, `DEM` y `RGB`, con posibilidad de especificar subconjuntos de bandas (por ejemplo, solo seis bandas de S2L2A) y de **omitir modalidades ausentes** ([ficha del modelo base](https://huggingface.co/ibm-esa-geospatial/TerraMind-1.0-base)). El artículo añade que durante el preentrenamiento se emplearon seis modalidades a nivel de píxel y ocho a nivel de token (§4.2), incluyendo NDVI y LULC a nivel de token, además de coordenadas y texto.

### Entradas y salidas

**[Documentado]**

- **Salida del *backbone* (codificador):** embebidos por *patch*. Con entradas de 224 × 224 píxeles, la ficha oficial muestra una salida de forma `B, 196, 768` (196 *patches* = 14 × 14, dimensión de embebido 768). Los embebidos de varias modalidades se combinan por defecto con media; también están disponibles `max`, `concat`, `dict` y `None` como estrategias de fusión ([ficha del modelo base](https://huggingface.co/ibm-esa-geospatial/TerraMind-1.0-base)).
- **Salida generativa:** registrando el modelo completo (`terramind_v1_base_generate`) se especifican `output_modalities` y un número de pasos de difusión (`timesteps`); la salida es un diccionario de tensores con las modalidades pedidas. La ficha oficial advierte explícitamente que **generadas no son reconstrucciones**, sino «imágenes mentales» que representan lo que el modelo imagina para esa modalidad.

### Variantes disponibles

| Variante | Dónde se publica | Notas documentadas |
| --- | --- | --- |
| `TerraMind-1.0-tiny` | Hugging Face (`ibm-esa-geospatial`) | Menor tamaño de *backbone*; el artículo menciona mejoras al pasar de *tiny* a *small* a *base* |
| `TerraMind-1.0-small` | Hugging Face | Usada en varios *notebooks* de ejemplo |
| `TerraMind-1.0-base` (`TerraMindv1-B`) | Hugging Face | Variante con más resultados publicados; preentrenada con 500B de tokens |
| `TerraMind-1.0-large` (`TerraMindv1-L`) | Hugging Face | Preentrenada con 500B de tokens, 10 días en 32 NVIDIA A100 |
| `TerraMindv1-B-single` | Reportada en el artículo | Modelo de escala única y principalmente a nivel de token, con S-2 L2A como única modalidad de píxel, preentrenado con más de un millón de muestras |
| `..._tim` (p. ej. `terramind_v1_base_tim`) | Vía TerraTorch | Variantes para *fine-tuning* con *Thinking-in-Modalities*; modalidad TiM por defecto: `LULC` |

**[Pendiente]** No se encontró información verificable sobre este aspecto en las fuentes consultadas: **el número de parámetros de cada variante** no aparece publicado en el artículo ni en las fichas de Hugging Face consultadas. (El archivo de pesos de la variante *base*, `TerraMind_v1_base.pt`, tiene un tamaño de 1 518 713 958 bytes ≈ 1,5 GB según la API de Hugging Face, pero de ese dato no se extrae aquí ningún recuento de parámetros.)

### Funcionamiento general

**[Documentado]** El flujo documentado es: (1) entrenamiento previo de los tokenizers por modalidad; (2) tokenización de las modalidades de TerraMesh; (3) preentrenamiento del codificador–decodificador con reconstrucción de tokens enmascarados sobre subconjuntos aleatorios de tokens y *patches*; (4) uso posterior como codificador congelado y cabeza aguas abajo, como generador *any-to-any*, o con TiM durante el ajuste fino y la inferencia ([artículo, §4](https://arxiv.org/abs/2504.11171); [repositorio oficial](https://github.com/IBM/terramind)).

### Notas de verificación sobre discrepancias entre fuentes

1. **Duración del preentrenamiento de `v1-B`:** el §4.2 indica «500B tokens en 6 días sobre 32 NVIDIA A100», mientras que el §9.2 (apéndice) indica «12 días sobre 32 A100, es decir, 9 216 GPU-horas». Ambas cifras proceden del mismo artículo; aquí se reportan sin conciliarlas.
2. **Número de modalidades:** el resumen del artículo habla de «nueve modalidades geoespaciales» y el §8 del apéndice habla de «nueve modalidades centrales» de TerraMesh, mientras que el artículo de TerraMesh y su ficha oficial describen **ocho modalidades** (con siete alineadas por muestra). No se encontró en las fuentes una enumeración única que explique la diferencia; se marca como pendiente de verificación.
3. **Vocabulario del tokenizer:** el §4.1 indica vocabulario de 16K códigos, mientras que el §9.1 describe una cuantización FSQ con niveles 8-8-8-6-5. Se reporta cada dato con su sección de origen.

---

## 7.4. Objetivos y usos previstos

### Capacidades generales del modelo

**[Documentado]** La documentación oficial describe tres modos de uso ([ficha del modelo](https://huggingface.co/ibm-esa-geospatial/TerraMind-1.0-base); [repositorio oficial](https://github.com/IBM/terramind)):

1. **Obtención de embebidos (*backbone*)** para cualquier red aguas abajo, con entradas multimodales y tolerancia a modalidades ausentes.
2. **Generación *any-to-any*:** generar una o varias modalidades a partir de cualquier subconjunto de entrada, incluida la generación encadenada (por ejemplo, óptica → DEM → LULC) para mantener coherencia entre productos generados.
3. **Fine-tuning con *Thinking-in-Modalities*:** el modelo genera primero modalidades artificiales (por defecto LULC) y luego la tarea final usa tanto las entradas originales como esas modalidades generadas.

### Tareas demostradas en publicaciones

**[Publicado]** Las tareas con resultados cuantitativos publicados por los desarrolladores son: segmentación semántica de uso/cubierta del suelo, detección de cuerpos de agua e inundaciones, mapeo de quemaduras por incendio, segmentación agrícola y tipificación de cultivos, detección de cambios (SpaceNet 7), detección de residuos marinos (MADOS), clasificación *few-shot* (EuroSAT, METER-ML), predicción de geolocalización, reconstrucción/generación entre modalidades (S-1 ↔ S-2, DEM, NDVI) y mapeo de agua en cero disparos (*zero-shot*). Detalle en la sección 7.9.

### Aplicaciones que requieren adaptación o evaluación adicional

**[Interpretación]** Las capacidades anteriores están demostradas en conjuntos de evaluación concretos (mayoritariamente de segmentación con sensores Sentinel-2/HLS, radar y datos globales). Cualquier uso en una región, sensor, época o tipo de objeto no cubierto por esas evaluaciones —por ejemplo, catastro, predial, ordenamiento territorial o evaluación ambiental específica— **requiere adaptación y validación propias** ([Experimentación propia requerida]). No se encontró ninguna evaluación de TerraMind sobre datos catastrales colombianos (ver sección 7.10 y 7.11).

---

## 7.5. Usuarios previstos

**[Interpretación]** Los perfiles para los que la documentación resulta pertinente, sin que los desarrolladores publiquen una lista oficial de usuarios recomendados, son:

- **Investigadores en modelos fundacionales y aprendizaje automático para teledetección**, que buscan un codificador multimodal preentrenado o un modelo generador de modalidades.
- **Especialistas en observación de la Tierra y sensores**, que necesitan combinar óptica, radar, topografía y capas temáticas sin construir su propio corpus de preentrenamiento.
- **Profesionales geoespaciales y de SIG**, interesados en representaciones útiles para tareas de cobertura del suelo, vegetación, hidrología o detección de cambios, siempre que validen localmente los resultados.
- **Equipos de ingeniería y ciencia de datos** que integran modelos preentrenados mediante herramientas como TerraTorch.

La ficha oficial no declara usuarios recomendados ni sectores prioritarios; esta sección es una inferencia razonable a partir de las tareas y *benchmarks* documentados.

---

## 7.6. Usos fuera del alcance previsto

Los siguientes usos **no deben darse por válidos sin evidencia adicional y validación específica**. Son **precauciones de uso derivadas de las limitaciones documentadas del modelo y del estado de sus evaluaciones, no una lista de prohibiciones oficiales de los desarrolladores**: no se encontró en las fuentes consultadas una sección de «usos no recomendados» publicada por IBM, la ESA o el proyecto FAST-EO.

1. **Decisiones catastrales automatizadas sin validación humana.** No existe evidencia de que TerraMind haya sido evaluado para tareas catastrales; sus salidas son embebidos o productos generados, no observaciones certificadas.
2. **Delimitación predial con precisión jurídica.** Las modalidades de TerraMesh tienen resolución de 10 m y las anotaciones de cubierta del suelo son de uso general; ninguna resolución documentada respalda límites prediales.
3. **Identificación o confirmación de derechos de propiedad.** El modelo no trabaja con información registral ni catastral de ninguna naturaleza.
4. **Evaluaciones ambientales sin verificación.** Los productos generados (por ejemplo DEM o NDVI sintéticos) son estimaciones del modelo; el propio artículo reporta errores notables en la generación de DEM y baja alineación estructural en la generación de radar a partir de óptica (Tabla 3 del artículo).
5. **Decisiones administrativas basadas exclusivamente en predicciones del modelo.** El rendimiento *zero-shot* publicado (45,40 IoU en mapeo de agua con S-2 para `v1-B`) queda muy por debajo del rendimiento con ajuste fino (82,17 IoU del modelo DeCUR de referencia en la misma tarea; Tabla 4 del artículo).
6. **Aplicaciones en regiones o condiciones no cubiertas por las evaluaciones.** Los conjuntos evaluados (PANGAEA y derivados) no incluyen, según las fuentes consultadas, ninguna región o tarea específica de Colombia; las muestras de entrenamiento están distribuidas globalmente pero con un muestreo deliberado que subrepresenta regiones homogéneas (ver `DATASET_CARD.md`, secciones 8.5 y 8.12).
7. **Tratar las modalidades generadas como observaciones verificadas.** La ficha oficial advierte que las generaciones no son reconstrucciones, sino «imágenes mentales» del modelo.

---

## 7.7. Datos de entrenamiento

**[Documentado]** Todas las versiones de TerraMind se preentrenaron sobre **TerraMesh** o un subconjunto de él ([artículo, apéndice §8](https://arxiv.org/abs/2504.11171); [ficha del modelo](https://huggingface.co/ibm-esa-geospatial/TerraMind-1.0-base)). Datos publicados:

| Aspecto | Información documentada | Fuente |
| --- | --- | --- |
| Conjunto principal de preentrenamiento | TerraMesh: ≈ 9 millones de muestras globalmente distribuidas, espaciotemporalmente alineadas, cada una de 264 × 264 píxeles a 10 m | Artículo §3; ficha de TerraMesh |
| Tokens de preentrenamiento | 500B tokens para `v1-B`, `v1-L` y `v1-B-single` | Artículo §4.2 |
| Subconjunto alternativo | `TerraMindv1-B-single` se preentrenó con más de 1 millón de muestras (ubicaciones de SSL4EO-S12 v1.1) y solo cuatro modalidades de imagen (S-2 L2A, S-1 GRD, DEM, LULC), más *fine-tuning* continuo con *captions* | Artículo §3 y §4.2 |
| *Captions* (descripciones textuales) | Generados de forma sintética con LLaVA-Next a partir de imágenes RGB de Sentinel-2 y etiquetas de Overture Maps | Artículo §3 |
| Calidad de los *captions* | Evaluación de expertos sobre 1 300 *captions*: 69 % sin alucinaciones; puntuación media de completitud 3,87 en escala 0–5 | Artículo, apéndice §8 |
| Fuentes de los datos satelitales | SSL4EO-S12 v1.1 y MajorTOM-Core (S-1, S-2); ESRI/Impact Observatory (LULC); Copernicus WorldDEM-30 (DEM); SEnSeI v2 (máscaras de nube) | Ficha de TerraMesh; artículo de TerraMesh |

**Diferencia entre preentrenamiento y evaluación.** **[Publicado]** Los conjuntos usados para evaluar el modelo (PANGAEA y sus tareas derivadas: Burn Scars, MADOS, PASTIS, Sen1Floods11, FBP, DEN, CTM-SS, SpaceNet 7, AI4Farms; EuroSAT; METER-ML; SA Crop; Sen1Floods11) son **conjuntos de evaluación aguas abajo, distintos de los datos de preentrenamiento**. La ficha de TerraMesh indica además que se añadió un búfer de 5 km alrededor de las celdas de validación de MajorTOM (agosto de 2026) precisamente para reducir la fuga de datos entre entrenamiento y validación.

**[Pendiente]** No se encontró información verificable sobre este aspecto en las fuentes consultadas: desglose completo de cuántas muestras de cada región, año o clase del conjunto de entrenamiento se usaron efectivamente en cada variante (más allá de las cifras generales anteriores).

---

## 7.8. Procedimiento de entrenamiento

**[Publicado/Documentado]** La información publicada permite reconstruir el procedimiento a nivel general, pero **no el proceso completo**.

**Lo que sí está documentado:**

- **Preentrenamiento en dos etapas:** primero se preentrenan tokenizers unimodales (por modalidad, con DDP sobre 4 GPUs, 100 épocas sobre la modalidad correspondiente de TerraMesh, cuantización FSQ, decodificador de difusión con 1 000 pasos y programación lineal de betas, precisión mixta fp16, optimizador AdamW); después se preentrena el codificador–decodificador de TerraMind con reconstrucción de tokens enmascarados ([artículo, §4 y §9.1](https://arxiv.org/abs/2504.11171)).
- **Cómputo del modelo principal:** 500B tokens sobre 32 NVIDIA A100; para `v1-B` el artículo indica 6 días (§4.2) y 12 días / 9 216 GPU-horas (§9.2); para `v1-L`, 10 días (§4.2). El artículo estima unas 30 000 GPU-horas en total si se suman los experimentos de preentrenamiento.
- **Hiperparámetros declarados:** tasa de aprendizaje base de 2e-4 con programador de coseno (*cosine annealing*) configurado para 500B tokens; comparación previa de tamaños de lote global de 2K, 4K y 8K; utilización de GPU del 85 % o más en `v1-L`; muestreo de entradas y objetivos desde una distribución de Dirichlet.
- **Validación durante el preentrenamiento:** pérdidas finales por modalidad publicadas (Tabla 9 del artículo): por ejemplo, `v1-B` con 5.67 en S-2 L2A y `v1-L` con 5.34, frente a 9.68 del modelo aleatorio teórico.

**Lo que no está documentado:** **[Pendiente]** No se encontró información verificable sobre este aspecto en las fuentes consultadas: valor final concreto del tamaño de lote, semillas de ejecución, duración exacta no ambigua, coste energético o huella de carbono declarada, listado completo de hiperparámetros de cada variante y procedimiento de selección de modelos. En consecuencia, **no puede reconstruirse el entrenamiento completo a partir de la información pública** y no se afirma aquí haberlo reproducido.

---

## 7.9. Evaluación del modelo

**Procedimiento documentado.** **[Publicado]** La evaluación principal sigue el protocolo de **PANGAEA** (Marsocci et al., 2024, <https://arxiv.org/abs/2412.04204>), un *benchmark* comunitario para modelos fundacionales geoespaciales en el que los modelos preentrenados se ajustan con el codificador congelado y una cabeza UperNet, y se reporta mIoU por conjunto y rango promedio. El artículo indica que, **siguiendo la recomendación de los autores de PANGAEA, se excluyeron las tareas xView2 y BioMassters por no poder reproducirse los rendimientos reportados**, de modo que se evalúan nueve de las once tareas. Se distinguen resultados **unimodales** (marcados con «uni» en la tabla original) y **multimodales**.

### Tabla A. PANGAEA, nueve tareas (resultados oficiales del artículo de TerraMind)

Métrica: **mIoU (%)**; mayor es mejor. *Rank* promedio: menor es mejor. Fuente: Jakubik et al. (2025), Tabla 6.

| Conjunto de evaluación (tarea) | Métrica | TerraMindv1-B | TerraMindv1-L |
| --- | --- | --- | --- |
| Burn Scars (uni) | mIoU | 82.42 | 82.93 |
| MADOS (uni) | mIoU | 69.52 | 75.57 |
| PASTIS | mIoU | 40.51 | 43.13 |
| Sen1Floods11 | mIoU | 90.62 | 90.78 |
| FBP (uni) | mIoU | 59.72 | 63.38 |
| DEN (uni) | mIoU | 37.87 | 37.89 |
| CTM-SS | mIoU | 55.80 | 55.04 |
| SpaceNet 7 (uni) | mIoU | 60.61 | 59.98 |
| AI4Farms (uni) | mIoU | 28.12 | 27.47 |
| **Promedio (9 tareas)** | **mIoU medio** | **58.35** | **59.57** |
| **Promedio (9 tareas)** | **Rango medio** ↓ | **3.94** | **3.44** |

Variante del modelo: `TerraMindv1-B` y `TerraMindv1-L`. El mismo artículo reporta que otros modelos fundacionales quedan por debajo (por ejemplo, CROMA 55.29 de mIoU medio, rango 6.61; Prithvi 1.0 100M 51.00; DOFA 53.59), y que los modelos de tarea específica entrenados desde cero obtienen 57.58 (U-Net) y 54.13 (ViT) de mIoU medio, de modo que TerraMind supera a los U-Net por 1 punto porcentual en promedio en configuración unimodal y por 4 puntos en multimodal (§11.3 del artículo).

### Tabla B. PANGAEA, seis tareas (resultados publicados en el artículo de TerraMesh)

Métrica: **mIoU (%)**. Fuente: Blumenstiel et al. (2025), Tabla 3; coincide con la ficha oficial de TerraMesh, que reporta 66,6 % y rango 2,33.

| Conjunto de evaluación (tarea) | Métrica | TerraMind-B (entrenado con TerraMesh) | TerraMind-B (modelo ablativo: solo ubicaciones SSL4EO-S12) |
| --- | --- | --- | --- |
| Burn Scars | mIoU | 82.42 | 84.00 |
| MADOS | mIoU | 69.52 | 65.01 |
| PASTIS | mIoU | 40.51 | 40.80 |
| Sen1Floods11 | mIoU | 90.62 | 90.32 |
| CTM-SS | mIoU | 55.80 | 52.66 |
| SpaceNet 7 | mIoU | 60.61 | 59.71 |
| **Promedio (6 tareas)** | **mIoU medio** | **66.58** | **65.42** |
| **Promedio (6 tareas)** | **Rango medio** ↓ | **2.33** | **3.00** |

**Aclaración metodológica importante:** las cifras 58.35 (Tabla A) y 66.58 (Tabla B) **no son contradictorias**, sino que corresponden a promedios sobre conjuntos de tareas distintos (nueve tareas frente a seis) dentro de la misma familia de resultados. Presentar 66.58 % como promedio global de TerraMind sin indicar que proviene de seis tareas concretas sería inexacto.

### Tabla C. Mapeo de cuerpos de agua en configuración *zero-shot*

Métrica: **IoU en la clase agua (%)**. Fuente: Jakubik et al. (2025), Tabla 4. Conjunto: Sen1Floods11.

| Modelo | Entrada | Tipo | IoU agua |
| --- | --- | --- | --- |
| TerraMindv1-B | S-2 | *zero-shot* | 45.40 |
| TerraMindv1-B-single | S-2 | *zero-shot* | 69.75 |
| Baseline (referencia) | S-2 | ajuste fino | 31.25 |
| Prithvi 2.0 300M | S-2 | ajuste fino | 80.97 |
| DeCUR | S-2 | ajuste fino | 82.17 |

Variante: `TerraMindv1-B` y `v1-B-single`. El artículo destaca que ningún modelo de referencia puede operar en *zero-shot*, pero también que el rendimiento sin ajuste fino queda por debajo del estado del arte con ajuste fino.

### Tabla D. Clasificación *few-shot* (1 disparo, 5 vías)

Métrica: **precisión media (%)**, promediada sobre 200 corridas, sin actualización de pesos. Fuente: Jakubik et al. (2025), Tabla 5.

| Modelo | Entrada | EuroSAT | METER-ML |
| --- | --- | --- | --- |
| CLIP-ViT-B/16 | S-2 RGB | 57.00 | 29.15 |
| DeCUR | S-2 L1C | 50.54 | 27.87 |
| Prithvi 1.0 100M | S-2 L1C | 60.11 | 26.08 |
| Prithvi 2.0 300M | S-2 L1C | 61.06 | 28.26 |
| **TerraMindv1-B** | S-2 L1C | **70.83** | **33.90** |
| TerraMindv1-B | NAIP | — | 32.23 |

Variante: `TerraMindv1-B`.

### Tabla E. Aporta de la entrada multimodal (PANGAEA, mIoU %)

Fuente: Jakubik et al. (2025), Tabla 7. Variante: `TerraMindv1-B`.

| Entrada | PASTIS | Sen1Floods11 | CTM-SS |
| --- | --- | --- | --- |
| S-1 (radar) | 20.04 | 80.39 | 24.45 |
| S-2 (óptica) | 40.20 | 89.57 | 50.90 |
| S-1 + S-2 (multimodal) | 40.51 | 90.62 | 55.80 |

### Tabla F. *Thinking-in-Modalities* (TiM)

Fuente: Jakubik et al. (2025), Tablas 8 y 13. Variante: `TerraMindv1-B`.

| Tarea / conjunto | Entrada | Métrica | Resultado |
| --- | --- | --- | --- |
| Sen1Floods11 (segmentación de agua) | S-1 | IoU agua / mIoU | 68.00 / 81.06 |
| Sen1Floods11 | S-2 | IoU agua / mIoU | 82.26 / 89.70 |
| Sen1Floods11 | S-1 + LULC generada (TiM) | IoU agua / mIoU | 72.25 / 83.65 |
| Sen1Floods11 | S-2 + LULC generada (TiM) | IoU agua / mIoU | 84.75 / 91.14 |
| SA Crop (tipificación de cultivos, Sudáfrica) | S-2 | mIoU | 41.87 |
| SA Crop | S-2 + LULC generada (TiM) | mIoU | 42.74 |

### Tabla G. Calidad de generaciones *any-to-any*

Métricas: MAE y RMSE (menor es mejor), SSIM y PSNR (mayor es mejor). Conjunto de validación global no visto, 10 pasos de difusión. Unidades físicas: metros (DEM), reflectancia (S-2), dB (S-1). Fuente: Jakubik et al. (2025), Tabla 3. Variante: `TerraMindv1-B`.

| De → A | MAE | RMSE | SSIM | PSNR |
| --- | --- | --- | --- | --- |
| S-1 GRD → S-2 L2A | 0.074 | 0.116 | 0.750 | 26.210 |
| S-1 GRD → DEM | 163.0 | 320.8 | 0.878 | 20.694 |
| S-1 GRD → NDVI | 0.180 | 0.225 | 0.438 | 18.990 |
| S-1 RTC → S-2 L2A | 0.113 | 0.194 | 0.695 | 24.251 |
| S-1 RTC → DEM | 298.8 | 799.2 | 0.873 | 20.009 |
| S-1 RTC → NDVI | 0.172 | 0.211 | 0.465 | 19.529 |
| S-2 L2A → S-1 GRD | 2.942 | 3.877 | 0.531 | 28.678 |
| S-2 L2A → S-1 RTC | 2.636 | 3.391 | 0.430 | 28.993 |
| S-2 L2A → DEM | 215.8 | 745.5 | 0.942 | 20.616 |

### Resultados adicionales relevantes

**[Publicado]** En el artículo de TerraMind se reporta también: (i) mapeo de agua por generación de LULC en Sen1Floods11, con 69.87 IoU para `TerraMindv1-B-single` en configuración *zero-shot*, frente a 71.98 del método de pseudoetiquetado con DynamicWorld y 76.32 con ajuste fino (Tabla 12); (ii) reconstrucciones de los tokenizers sobre 8 500 muestras de validación de SSL4EO-S12 (Tabla 10), con SSIM de 0.851 para S-2 L2A, 0.565 para S-1 GRD y 0.466 para S-1 RTC; y (iii) pérdidas de validación del preentrenamiento (Tabla 9).

### Evaluaciones de terceros

**[Pendiente]** No se encontró información verificable sobre este aspecto en las fuentes consultadas: **no se localizaron evaluaciones independientes de terceros** (por ejemplo, estudios de organizaciones ajenas a IBM/ESA que comparen TerraMind con otros modelos de forma independiente). Todos los resultados anteriores provienen de los equipos desarrolladores (artículos y fichas oficiales).

### Resultados no obtenidos en esta actividad

**[Experimentación propia requerida]** No se ejecutó ningún experimento, entrenamiento ni evaluación para elaborar esta ficha. Ninguna cifra de este documento es producto de una corrida propia; todas proceden de las fuentes citadas. Tampoco se evaluó a TerraMind sobre datos de Colombia ni sobre tareas catastrales.

---

## 7.10. Limitaciones técnicas

### Limitaciones reconocidas en las publicaciones

**[Publicado]**

1. **Calidad desigual entre modalidades generadas.** El artículo describe una asimetría en la generación: de radar a óptica se logra fidelidad razonable en SSIM/PSNR con degradación perceptual; de óptica a radar se obtienen PSNR altos pero SSIM bajos, es decir, salidas visualmente plausibles con débil alineación estructural. La generación de DEM resulta «estructuralmente muy fuerte pero ruidosa», y los autores concluyen que **el nivel de altitud es difícil de inferir para el modelo** (§5.2 y Tabla 3).
2. **Sensibilidad de los tokenizers.** Los tokenizers son sensibles a la tasa de aprendizaje (tasas altas provocan pérdidas NaN, tasas bajas producen imágenes borrosas); aumentar el vocabulario por encima de 16K en modalidades con más de tres bandas introdujo artefactos; la reconstrucción de datos multibanda es de menor detalle que la de canal único (§10).
3. **Rendimiento inferior en cero disparos frente a modelos ajustados.** En mapeo de agua, `TerraMindv1-B` obtiene 45.40 IoU *zero-shot* frente a 82.17 de un modelo con ajuste fino (Tabla 4), y la generación de LULC no supera al pseudoetiquetado con DynamicWorld (Tabla 12). Los autores presentan esto como resultado esperado de un modelo no especializado.
4. **Reconstrucción del radar con desviaciones estructurales.** Las reconstrucciones de S-1 GRD y S-1 RTC son «visualmente plausibles» pero con desviaciones estructurales moderadas (SSIM 0.565 y 0.466; Tabla 10), lo que afecta a cualquier tarea aguas abajo que dependa de esas reconstrucciones.
5. **Exclusiones y problemas de reproducibilidad en la evaluación.** Se excluyeron xView2 y BioMassters de PANGAEA por no poder reproducirse los rendimientos reportados.
6. **Problema técnico conocido en la documentación del repositorio.** El repositorio oficial advierte que, en macOS, el decodificador UNet puede fallar por las normalizaciones por lotes (*batch norms*) durante el entrenamiento en MPS; recomienda usar un decodificador FCN o instalar una versión corregida de TerraTorch.
7. **Dependencia de las modalidades de entrada.** El rendimiento depende de qué modalidades estén disponibles: en PANGAEA, la entrada combinada S-1 + S-2 supera de forma consistente a cada entrada unimodal, y la ausencia de LULC es el escenario donde la fusión temprana aporta más (Tablas 7 y 2 del artículo). El modelo admite entradas ausentes, pero **[Interpretación]** es razonable esperar que su rendimiento disminuya con menos información, algo que las fuentes no cuantifican de forma sistemática para todos los casos.

### Consideraciones derivadas del tipo de modelo

**[Interpretación — no son afirmaciones de los desarrolladores]**

- **Generalización geográfica y temporal.** El preentrenamiento se basa en datos de 2019–2023 en su mayoría (ocho años cubiertos en TerraMesh); la aplicación a épocas posteriores, sensores distintos o regiones con coberturas atípicas no está documentada y exige verificación.
- **Calidad y resolución de las observaciones.** Las modalidades de TerraMesh están alineadas a una cuadrícula de 10 m basada en Sentinel-2, con DEM de origen 30 m resempleado; estas diferencias de resolución nativa limitan la fidelidad que puede esperarse en detalles finos.
- **Incertidumbre de las predicciones.** Las fichas no publican intervalos de incertidumbre, calibración de probabilidades ni métricas de confianza; cualquier decisión basada en el modelo debe incorporar validación y cuantificación de incertidumbre propias ([Experimentación propia requerida]).
- **Necesidad de validación específica por tarea.** Los resultados publicados corresponden a tareas y conjuntos concretos; no extrapolan automáticamente a otra región, sensor o época.

**[Pendiente]** No se encontró información verificable sobre este aspecto en las fuentes consultadas: el artículo de TerraMind **no incluye una sección explícita de «Limitaciones» ni de «Consideraciones éticas»**; las limitaciones anteriores se han recopilado de los resultados, discusiones y advertencias dispersas por el artículo, las fichas y el repositorio.

---

## 7.11. Consideraciones éticas y riesgos

Se distingue entre **riesgos documentados o directamente derivados de datos publicados** y **riesgos potenciales que requieren evaluación propia**.

### Riesgos con respaldo en datos publicados

**[Publicado/Documentado]**

- **Sesgo geográfico en los datos de entrenamiento.** TerraMesh aplica un muestreo deliberado: las regiones homogéneas como el Sahara o la tundra están **submuestreadas por diseño** y las zonas urbanas están sobrerrepresentadas (4 % de superficie urbana frente a menos del 1 % del total de tierras emergidas), por el carácter urbano-centrado de SSL4EO-S12 (artículo de TerraMesh, §3.3). Esto implica que el modelo ve menos ejemplos de ciertos paisajes y más de otros.
- **Sesgo temporal.** La mayoría de las adquisiciones se concentran entre 2019 y 2023; el 1 % de las muestras presenta diferencias temporales superiores al año entre modalidades (ficha de TerraMesh). El propio artículo de TerraMesh documenta que intentos de usar pseudoetiquetas de otros modelos (DynamicWorld, ETH-GCHM) fallaron por limitada cobertura temporal en los datos de entrenamiento de esos modelos, con predicciones falsas (nieve en desiertos, alturas negativas), lo que ilustra el riesgo de extrapolar entre épocas.
- **Riesgo de usar generaciones como observaciones.** Las modalidades generadas son invenciones del modelo («imágenes mentales»), no mediciones: tratarlas como observaciones verificadas introduce error sistemático no cuantificado.
- **Brecha cero disparos vs. ajuste fino.** La diferencia de rendimiento publicada (45.40 vs. 82.17 IoU en mapeo de agua) muestra que el uso sin adaptación puede producir resultados muy pobres.

### Riesgos potenciales que requerirían evaluación

**[Interpretación — no documentados por los desarrolladores]**

- **Consecuencias en decisiones territoriales y ambientales.** Emular resultados del modelo sin validación local en procesos de ordenamiento, catastro o gestión ambiental puede generar decisiones equivocadas; no hay evidencia documentada de incidentes concretos atribuibles a TerraMind, y **no se le atribuye ninguno en esta ficha**.
- **Diferencias en disponibilidad de datos entre regiones.** Los países con menor cobertura en los conjuntos base (SSL4EO-S12 se concentra en ciudades globales; MajorTOM-Core cubre casi toda la superficie terrestre con un solo timestamp) pueden recibir representaciones menos ricas.
- **Uso en contextos jurídicos o administrativos.** Ninguna documentación consultada respalda el uso de TerraMind como prueba, certificación o insumo decisivo en trámites con efectos legales.

### Recomendaciones de uso (derivadas de las limitaciones anteriores)

1. Validar cualquier salida con observaciones de campo, datos oficiales o productos alternativos verificados para la zona y época de interés.
2. Reportar siempre la variante, las modalidades de entrada y el procedimiento de adaptación utilizados, para que los resultados sean reproducibles.
3. Mantener supervisión humana experta en toda decisión con impacto territorial, ambiental o administrativa.
4. Verificar la licencia y las condiciones de atribución de cada fuente de datos subyacente antes de reutilizar resultados (ver sección 7.13 y `DATASET_CARD.md`, sección 8.13).

---

## 7.12. Condiciones de uso

| Componente | Licencia / condición | Fuente verificada |
| --- | --- | --- |
| **Código** (repositorio `IBM/terramind`) | **Apache License 2.0** | API de GitHub (`license: Apache-2.0`) consultada el 14 de octubre de 2026 |
| **Pesos del modelo** (fichas `TerraMind-1.0-*` en Hugging Face) | **Apache License 2.0** (`license: apache-2.0` en los metadatos de las cuatro variantes) | API y fichas de Hugging Face |
| **Conjunto de datos de preentrenamiento (TerraMesh)** | **CC BY-SA 4.0** — distinta de la de código y modelo | Ficha de TerraMesh |
| **Datos fuente dentro de TerraMesh** | SSL4EO-S12 v1.1: CC BY 4.0; MajorTOM-Core: CC BY-SA 4.0; LULC ESRI/Impact Observatory/Microsoft: CC BY 4.0; Copernicus WorldDEM-30: © DLR e.V. 2010-2014 y © Airbus Defence and Space GmbH 2014-2018, facilitado por Copernicus/UE/ESA, todos los derechos reservados | Ficha de TerraMesh |

**Acceso a los pesos.** **[Documentado]** Pesos públicos y descargables desde Hugging Face (la ficha oficial declara `apache-2.0`); el artículo indica que el conjunto de datos de preentrenamiento, los pesos del modelo y el código se publican bajo licencia permisiva. La descarga de pesos se realiza automáticamente mediante TerraTorch (`pretrained=True`).

**Requisitos técnicos publicados** ([repositorio oficial](https://github.com/IBM/terramind)):

- Python **3.11 o superior**.
- `terratorch>=1.2.5` (herramienta de ajuste fino).
- Para los ejemplos con *notebooks*: `jupyter`, `tensorboard`, `setuptools<81`.
- Para las generaciones de TerraMind: `diffusers==0.30.0`.
- Sistema operativo: se documenta una advertencia específica para usuarios de macOS (MPS) relacionada con el decodificador UNet.

**Dependencias documentadas de la ficha del modelo:** TerraTorch como paquete de integración y *fine-tuning*; la ficha está etiquetada con `library_name: terratorch` y el registro `BACKBONE_REGISTRY` / `FULL_MODEL_REGISTRY`.

**[Pendiente]** No se encontró información verificable sobre este aspecto en las fuentes consultadas: requisitos mínimos de GPU/CPU/RAM declarados para inferencia, ni requisitos de memoria para cada variante.

---

## 7.13. Información de contacto y mantenimiento

**[Documentado]**

- **Repositorio oficial de código y canales de incidencias:** <https://github.com/IBM/terramind> (opción *issues*). Última actualización registrada por la API de GitHub: 13 de octubre de 2026.
- **Fichas de modelo y discusiones:** <https://huggingface.co/ibm-esa-geospatial/TerraMind-1.0-base> (y variantes `tiny`, `small`, `large` en la misma organización).
- **Documentación técnica de uso:** <https://torchgeo.org/terratorch/stable/guide/terramind/> (guía de TerraTorch).
- **Sitio web del proyecto:** <https://ibm.github.io/terramind/>.
- **Publicaciones institucionales:** blog de IBM Research (<https://research.ibm.com/blog/terramind-esa-earth-observation-model>) y nota de la ESA (<https://www.esa.int/Applications/Observing_the_Earth/ESA_and_IBM_collaborate_on_TerraMind>).
- **Publicación revisada:** ICCV 2025 (<https://openaccess.thecvf.com/content/ICCV2025/html/Jakubik_TerraMind_Large-Scale_Generative_Multimodality_for_Earth_Observation_ICCV_2025_paper.html>).
- **Proyecto financiador:** FAST-EO (*Fostering Advancements in Foundation Models via Unsupervised and Self-supervised Learning for Downstream Tasks in Earth Observation*), financiado por la ESA Φ-lab, contrato n.º 4000143501/23/I-DT.

**[Pendiente]** No se encontró información verificable sobre este aspecto en las fuentes consultadas: canal de soporte formal con compromisos de tiempo de respuesta, política de versiones o hoja de ruta pública del modelo.

---

## 7.14. Autor de la ficha

- **Nombre:** Washington Noriel Yela Portocarrero.
- **Programa académico:** Ingeniería Catastral y Geodesia.
- **Institución:** Universidad Distrital Francisco José de Caldas.
- **Asignatura:** Inteligencia Artificial para el Territorio.
- **Fecha:** 14 de octubre de 2026.

**Aclaración:** se trata del autor de **esta documentación académica**, no del desarrollador original de TerraMind. El modelo fue desarrollado por IBM, la ESA Φ-lab, el Forschungszentrum Jülich y el proyecto FAST-EO; los resultados aquí recopilados pertenecen a sus autores originales.

---

## 7.15. Referencias

Las referencias completas de todos los documentos consultados para elaborar esta ficha se encuentran en [`REFERENCES.md`](./REFERENCES.md). Las principales son:

1. Jakubik, J., Yang, F., Blumenstiel, B., Scheurer, E., Sedona, R., Maurogiovanni, S., … Longépé, N. (2025). *TerraMind: Large-Scale Generative Multimodality for Earth Observation*. ICCV 2025. <https://arxiv.org/abs/2504.11171>
2. Blumenstiel, B., Fraccaro, P., Marsocci, V., Jakubik, J., Maurogiovanni, S., Czerkawski, M., … Longépé, N. (2025). *TerraMesh: A Planetary Mosaic of Multimodal Earth Observation Data*. CVPR Workshops 2025. <https://arxiv.org/abs/2504.11172>
3. Mitchell, M., Wu, S., Zaldivar, A., Barnes, P., Vasserman, L., Hutchinson, B., Spitzer, E., Raji, I. D., & Gebru, T. (2019). *Model Cards for Model Reporting*. FAT* '19, 220–229. <https://doi.org/10.1145/3287560.3287596>
4. Marsocci, V., Jia, Y., Bellier, G. L., Kerekes, D., Zeng, L., Hafner, S., … Shibli, A. (2024). *PANGAEA: A Global and Inclusive Benchmark for Geospatial Foundation Models*. <https://arxiv.org/abs/2412.04204>
5. Repositorio oficial de TerraMind: <https://github.com/IBM/terramind>
6. Ficha oficial de TerraMind 1.0 base: <https://huggingface.co/ibm-esa-geospatial/TerraMind-1.0-base>
7. Ficha oficial de TerraMesh: <https://huggingface.co/datasets/ibm-esa-geospatial/TerraMesh>
8. Plantillas de Hugging Face: <https://github.com/huggingface/huggingface_hub/tree/main/src/huggingface_hub/templates>
