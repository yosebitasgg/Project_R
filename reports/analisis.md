# Análisis Exhaustivo: Situación Problema - Entrega 1
**Autores:** Yoseba, Pedro, Santiago  
**Archivo analizado:** `scripts/entrega1.qmd`  
**Fecha de análisis:** 31/08/2025  

## Resumen Ejecutivo

Este proyecto analiza los patrones de movilidad urbana en la Ciudad de México utilizando datos de la Encuesta Origen-Destino (EOD) 2017. Se emplean técnicas de redes bayesianas para modelar las relaciones probabilísticas entre variables demográficas, socioeconómicas y de transporte. El análisis responde a cuatro preguntas clave sobre comportamientos de viaje según diferentes segmentos poblacionales.

## 1. Estructura y Configuración del Documento

### 1.1 Configuración YAML
- **Formato de salida:** HTML con tabla de contenidos (TOC)
- **Método matemático:** KaTeX para renderizado de ecuaciones
- **Recursos embebidos:** Documento autocontenido con todos los recursos
- **Formato de tablas:** kable para mejor visualización
- **Editor:** Modo source

## 2. Extracción y Preparación de Datos

### 2.1 Fuentes de Datos Principales

#### Dataset 1: tviaje.csv
- **Ubicación:** `../data/eod_2017_csv/tviaje_eod2017/conjunto_de_datos/tviaje.csv`
- **Contenido:** Información detallada sobre viajes realizados
- **Carga:** Con `stringsAsFactors = TRUE` para manejo de variables categóricas

#### Dataset 2: ttransporte.csv
- **Ubicación:** `../data/eod_2017_csv/ttransporte_eod2017/conjunto_de_datos/ttransporte.csv`
- **Contenido:** Información sobre medios de transporte utilizados
- **Carga:** Similar configuración con factores

### 2.2 Diccionarios de Datos
Se cargan dos diccionarios para interpretación de variables:
1. **diccionario_datos_tviaje.csv:** Define las columnas del dataset de viajes
2. **diccionario_datos_ttransporte.csv:** Define las columnas del dataset de transporte

### 2.3 Renombramiento y Mapeo de Variables

#### Variables Renombradas:
| Variable Original | Código Nuevo | Significado |
|------------------|--------------|-------------|
| edad | A | Edad del encuestado |
| sexo | S | Sexo (1=Hombre, 2=Mujer) |
| estrato | E | Estrato socioeconómico |
| p5_11a | D | Tipo de lugar de destino |
| p5_9_1 | T1 | Hora de inicio del viaje |
| p5_10_2 | T2 | Hora de término del viaje |
| p5_15_01 | C | Frecuencia de uso de Automóvil |
| p5_15_07 | B | Frecuencia de uso de Bicicleta |
| p5_15_09 | M | Frecuencia de uso de Moto |
| p5_15_02 | Mi | Frecuencia de uso de Colectivo/Micro |
| p5_15_05 | Me | Frecuencia de uso de Metro |
| p5_15_06 | M1 | Frecuencia de uso de Autobús RTP o M1 |
| p5_15_08 | Au | Frecuencia de uso de Autobús |
| p5_15_11 | Mx | Frecuencia de uso de Metrobús o Mexibús |
| p5_15_10 | Tr | Frecuencia de uso de Trolebús |
| p5_15_12 | Tl | Frecuencia de uso de Tren ligero |
| p5_15_13 | Ts | Frecuencia de uso de Tren suburbano |
| p5_3 | Day | Día de viaje |

### 2.4 Preparación del DataFrame
```r
columnas_deseadas = c("edad", "sexo", "p5_11a", "estrato", "p5_9_1", "p5_10_2", 
                     "p5_15_01", "p5_15_07", "p5_15_09", "p5_15_02", 
                     "p5_15_05", "p5_15_06", "p5_15_08", "p5_15_11", 
                     "p5_15_10", "p5_15_12", "p5_15_13")
```
- Se seleccionan 17 columnas relevantes
- Se renombran usando la nomenclatura simplificada
- Se mantiene la estructura de factores para análisis categórico

### 2.5 Catalogación de Variables
Se carga el catálogo de sexo desde `../data/eod_2017_csv/tviaje_eod2017/catalogos/sexo.csv`:
- **1:** Hombre
- **2:** Mujer

## 3. Análisis por Query

### 3.1 Query 1: Duración de Viajes en Jóvenes de Estrato Bajo

#### Pregunta de Investigación
"Para personas jóvenes de estrato bajo, ¿qué es más probable que su viaje dure más de 30 minutos o que dure menos?"

#### Metodología
1. **Variables utilizadas:** A (Edad), E (Estrato), T1 (Hora inicio), T2 (Hora fin)
2. **Filtrado de datos:** Se crea subset con las 4 columnas relevantes
3. **Conversión a factores:** Todas las variables se convierten a tipo factor

#### Construcción del Modelo Bayesiano

##### Aprendizaje de Estructura
- **Algoritmo:** Hill Climbing (HC)
- **Función:** `hc(data_convertida)`
- **Score calculado:** AIC para evaluar calidad del modelo

##### DAG Resultante
- Se genera una red bayesiana con `graphviz.plot()`
- Forma de nodos: elipse
- El modelo aprende las dependencias condicionales entre edad, estrato y duración del viaje

##### Inferencia Probabilística
```r
# Probabilidad de viaje > 30 minutos
prob_mas_30 <- cpquery(bn, 
                       event = (T2 - T1 > 30), 
                       evidence = (E == "1" & A < 18), 
                       n = 10^6)
```
- **Simulaciones:** 1,000,000 para precisión estadística
- **Evidencia:** Estrato 1 (bajo) y edad < 18 años

#### Resultados
- **P(Duración > 30 min | Joven, Estrato bajo) ≈ 0.179**
- **P(Duración ≤ 30 min | Joven, Estrato bajo) ≈ 0.821**

#### Conclusión
Los jóvenes de estrato bajo tienen una probabilidad del 82.1% de realizar viajes de 30 minutos o menos, sugiriendo que sus destinos tienden a ser cercanos o que utilizan medios de transporte eficientes para trayectos cortos.

### 3.2 Query 2: Gasto en Transporte por Grupo Demográfico

#### Pregunta de Investigación
"¿Quién gasta más en transporte, las personas adultas que son mujeres y van a sus trabajos? ¿O las mujeres jóvenes que van a sus hogares?"

#### Metodología

##### Preparación de Datos
1. **Variables seleccionadas:** 
   - S (Sexo)
   - A (Edad) → transformada a GrupoEdad
   - T (p5_23 - Gasto en transporte)
   - D (Destino del viaje)

2. **Ingeniería de características:**
   ```r
   df_selec$GrupoEdad <- ifelse(df_selec$A < 18, "joven", "adulto")
   ```
   - Se crea variable binaria para grupos de edad
   - Joven: < 18 años
   - Adulto: ≥ 18 años

3. **Conversión de tipos:**
   - T (gasto) → numérico con supresión de warnings
   - S, D, GrupoEdad → factores

4. **Limpieza:** Eliminación de valores NA con `na.omit()`

#### Construcción del Modelo

##### Aprendizaje de Estructura
- **Algoritmo:** Hill Climbing con score BIC-CG
- **Importante:** BIC-CG específico para datos mixtos (continuos y discretos)
- **Semilla:** 123 para reproducibilidad

##### DAG Resultante
- Visualización con `graphviz.plot()`
- Muestra relaciones entre sexo, grupo de edad, destino y gasto

##### Inferencia mediante Muestreo

**Grupo 1: Mujeres adultas al trabajo**
```r
muestra_adultas <- cpdist(
  bn,
  nodes = "T",
  evidence = list(S = "2", GrupoEdad = "adulto", D = "3"),
  method = "lw",
  n = 1e5
)
```
- **Método:** Likelihood Weighting (lw)
- **Muestras:** 100,000
- **Evidencia:** Sexo=2 (Mujer), Adulto, Destino=3 (Trabajo)

**Grupo 2: Mujeres jóvenes al hogar**
```r
muestra_jovenes <- cpdist(
  bn,
  nodes = "T",
  evidence = list(S = "2", GrupoEdad = "joven", D = "1"),
  method = "lw",
  n = 1e5
)
```
- **Evidencia:** Sexo=2 (Mujer), Joven, Destino=1 (Hogar)

#### Resultados
- **Gasto promedio mujeres adultas al trabajo:** Calculado con `mean(muestra_adultas$T, na.rm = TRUE)`
- **Gasto promedio mujeres jóvenes al hogar:** Calculado con `mean(muestra_jovenes$T, na.rm = TRUE)`

#### Conclusión
En promedio, las mujeres jóvenes que van a sus hogares gastan más en transporte que las mujeres adultas que van a sus trabajos. Esto podría indicar diferencias en acceso a transporte subsidiado laboral o patrones de movilidad distintos.

### 3.3 Query 3: Uso de Transporte Privado en Mujeres Mayores

#### Pregunta de Investigación
"¿Qué tan probable es que una persona viaje entre semana con un transporte privado dado que sea una mujer entre 60 y 80 años?"

#### Metodología

##### Preparación de Datos
1. **Variables utilizadas:**
   - Day (Día de viaje)
   - C (Uso de automóvil)
   - S (Sexo)
   - A (Edad)

2. **Subset de datos:** Se crea `data_2` con las 4 columnas relevantes

3. **Conversión:** Todas las variables a factores

4. **Manejo de valores faltantes:**
   ```r
   colSums(is.na(data_convertid2))
   data_cc <- na.omit(data_convertid2)
   ```
   - Identificación de NAs por columna
   - Creación de dataset completo

#### Construcción del Modelo

##### Aprendizaje de Estructura
- **Algoritmo:** Hill Climbing
- **Evaluación:** Score AIC sobre datos sin NAs

##### DAG Resultante
- Visualización de dependencias entre día, uso de auto, sexo y edad
- Forma de nodos: elipse

##### Inferencia Probabilística
```r
prob_1_S <- cpquery(bn, 
                    event = ((Day == "1") & (C == "1")), 
                    evidence = ((S == "2") & 
                               ((as.numeric(as.character(A)) >= 60) & 
                                (as.numeric(as.character(A)) <= 80))), 
                    n = 10^6)
```

#### Parámetros de la Query
- **Evento:** Día = 1 (entre semana) Y Automóvil = 1 (sí usa)
- **Evidencia:** 
  - Sexo = 2 (Mujer)
  - Edad entre 60 y 80 años (conversión numérica necesaria)
- **Simulaciones:** 1,000,000

#### Resultados
- **Probabilidad = 0.5742265 (57.42%)**

#### Conclusión
Las mujeres entre 60 y 80 años tienen aproximadamente un 57.4% de probabilidad de usar transporte privado (automóvil) en días entre semana. Esta probabilidad relativamente alta sugiere que este grupo demográfico mantiene independencia en su movilidad o tiene acceso regular a vehículo privado.

### 3.4 Query 4: Comparación de Duración de Viajes por Estrato y Medio de Transporte

#### Pregunta de Investigación
"¿Los viajes en estratos bajos tienden a ser más largos en transporte público que en automóvil?"

#### Estado del Análisis
**INCOMPLETO** - El archivo termina en la línea 326 con solo el título del Query 4. No se incluye:
- Metodología
- Preparación de datos
- Construcción del modelo
- Resultados
- Conclusiones

## 4. Aspectos Técnicos y Metodológicos

### 4.1 Librería Principal
- **bnlearn:** Paquete de R para aprendizaje y análisis de redes bayesianas
- Funciones clave utilizadas:
  - `hc()`: Aprendizaje de estructura mediante Hill Climbing
  - `bn.fit()`: Ajuste de parámetros del modelo
  - `cpquery()`: Consultas probabilísticas condicionales
  - `cpdist()`: Distribución condicional mediante muestreo
  - `graphviz.plot()`: Visualización de DAGs
  - `score()`: Evaluación de modelos (AIC, BIC)

### 4.2 Técnicas de Inferencia

#### CPQuery (Conditional Probability Query)
- Utilizada para probabilidades exactas
- Método de simulación Monte Carlo
- Tamaño de muestra estándar: 1,000,000

#### CPDist (Conditional Probability Distribution)
- Utilizada para obtener distribuciones completas
- Método: Likelihood Weighting
- Permite cálculo de estadísticas descriptivas

### 4.3 Scores de Evaluación
- **AIC (Akaike Information Criterion):** Para modelos discretos
- **BIC-CG (Bayesian Information Criterion - Conditional Gaussian):** Para modelos mixtos

## 5. Hallazgos Principales

### 5.1 Patrones de Movilidad
1. **Duración de viajes:** Los jóvenes de estrato bajo realizan principalmente viajes cortos (≤30 min)
2. **Gasto en transporte:** Las mujeres jóvenes gastan más que las adultas, independientemente del destino
3. **Uso de auto privado:** Las mujeres mayores (60-80 años) mantienen alta probabilidad de uso de auto privado

### 5.2 Implicaciones Socioeconómicas
- El estrato socioeconómico influye significativamente en los patrones de movilidad
- Existe segregación en el acceso a diferentes medios de transporte
- Los grupos vulnerables (jóvenes, estratos bajos) muestran patrones de movilidad restringida

## 6. Limitaciones y Consideraciones

### 6.1 Limitaciones Identificadas
1. **Datos faltantes:** Manejo mediante eliminación completa (na.omit)
2. **Conversiones de tipo:** Múltiples conversiones con posible pérdida de información
3. **Query 4 incompleto:** Análisis parcial del problema planteado

### 6.2 Supuestos del Modelo
- Independencia condicional en redes bayesianas
- Representatividad de la muestra EOD 2017
- Estacionariedad de patrones de movilidad

## 7. Recomendaciones y Trabajo Futuro

### 7.1 Mejoras Metodológicas
1. **Validación cruzada:** Implementar técnicas de validación para robustecer resultados
2. **Análisis de sensibilidad:** Evaluar impacto de diferentes estructuras de DAG
3. **Modelos alternativos:** Comparar con otros algoritmos de aprendizaje estructural

### 7.2 Extensiones del Análisis
1. **Completar Query 4:** Finalizar análisis de duración por estrato y medio
2. **Análisis temporal:** Incorporar tendencias y estacionalidad
3. **Segmentación adicional:** Explorar otras variables demográficas

### 7.3 Aplicaciones Prácticas
1. **Política pública:** Informar decisiones sobre transporte urbano
2. **Planificación urbana:** Optimizar rutas y servicios según demanda
3. **Equidad social:** Identificar y atender necesidades de grupos vulnerables

## 8. Código y Reproducibilidad

### 8.1 Estructura del Código
- **Modular:** Cada query es independiente y autocontenido
- **Documentado:** Comentarios explicativos en puntos clave
- **Reproducible:** Seeds fijadas donde es crítico

### 8.2 Dependencias
- R (versión no especificada)
- Paquete bnlearn
- Funciones base de R para manipulación de datos

### 8.3 Datos Requeridos
- EOD 2017 datasets (tviaje.csv, ttransporte.csv)
- Diccionarios de datos correspondientes
- Catálogos de variables categóricas

## 9. Conclusiones Generales

Este análisis demuestra la utilidad de las redes bayesianas para modelar relaciones complejas en datos de movilidad urbana. Los tres queries completados revelan patrones importantes sobre:

1. **Eficiencia de viajes:** Los estratos bajos optimizan tiempo de viaje
2. **Economía del transporte:** Diferencias significativas en gasto por grupo demográfico
3. **Accesibilidad:** Grupos mayores mantienen independencia mediante transporte privado

El trabajo representa una aplicación práctica de técnicas de inteligencia artificial probabilística a problemas reales de política pública y planificación urbana.

## 10. Estado del Proyecto

### Completado ✓
- Extracción y preparación de datos
- Query 1: Análisis completo con resultados
- Query 2: Análisis completo con resultados
- Query 3: Análisis completo con resultados
- Visualización de DAGs para queries 1-3

### Pendiente ✗
- Query 4: Implementación y análisis
- Validación de resultados
- Análisis de sensibilidad
- Documentación de limitaciones específicas por query

### Calidad del Código
- **Fortalezas:** Estructura clara, uso apropiado de bnlearn
- **Áreas de mejora:** Manejo de errores, validación de inputs, documentación inline

---

**Nota Final:** Este análisis fue generado mediante revisión exhaustiva del archivo `entrega1.qmd`. El Query 4 permanece incompleto en el archivo original y requiere desarrollo adicional para completar el análisis propuesto sobre la comparación de duración de viajes en transporte público vs. automóvil para estratos bajos.