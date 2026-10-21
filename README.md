# Morphing facial con modelos generativos

Proyecto de la asignatura de Visión por Computador (CDIA, ETSISI-UPM). Transforma progresivamente un rostro en otro (facial morphing) y compara tres enfoques bajo las mismas condiciones: deformación geométrica clásica, un VAE propio y una DCGAN propia.

> Estado: en planificación. Las secciones de estructura y roadmap describen lo que se va a construir.

## Objetivo

Comparar de forma rigurosa y reproducible qué aporta cada enfoque al problema del morphing facial: calidad visual, conservación de la identidad, continuidad de la transición y coste computacional.

| Enfoque | Descripción |
| --- | --- |
| Morphing clásico | Baseline con landmarks, triangulación de Delaunay y transformaciones afines locales |
| VAE convolucional | Modelo propio entrenado desde cero; interpolación en el espacio latente |
| DCGAN | Modelo propio con los mismos datos y resolución que el VAE; inversión de fotos reales por optimización |
| StyleGAN preentrenada | Extra opcional, tratado aparte (no comparable de forma justa a 64 × 64) |

## Dataset

**CelebA**, con anotaciones de atributos.

- Resolución de trabajo: 64 × 64 RGB para VAE y DCGAN. Las imágenes originales se conservan en alta resolución para el baseline clásico.
- Preprocesamiento común: selección de imágenes válidas, recorte y alineación facial, redimensionado y normalización.
- Particiones de entrenamiento, validación y prueba.
- **Conjunto de evaluación:** 20-30 pares de rostros, iguales para todos los métodos, repartidos entre fáciles (pose y expresión similares) y difíciles (poses muy distintas). Se reporta media y desviación típica.

## Estructura propuesta del repositorio

```
.
├── data/                 # CelebA y pares de evaluación (no se versiona)
├── src/
│   ├── preprocessing/    # recorte, alineación, particiones
│   ├── classic_morph/    # landmarks, Delaunay, warping afín
│   ├── vae/              # modelo, entrenamiento, interpolación
│   ├── dcgan/            # modelo, entrenamiento, inversión
│   └── evaluation/       # métricas y curvas de identidad
├── experiments/          # configuraciones y resultados por experimento
├── outputs/              # GIFs, vídeos, figuras y tablas
└── report/               # memoria
```

## Métodos

### 1. Morphing clásico (baseline)

1. Detección de landmarks y correspondencia entre ambos rostros.
2. Triangulación de Delaunay y deformación con transformaciones afines locales.
3. Mezcla de las imágenes deformadas para generar los fotogramas intermedios.
4. Exportación como GIF o vídeo.

Los landmarks se detectan sobre la imagen original, el morphing se hace a mayor resolución y el resultado se reduce a 64 × 64 para compararlo con el resto de métodos.

### 2. VAE convolucional (núcleo del proyecto)

El encoder representa cada imagen como una distribución latente (media y varianza), el decoder la reconstruye y el entrenamiento combina reconstrucción y divergencia KL:

```
L = L_recon + β · D_KL( q_φ(z|x) ‖ p(z) )
```

Para generar el morphing se codifican dos rostros, se interpola entre sus latentes y se decodifican los puntos intermedios.

Configuración base: dimensión latente 32 y β = 1. En cada experimento se varía un solo factor.

### 3. DCGAN

Entrenada desde cero con CelebA a 64 × 64, vigilando la estabilidad (curvas de pérdida, muestras periódicas, mode collapse). Para usar fotos reales se invierte la imagen por optimización: se ajusta z para minimizar la distancia entre G(z) y la foto. Después se interpola entre los latentes (lineal y SLERP).

Si la inversión no reconstruye bien, se interpola entre dos rostros generados por la propia GAN, separando esos resultados de los de fotos reales.

## Experimentos

| ID | Experimento | Prioridad |
| --- | --- | --- |
| A | Interpolación latente: lineal frente a SLERP | Imprescindible |
| B | Dimensión latente: 16, 32 y 64 | Si hay tiempo |
| C | Regularización: β = 0,5, 1 y 2 | Imprescindible |
| D | Comparación con el morphing clásico (mismos pares) | Imprescindible |
| E | Direcciones semánticas con atributos de CelebA (sonrisa, gafas...) | Muy recomendable |
| F | Pérdida perceptual (LPIPS o VGG) frente al VAE base | Muy recomendable |

Experimento E: la dirección latente de un atributo se calcula como la diferencia entre la media de los latentes con y sin ese atributo; se comprueba si moverse en esa dirección cambia solo ese rasgo.

## Evaluación

Se evalúan por separado la calidad de los fotogramas, la continuidad de la transición y la conservación de la identidad.

| Métrica | Qué mide | Limitación |
| --- | --- | --- |
| MSE y SSIM | Calidad de reconstrucción | Penalizan reconstrucciones plausibles pero distintas del original |
| FID o KID (una de las dos) | Calidad global de las imágenes generadas | No mide la calidad del morphing |
| Curva de identidad | Similitud de cada fotograma con el rostro de origen y con el de destino (embeddings de un modelo de reconocimiento facial) | Esos modelos usan mayor resolución: se reescala y se declara |
| Suavidad temporal | Cambio entre fotogramas consecutivos y artefactos visuales | La continuidad no garantiza realismo |
| Coste computacional | Tiempos de entrenamiento y generación, hardware | Depende del equipo |

Se mantienen constantes los pares, el número de fotogramas y las condiciones de evaluación. Si no es posible una comparación equivalente, se documenta la limitación.

## Roadmap

- [ ] **1. Preparación y baseline:** preprocesar CelebA, fijar los pares de evaluación, implementar el morphing clásico.
- [ ] **2. VAE:** implementar el modelo base, entrenarlo, validar reconstrucciones y generación.
- [ ] **3. Experimentos con el VAE:** A, C, D, E y F (y B si hay tiempo), con métricas y casos de fallo.
- [ ] **4. DCGAN:** entrenarla, invertir fotos reales, interpolar y comparar con los otros métodos.
- [ ] **5. Evaluación final y memoria:** consolidar métricas, tablas y secuencias comparativas, documentar limitaciones.
- [ ] **Extra opcional:** StyleGAN preentrenada, solo si todo lo anterior está completo.

## Prioridades

- **Imprescindible:** morphing clásico, VAE propio, experimentos A, C y D, curva de identidad.
- **Muy recomendable:** direcciones semánticas (E) y pérdida perceptual (F).
- **Si hay tiempo:** DCGAN con inversión por optimización y experimento B.
- **Extra opcional:** StyleGAN preentrenada.
gi
Fuera de alcance: entrenar una GAN de gran escala o un modelo de difusión desde cero.

## Resultado esperado

1. Un sistema de morphing facial clásico.
2. Un VAE convolucional propio con estudio de interpolación latente, hiperparámetros, direcciones semánticas y pérdida perceptual.
3. Una DCGAN propia con morphing por inversión de imágenes reales, si resulta viable.
4. Una evaluación comparativa con métricas y análisis visual.
5. Una memoria con fundamentos matemáticos, arquitectura, entrenamiento, experimentos, resultados y limitaciones.

El criterio de éxito no es la animación más atractiva, sino una comparación rigurosa y reproducible que explique qué aporta cada enfoque y por qué funciona o falla.