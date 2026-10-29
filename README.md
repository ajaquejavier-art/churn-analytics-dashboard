# Churn Analytics & User Retention Dashboard

Un panel ejecutivo interactivo de 3 páginas desarrollado en **Looker Studio** y respaldado por un modelo dimensional de datos en **Google BigQuery**. Este proyecto analiza la retención de clientes, el impacto financiero del churn en el ingreso recurrente mensual (MRR), la distribución demográfica y los patrones de actividad/engagement de los usuarios.

---

## 🛠️ Tech Stack & Arquitectura

- **Data Warehouse:** Google BigQuery (SQL ANSI).
- **Business Intelligence:** Google Looker Studio.
- **Modelado de Datos:** Esquema en Estrella (*Star Schema*) compuesto por 1 tabla de dimensión y 2 tablas de hechos.

---

## 🗄️ Modelo de Datos (Star Schema)

El modelo está optimizado para análisis de alto rendimiento y filtrado cruzado dinámico:

### 1. `dim_usuarios` (Dimensión)
Atributos demográficos y registro de usuarios.
- `user_id` *(STRING, PK)*: Identificador único del usuario.
- `pais` *(STRING)*: País de residencia.
- `fecha_registro` *(DATE)*: Fecha de alta en la plataforma.

### 2. `fact_suscripciones` (Hechos)
Historial transaccional, planes contratados y estados de cancelación.
- `subscription_id` *(STRING, PK)*: Identificador único de suscripción.
- `user_id` *(STRING, FK)*: Identificador del usuario.
- `tipo_plan` *(STRING)*: Nivel de suscripción (*Básico, Pro, Premium*).
- `monto_mensual` *(NUMERIC)*: Ingreso mensual recurrente (MRR).
- `estado_actual` *(STRING)*: Estado (*Activo, Cancelado*).
- `fecha_inicio` *(DATE)*: Fecha de inicio de contrato.
- `fecha_cancelacion` *(DATE)*: Fecha de baja de la suscripción.

### 3. `fact_actividad` (Hechos)
Registro continuo de interacción y engagement.
- `event_id` *(STRING, PK)*: Identificador de sesión/evento.
- `user_id` *(STRING, FK)*: Identificador del usuario.
- `fecha_evento` *(DATE)*: Fecha de la actividad.
- `tiempo_sesion_minutos` *(NUMERIC)*: Duración promedio por sesión.
- `dias_activo_ultimo_mes` *(INTEGER)*: Días activos en los últimos 30 días.

---

## 📊 Estructura del Dashboard

### Página 1: Suscripciones y MRR
- **KPIs Clave:** MRR Total, Tasa de Churn Mensual (%), Cancelaciones Totales y Ticket Promedio por Plan.
- **Visualizaciones:** Evolución temporal de MRR vs. Churn, distribución de ingresos por plan y tendencias de cancelación.

### Página 2: Usuarios y Demografía
- **KPIs Clave:** Base Total de Usuarios y Nuevos Usuarios Adquiridos.
- **Visualizaciones:** Distribución geográfica escalonada por país, curva de adquisición histórica y tabla de detalle de clientes.

### Página 3: Actividad y Engagement
- **KPIs Clave:** Promedio de Días Activos, Tiempo Promedio de Sesión y Usuarios Inactivos.
- **Visualizaciones:** Tendencia de actividad temporal y clasificación por niveles de engagement mediante campo calculado (`CASE`):

<pre><code>CASE 
  WHEN dias_activo_ultimo_mes >= 20 THEN 'Alta (20-30 días)'
  WHEN dias_activo_ultimo_mes >= 8 THEN 'Media (8-19 días)'
  WHEN dias_activo_ultimo_mes >= 1 THEN 'Baja (1-7 días)'
  ELSE 'Sin actividad (0 días)'
END</code></pre>

---

## ⚙️ Transformación y Limpieza SQL Destacada

Para balancear la distribución geográfica y simular variaciones realistas en la visualización, se ejecutó la siguiente consulta en BigQuery excluyendo columnas duplicadas:

<pre><code>CREATE OR REPLACE TABLE `churn_analytics.dim_usuarios` AS
SELECT 
  * EXCEPT(pais),
  CASE MOD(CAST(SUBSTR(user_id, 5) AS INT64), 10)
    WHEN 0 THEN 'Argentina'
    WHEN 1 THEN 'Argentina'
    WHEN 2 THEN 'Argentina'
    WHEN 3 THEN 'México'
    WHEN 4 THEN 'México'
    WHEN 5 THEN 'Colombia'
    WHEN 6 THEN 'Chile'
    WHEN 7 THEN 'España'
    WHEN 8 THEN 'Perú'
    ELSE 'Uruguay'
  END AS pais
FROM `churn_analytics.dim_usuarios`;</code></pre>

---

## 💡 Insights de Negocio Clave

1. **Riesgo de Churn Temprano:** Los usuarios en la categoría de 'Baja actividad' (< 8 días activos/mes) tienen una alta probabilidad de cancelar en los siguientes 60 días.
2. **Retención por Plan:** El segmento *Premium* presenta la tasa de retención más sólida, compensando la volatilidad del plan *Básico*.
3. **Concentración Geográfica:** Argentina y México representan los principales focos de volumen de usuarios y demanda.
