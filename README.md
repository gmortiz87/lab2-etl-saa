# Laboratorio No. 2: ETL y Predicción de Conversión SaaS

Este proyecto implementa un pipeline de **Extracción, Transformación y Carga (ETL)** y un modelo de **Regresión Logística** para predecir la conversión de usuarios de un periodo de prueba a planes de pago en una plataforma SaaS.

---
## Estructura del Directorio de Proyecto
```text
├── data/
│   ├── raw/             # Archivos CSV originales (fuente)
│   └── processed/       # Dataset final limpio y transformado
├── notebooks/           # Jupyter Notebooks
├── .venv/               # Entorno virtual 
└── README.md            # Documentación
```


## 1. Descripción del Problema
Una plataforma de servicios en línea busca identificar, basándose en la actividad durante el periodo de prueba, qué usuarios tienen la mayor probabilidad de contratar un plan de pago. El objetivo es entrenar un modelo estandarizado y reproducible que permita al equipo comercial enfocar sus esfuerzos de venta de manera científica y eficiente.

## 2. Decisiones de ETL (Limpieza y Transformación)
Se abordaron rigurosamente los problemas de calidad en los 2,545 registros fuente con las siguientes decisiones:
* **Valores Nulos:** Se aplicó imputación por la Mediana para las variables continuas (`age`, `avg_session_minutes`, `satisfaction_score`, `monthly_income_usd`) para evitar el sesgo de outliers extremos, y por Moda para variables categóricas.
* **Duplicados:** Se aislaron duplicados en la columna maestra `user_id`. Se conservó la primera ocurrencia y se retiraron los registros en conflicto.
* **Formatos y Normalización:** Se limpiaron caracteres numéricos especiales como `$` y `%` transformándolos a variables escalares (`float`). Las categorías de texto se estandarizaron en minúsculas y se les retiraron los acentos.
* **Fuga de Información (Data Leakage):** Previo al salto al algoritmo, se purgaron características que representaban fraude analítico (info post-compra) como `selected_plan` y `preferred_plan_before_conversion`.

## 3. Feature Engineering (Variables Derivadas Finales)
Al dataset de datos limpios se le integraron nuevas métricas representativas:
1. `sessions_per_active_day`: Frecuencia de acceso (`sessions_count` / `days_active_trial`).
2. `total_usage_minutes`: Estimación total de uso temporal en la plataforma.
3. `trial_engagement_ratio`: Proporción de días activos sobre el gran total de días del trial.
4. `high_commercial_intent`: Booleano (1, 0) para usuarios con método de pago registrado Y vistas a la página del plan.

## 4. Preparación de Modelado Científico
* Se procesó un **One-Hot Encoding** (`pd.get_dummies`) en todas las columnas nominales.
* La masa se particionó algorítmicamente en **`Train (60%)`**, **`Validation (20%)`** y **`Test (20%)`** activando la mitigación `stratify` para mantener el desbalance natural del 23.5% de conversión en todos los sets.
* Las distribuciones numéricas se uniformaron usando un **`StandardScaler`** aplicado y ajustado estrictamente sobre el Train (Entrenamiento) para conservar la pureza ciega de validación.

## 5. Resultados del Modelo (Regresión Logística Final)
El pronóstico matemático del modelo (probado ciegamente sobre el conjunto `Test`) entregó el siguiente rendimiento:
* **Accuracy (Exactitud de clase):** **79.0%** (Clasificación acertada holística).
* **Precision Técnica (Sobre compras reales):** **61.0%** (De cada 10 presuntas alertas comerciales positivas del modelo, ~6 fueron clientes reales confirmados).
* **ROC-AUC Score:** **0.7036** (Confirma que la regresión logró discriminar y ponderar asertivamente a los usuarios versus una predicción al azar).


