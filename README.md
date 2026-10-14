# TerraMind: documentación del modelo y del conjunto de datos TerraMesh.

## Descripción general

TerraMind es un modelo fundacional multimodal (en inglés, *foundation model*) desarrollado por IBM, la Agencia Espacial Europea (ESA Φ-lab) y el proyecto FAST-EO, concebido para la observación de la Tierra (*Earth Observation*, EO). Según sus desarrolladores, es el primer modelo fundacional generativo *any-to-any* (de cualquier modalidad a cualquier modalidad) para observación de la Tierra: se entrena de forma no supervisada con datos de múltiples sensores y puede tanto generar representaciones útiles para tareas aguas abajo (segmentación de cubiertas y uso del suelo, mapeo de cuerpos de agua, tipificación de cultivos) como producir datos artificiales de una modalidad a partir de otra (por ejemplo, generar imagen de radar a partir de imagen óptica). El entrenamiento de TerraMind se realizó sobre **TerraMesh**, un conjunto de datos global, multimodal y listo para análisis (*Analysis-Ready Data*) que fusiona productos de Sentinel-1, Sentinel-2, modelos digitales de elevación, índices de vegetación y capas de cobertura del suelo en muestras espaciotemporalmente alineadas; TerraMesh funciona, por tanto, como la materia prima sobre la que el modelo aprende relaciones entre modalidades geoespaciales.

## Objetivo del repositorio

Este repositorio tiene como propósito documentar, con rigor y trazabilidad de las fuentes, las características, los usos previstos, los datos, la evaluación y las limitaciones del modelo TerraMind y del conjunto de datos TerraMesh. La documentación se organiza en dos fichas independientes —una para el modelo y otra para el conjunto de datos— elaboradas siguiendo los principios de las *Model Cards* de Mitchell et al. (2019) y las plantillas de documentación de Hugging Face, con el fin de describir de forma verificable qué son, para qué sirven, cómo se evaluaron y qué precauciones deben acompañar su uso en aplicaciones de observación de la Tierra.

## Ficha del modelo

[Consultar la ficha del modelo TerraMind](./MODEL_CARD.md)

El documento `MODEL_CARD.md` describe la identificación del modelo, su arquitectura, usos previstos, datos y procedimiento de entrenamiento, resultados de evaluación publicados, limitaciones técnicas, consideraciones éticas y condiciones de uso.

## Ficha del conjunto de datos

[Consultar la ficha del conjunto de datos TerraMesh](./DATASET_CARD.md)

El documento `DATASET_CARD.md` describe la identificación de TerraMesh, su composición y modalidades, cobertura geográfica y temporal, fuentes de información, estructura, características técnicas, procesamiento, etiquetas, usos previstos, limitaciones y licencia.

## Autor

- **Nombre completo:** Washington Noriel Yela Portocarrero.
- **Programa académico:** Ingeniería Catastral y Geodesia.
- **Institución:** Universidad Distrital Francisco José de Caldas.
- **Asignatura:** Inteligencia Artificial para el Territorio.
- **Fecha de elaboración:** 14 de octubre de 2026.

## Estructura del repositorio

| Archivo | Contenido |
| --- | --- |
| `README.md` | Presentación general del repositorio, objetivo, enlaces a las fichas, autoría y fuentes principales. |
| `MODEL_CARD.md` | Ficha del modelo TerraMind, organizada según los apartados de las *Model Cards* de Mitchell et al. (2019). |
| `DATASET_CARD.md` | Ficha del conjunto de datos TerraMesh, organizada según las plantillas de documentación de conjuntos de datos de Hugging Face. |
| `REFERENCES.md` | Referencias bibliográficas y enlaces de todas las fuentes realmente consultadas, en estilo APA 7. |

## Fuentes principales

- Repositorio oficial de TerraMind (código y ejemplos): <https://github.com/IBM/terramind>
- Conjunto de datos TerraMesh en Hugging Face: <https://huggingface.co/datasets/ibm-esa-geospatial/TerraMesh>
- Ficha oficial de TerraMind 1.0 *base* en Hugging Face: <https://huggingface.co/ibm-esa-geospatial/TerraMind-1.0-base>
- Artículo científico de TerraMind (ICCV 2025): <https://arxiv.org/abs/2504.11171>
- Artículo científico de TerraMesh (CVPR Workshops 2025): <https://arxiv.org/abs/2504.11172>
- Artículo de referencia sobre *Model Cards*: <https://arxiv.org/abs/1810.03993>
- Plantillas de Hugging Face utilizadas como guía de estructura: <https://github.com/huggingface/huggingface_hub/tree/main/src/huggingface_hub/templates>

> **Nota:** este repositorio está publicado en <https://github.com/norielyela13-collab/terramind-terraMesh-model-card> (verificado al momento de su publicación el 14 de octubre de 2026).
