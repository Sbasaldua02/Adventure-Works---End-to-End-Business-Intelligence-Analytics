# 🚴‍♂️ Adventure Works - End-to-End Business Intelligence Analytics

## Resumen del Proyecto
Desarrollo de un dashboard corporativo interactivo en Power BI para *Adventure Works*, una empresa global de equipamiento deportivo ficticia. Este proyecto abarca el ciclo de vida completo de los datos: desde la extracción y limpieza profunda (ETL) de archivos `.csv` crudos, pasando por un modelado relacional complejo (Star/Snowflake) y cálculos DAX avanzados, hasta el diseño de una interfaz optimizada con herramientas de Inteligencia Artificial y diseño responsivo para dispositivos móviles.

---

## Demos Interactivas del Tablero
*Hacé clic en los enlaces a continuación para ver las grabaciones de la interacción real con el reporte:*

1. 📊 **[Exac_Dash_Video](https://github.com/user-attachments/assets/54854e68-84dc-4230-99fa-3e704faed425):** Navegación general, uso de slicers temporales y geográficos. Demostración de *Custom Tooltips* al pasar el cursor sobre los gráficos de barras y tendencias, y cross-filtering (edición de interacciones visuales) entre los distintos KPIs.
2. 🔍 **[Product_Dashboard_Video](https://github.com/user-attachments/assets/ebb7b48b-0416-4f70-b7e4-e6160d550f7f):** Viaje de datos (*Drill-through*) desde la matriz del Top 10 de productos hacia una vista de detalle a nivel producto. Interacción con parámetros *What-If* para simular escenarios de rentabilidad, y uso de parámetros de campo (Field Parameters) para alternar dinámicamente las métricas de un gráfico de áreas.
3. 🌍 **[Map_Dashboard_Video](https://github.com/user-attachments/assets/464c9561-5a5a-4da6-b64d-4bedf2b7e0d3)** Navegación mediante botones y *Bookmarks* hacia el reporte geográfico. Exploración interactiva del mapa de burbujas filtrando por continentes y países.
4. 👥 **[Client_Dashboard_Video](https://github.com/user-attachments/assets/d473d860-fd6c-4b97-bdc9-93dcfc7a235b)** Análisis demográfico interactivo, aplicación de filtros visuales a nivel de página y reporte, y funcionamiento de los botones de navegación globales.

---

## Desarrollo Paso a Paso

### Fase 1: ETL y Preparación de Datos (Power Query)
El punto de partida fueron múltiples archivos raw. El proceso de limpieza y perfilamiento garantizó la calidad de los datos antes de cargarlos al modelo:
* **Perfilamiento de datos:** Análisis de distribución, valores nulos y calidad general utilizando las herramientas de vista de columnas. Validación estricta de tipos de datos.
* **Transformaciones y Columnas Calculadas:** Separación de strings por delimitadores, extracción de texto y uso de funciones matemáticas para derivar nuevos valores.
* **Lógica Condicional:** Creación de columnas condicionales en M para categorizar rangos directamente desde la ingesta.
* **Dimensión de Tiempo (Calendar):** Construcción de una tabla de calendario completa generada a partir de las fechas de ventas, extrayendo jerarquías clave: *Start of Month*, *Start of Week*, *Year*, *Quarter*, y nombres de días/meses.

### Fase 2: Arquitectura y Modelado Relacional
Se diseñó una base de datos analítica estructurada para optimizar el rendimiento y asegurar la propagación correcta de los filtros:
* **Star & Snowflake:** Organización de tablas de hechos (*Sales* y *Returns*) rodeadas por dimensiones descriptivas (*Customers*, *Products*, *Territories*).
* **Integridad Relacional:** Identificación precisa de Claves Primarias (Primary Keys) y Foráneas (Foreign Keys).
* **Flujo de Filtros:** Configuración de relaciones de **Uno a Muchos (1:*)** con dirección de filtro único (Single) desde las *Lookup Tables* hacia las *Fact Tables*, manteniendo la consistencia total de los datos al aplicar segmentaciones.
  <img width="1741" height="683" alt="image" src="https://github.com/user-attachments/assets/8eec71fa-1638-42a7-8f9f-1d28dcce3f22" />


### Fase 3: Cálculos Avanzados y Lógica de Negocio (DAX)
El núcleo analítico del proyecto se construyó mediante un extenso diccionario de medidas (DAX). Se utilizaron funciones lógicas, iteradores y funciones de inteligencia de tiempo para resolver requerimientos de negocio:

**1. Columnas Calculadas Analíticas (Dimensiones)**
* Segmentación de clientes por ingresos usando `SWITCH(TRUE())`:
  ```dax
  Income Level = SWITCH(TRUE(), 
      'Customer Lookup'[AnnualIncome] >= 150000, "Very High", 
      'Customer Lookup'[AnnualIncome] >= 100000, "High", 
      'Customer Lookup'[AnnualIncome] >= 50000, "Average", 
      "Low"
  )
  ```
* Identificación de estacionalidad semanal (`IF('Calendar Lookup'[Day of Week] IN {6,7}, "Weekend", "Weekday")`).

**2. Iteradores y Modificadores de Contexto (Medidas Base)**
* Cálculo de ingresos netos fila por fila conectando tablas de hechos y dimensiones con `SUMX` y `RELATED`:
  ```dax
  Total Revenue = SUMX('Sales Data', RELATED('Product Lookup'[ProductPrice]) * 'Sales Data'[OrderQuantity])
  ```
* Aislamiento de contexto usando `HASONEVALUE` para mostrar el *Total Revenue* única y exclusivamente cuando se filtra a un cliente individual (evitando totales confusos a nivel global).

**3. Inteligencia de Tiempo y Ventanas Móviles (Rolling)**
* Comparativas mensuales (`DATEADD`) para establecer objetivos como `Profit Target = [Previous Month Profit] * 1.1`.
* Suavizado de tendencias temporales para análisis de largo plazo:
  ```dax
  90-days Rolling Profit = CALCULATE([Total Profit], 
      DATESINPERIOD('Calendar Lookup'[Date], LASTDATE('Calendar Lookup'[Date]), -90, DAY)
  )
  ```

### Fase 4: Reporte, UI/UX y Analítica Visual
La capa de visualización se construyó priorizando el descubrimiento interactivo (*drill-down*) y la experiencia del usuario final:
* **Jerarquías de Datos:** Implementación de matrices con *Drill-down* y *Drill-up* geográfico (Continente > País) y de productos (Categoría > Subcategoría > Nombre del Producto).
* **Visual Calculations:** Uso de cálculos visuales directamente sobre matrices estructuradas para obtener insights contextuales rápidos sin sobrecargar el modelo DAX.
* **Interactividad y Navegación:** Creación de un menú lateral con formas formateadas, botones con imágenes dinámicas y acciones de *Bookmarks* para transicionar entre páginas (Exec, Map, Product, Client).
* **Diseño Condicional y KPIs:** Tarjetas de tendencia que comparan el cierre de mes vs el mes anterior, aplicando formato condicional (Negro/rojo) al color del número basándose en el porcentaje de crecimiento.
* **Executive Dashboard**:
  <img width="1296" height="730" alt="image" src="https://github.com/user-attachments/assets/90f592f5-4aad-46f2-a062-1d5540c7d917" />
  
* **Map**:
  <img width="1293" height="705" alt="image" src="https://github.com/user-attachments/assets/e34be00e-fdf5-4c75-9c6d-eddbe3ba7c7b" />

* **Product Detail**:
  <img width="1307" height="723" alt="image" src="https://github.com/user-attachments/assets/60582f6c-96e2-4b4f-8e2b-59312eaee507" />
  
* **Customer Detail**:
  <img width="1297" height="731" alt="image" src="https://github.com/user-attachments/assets/4e932aa3-81fb-4c8a-a9d2-c3d512ded394" />
  
* **Tooltips Personalizados:** Creación de páginas de reporte ocultas con dimensiones específicas para actuar como tooltips ricos en datos al pasar el cursor sobre gráficos de barras.
  <img width="522" height="277" alt="image" src="https://github.com/user-attachments/assets/a0517c6d-a8dc-4ec0-ae6b-681c41bdf92d" />

* **Mobile Layout:** Tablero completamente formateado y optimizado para su consumo en dispositivos móviles.
  <img width="461" height="723" alt="image" src="https://github.com/user-attachments/assets/2dc04fab-bdf5-4b71-a013-21d117aecf73" />


### Fase 5: IA, Parámetros y Simulaciones "What-If"
* **Escenarios Dinámicos (What-If):** Implementación de una perilla de incremento porcentual de precios. Al ajustar el parámetro, el motor DAX recalcula dinámicamente el `Adjusted Price`, `Adjusted Revenue` y el `Adjusted Profit`.
* **Field Parameters (Parámetros de Campo):** Integración de un slicer con 5 métricas seleccionables por el usuario para dinamizar un gráfico de áreas de tendencias sin saturar el lienzo.
* **Inteligencia Artificial:** * Entrenamiento del motor **Q&A** para habilitar consultas en lenguaje natural.
  * Análisis automatizado de **Detección de Anomalías** sobre los gráficos de tendencia lineal.
  * Integración de un **Decomposition Tree** (Mapa Jerárquico) para realizar análisis de causa raíz multidimensional.
  [Descomposition_Tree_Video](https://github.com/user-attachments/assets/a69a9a52-dffb-4cde-80b6-7c4245f46753)

---

