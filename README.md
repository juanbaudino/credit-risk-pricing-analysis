# 📊 Credit Risk & Loan Pricing Analysis (LendingClub)

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python)
![Econometrics](https://img.shields.io/badge/Methodology-OLS%20Regressions-green)
![Libraries](https://img.shields.io/badge/Stack-Pandas%20%7C%20Statsmodels%20%7C%20Seaborn-orange)

<p align="justify">
Análisis econométrico y empírico de la política de <strong>Risk-Based Pricing</strong> en préstamos personales utilizando microdatos de LendingClub.
</p>

---

## Executive Memo

### Determinantes del Pricing Crediticio y Modelado Econométrico en Préstamos Personales

**Autor:** Juan Ignacio Baudino

**Herramientas:** Python (pandas, numpy, seaborn, statsmodels)

**Metodología:** Análisis Exploratorio de Datos (EDA) y Mínimos Cuadrados Ordinarios (OLS)

---

## 1. Contexto y Pregunta de Negocio

<p align="justify">
El objetivo de este proyecto fue analizar empíricamente la política de fijación de precios basada en el riesgo (Risk-Based Pricing) en el mercado de crédito al consumo, utilizando microdatos de préstamos personales de la plataforma LendingClub.
</p>

<p align="justify">
Específicamente, se buscó responder:
</p>

> **¿Qué factores determinan la Tasa Nominal Anual (APR) cobrada a los prestatarios y en qué medida las variables de solvencia, condiciones contractuales y la nota crediticia interna explican la dispersión del costo del financiamiento?**

---

## 2. Datos y Preparación Técnica

<p align="justify">
Se trabajó con un portfolio de 10.000 solicitudes de crédito otorgadas con 55 variables financieras. Para garantizar robustez metodológica y eficiencia computacional, se implementó el siguiente pipeline de datos:
</p>

### Tratamiento de Valores Faltantes (Missing Values)

**`debt_to_income` (DTI):**

<p align="justify">
Se identificaron 24 observaciones nulas (0,24% de la muestra). Al tratarse de una fracción marginal y estadísticamente despreciable, se optó por su eliminación directa (listwise deletion), descartando riesgos de sesgo muestral.
</p>

**`emp_length` (antigüedad laboral):**

<p align="justify">
Presentaba 817 registros vacíos (~8,2% del portfolio). Borrar estas filas habría introducido un sesgo de selección sistemático, ya que los datos faltantes suelen concentrar a trabajadores independientes, freelancers o nuevos empleados con perfiles de riesgo específicos. Por tanto, se conservaron en la muestra general.
</p>

### Ingeniería de Variables y Corrección de Asimetría (Feature Engineering)

<p align="justify">
La variable de ingreso anual (annual_income) presentaba una severa asimetría hacia la derecha (mediana de $65.000 vs. un máximo extremo de $2,3 millones).
</p>

<p align="justify">
Se aplicó una transformación logarítmica (
ln(Ingreso)
), lo que permitió comprimir la escala, estabilizar la varianza (reduciendo el riesgo de heterocedasticidad) y evitar que valores atípicos (outliers) ejercieran un apalancamiento artificial sobre la pendiente de regresión.
</p>

---

## 3. Hallazgos Cuantitativos y Resultados del Modelo

<p align="justify">
A través de sucesivas iteraciones de regresión lineal por MCO (OLS), se cuantificaron los siguientes efectos marginales bajo la condición de <em>Ceteris Paribus</em>:
</p>

### La Supremacía del Scoring Crediticio

<p align="justify">
La calificación asignada (grade) es el principal determinante del costo del crédito. Manteniendo constantes el ingreso, la deuda y las condiciones del contrato, un prestatario de Grado B paga 3,72 puntos porcentuales (372 bps) más que uno de Grado A.
</p>

<p align="justify">
La penalización para los prestatarios de mayor riesgo (Grado G) escala a +23,88 puntos porcentuales (+2.388 bps) respecto al Grado A, evidenciando una curva de prima de riesgo altamente convexa.
</p>

### Prima por Plazo y Liquidez

<p align="justify">
El plazo de financiamiento tiene un recargo sustancial: extender el préstamo de 36 a 60 meses (+24 meses) incrementa la tasa de interés en 4,10 puntos porcentuales (410 bps), reflejando el costo de oportunidad del capital y la incertidumbre temporal asociada al largo plazo.
</p>

### Evolución de la Capacidad Explicativa del Modelo ($R^2$)

| Modelo                                        | Variables                         |     $R^2$ |
| --------------------------------------------- | --------------------------------- | --------: |
| **Modelo Simple**                             | DTI                               |  **2,0%** |
| **Modelo Multivariado de Fundamentales**      | DTI + ln(Ingreso) + Monto + Plazo | **16,1%** |
| **Modelo Completo con Variables Categóricas** | Dummies de grade y homeownership  | **95,0%** |

<p align="justify">
<strong>Modelo Simple (DTI):</strong> Explicaba únicamente el 2,0% de la variabilidad de la tasa, demostrando que ninguna variable continua aislada define el precio del crédito.
</p>

<p align="justify">
<strong>Modelo Multivariado de Fundamentales (DTI + ln(Ingreso) + Monto + Plazo):</strong> Elevó la capacidad explicativa al 16,1%, revelando que las variables financieras continuas operan en conjunto pero dejan fuera la mayor parte del criterio del negocio.
</p>

<p align="justify">
<strong>Modelo Completo con Variables Categóricas (Dummies de grade y homeownership):</strong> El $R^2$ saltó al 95,0%.
</p>

> **Conclusión de gobernanza y negocio:** Un $R^2$ del 95% prueba empíricamente que la entidad opera bajo un esquema de pricing algorítmico y estrictamente reglado. La tasa de interés no se negocia de forma discrecional por oficiales de crédito, sino que responde a matrices algorítmicas estandarizadas derivadas del score interno.

---

## 4. Limitaciones Metodológicas y Recomendaciones de Negocio

### Supuesto de Linealidad en el Riesgo de Deuda

<p align="justify">
El modelo MCO asume un efecto marginal constante para el DTI (+3 puntos básicos de tasa por cada punto de DTI). Económicamente, esta relación dista de ser lineal: un salto de DTI del 10% al 20% representa un incremento marginal de riesgo bajo, mientras que pasar del 50% al 60% compromete severamente la liquidez del deudor.
</p>

**Recomendación:** Implementar segmentaciones por tramos (splines o buckets de riesgo) o modelos no lineales para tarificar el sobreendeudamiento de forma progresiva.

### Sesgo de Supervivencia y Selección Truncada (Heckman Bias)

<p align="justify">
El análisis se basó exclusivamente en créditos aprobados y originados, omitiendo las solicitudes rechazadas.
</p>

<p align="justify">
En la realidad de la originación crediticia, los perfiles con combinaciones extremas de riesgo (por ejemplo, Grado G con DTI > 50%) son rechazados antes de recibir una oferta de tasa. Por ende, las estimaciones observadas reflejan una muestra truncada. Integrar los datos de rechazos permitiría calibrar un modelo en dos etapas (como un Probit + OLS de Heckman) para capturar el verdadero riesgo subyacente de la demanda de crédito.
</p>

