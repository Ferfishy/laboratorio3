**Laboratorio de aplicación inteligente adaptativa**  
**Identificación**  
- **Integrantes**: Fernando Figueroa Olivares  
- **Curso y sección**: Ingeniería Web Avanzada | OII436-1  
- **URL del repositorio**: https://github.com/Ferfishy/laboratorio3.git  
- **Fecha**: 09 de noviembre de 2026  
**Objetivo y alcance**  
Este proyecto implementa un recomendador de recursos de aprendizaje basado en contenido que representa los intereses de cada usuario en un espacio vectorial TF-IDF y actualiza automáticamente su perfil al recibir la interacción **me interesa**. El núcleo adaptativo está implementado en Python (`motor.py`) y se ejecuta por consola (`app.py`).  
**Estructura y validación del catálogo y perfiles (Paso 1)**  
- **Campos que intervienen en el cálculo vs. presentación**:  
   
 El archivo `datos/recursos.csv` contiene **12 recursos** (`R01` a `R12`) estructurados en cuatro columnas: `id`, `titulo`, `descripcion` y `etiquetas`.  
  - **Cálculo de similitud**: Únicamente la columna **`etiquetas`** se utiliza en `construir_indice(recursos)` para ajustar el `TfidfVectorizer` y construir la matriz TF-IDF del catálogo (de dimensión 12 \times 17, con los 17 términos del vocabulario: `angular, api, componentes, consultas, datos, formularios, funciones, graficos, http, interfaces, pandas, python, sql, tablas, validacion, variables, web`).  
  - **Presentación e identificación**: Las columnas `id`, `titulo` y `descripcion` sirven para identificar unívocamente cada recurso y presentar los resultados de forma legible al usuario.  
- **Gestión de valores vacíos e identificadores repetidos**:  
  - En el **catálogo (`cargar_catalogo` en `motor.py`)**: Se carga el CSV sin convertir cadenas vacías en `NaN` (`keep_default_na=False`), se verifica la presencia de las 4 columnas obligatorias (`id`, `titulo`, `descripcion`, `etiquetas`) y que el DataFrame no esté vacío. A continuación, se limpian espacios en blanco extremos con `.str.strip()` y se comprueba si existe algún valor vacío (`(recursos[columna] == "").any()`), lanzando un `ValueError` si alguna celda está vacía. Asimismo, se verifica la unicidad de los identificadores mediante `recursos["id"].duplicated().any()`, lanzando un `ValueError("Los identificadores de recursos deben ser únicos.")` si hay IDs repetidos.  
  - En los **perfiles de usuario (`cargar_perfiles` y `crear_perfil` en `motor.py`)**: Los intereses declarados en `datos/perfiles.json` pueden estar vacíos (`u3: ""`) o contener términos desconocidos fuera del vocabulario del catálogo (`u4: "zzxyz123"`). En ambos casos, `vectorizador.transform([intereses])` genera un vector numérico 1D nulo (todos sus 17 componentes iguales a `0.0`), lo que produce puntuaciones de similitud `0.0` para los 12 recursos y devuelve una lista de recomendaciones vacía (`[]`) junto con el mensaje informativo: `"No hay coincidencias positivas. Declare intereses presentes en el catálogo."`  
  - **Estados independientes en memoria**: `crear_estado` y `crear_historial` en `app.py` almacenan en diccionarios separados por `usuario_id` el vector actual y la lista de eventos de cada usuario (`u1`, `u2`, `u3`, `u4`). Durante la sesión, actualizar a `u1` modifica únicamente `estado["u1"]` e `historial["u1"]`, manteniendo intactos los demás perfiles. Al reiniciar la aplicación, los perfiles se restauran desde `datos/perfiles.json`.  
  
---  
**Preparación y reproducibilidad (Paso 0)**  
- **Versión de Python utilizada**: `Python 3.12.3`  
- **Intérprete del entorno virtual**: `.venv/bin/python`  
- **Comando de activación utilizado (Linux)**: `source .venv/bin/activate`  
**Instrucciones paso a paso para crear el entorno, instalar dependencias y ejecutar la aplicación**  
1. **Abrir una terminal en la carpeta raíz del proyecto y crear el entorno virtual**:  
2.   
```  
python3 -m venv .venv  
   
```  
3. *(En Windows puede usar `python -m venv .venv` o `py -m venv .venv`).*  
4. **Activar el entorno virtual**:  
  - **macOS o Linux**:  
  -   
```  
source .venv/bin/activate  
   
```  
  - **Windows PowerShell**:  
  -   
```  
.venv\Scripts\Activate.ps1  
   
```  
  - **Windows CMD**:  
  -   
```  
.venv\Scripts\activate.bat  
   
```  
5. **Instalar las dependencias y comprobar la preparación del material**:  
6.   
```  
python -m pip install -r requirements.txt  
 python -c "import sys; print(sys.executable)"  
 python app.py --comprobar  
   
```  
7. Salida obtenida con `--comprobar`:  
8.   
```  
Comprobación de preparación completada.  
 Recursos: 12 | Usuarios: 4  
 Matriz TF-IDF: 12 filas x 17 términos  
 Vocabulario: angular, api, componentes, consultas, datos, formularios, funciones, graficos, http, interfaces, pandas, python, sql, tablas, validacion, variables, web  
 Se cargaron los datos y el índice sin ejecutar los bloques del estudiante.  
   
```  
9. **Congelar las versiones instaladas (ya registrado en `requirements.txt`)**:  
10.   
```  
python -m pip freeze > requirements.txt  
   
```  
11. **Ejecutar la aplicación**:  
  - **Modo demostración reproducible (ejecuta las pruebas `T1`–`T4` y genera los archivos en `evidencias/`)**:  
  -   
```  
python app.py --demo  
   
```  
  - **Modo interactivo por consola**:  
  -   
```  
python app.py  
   
```  
  
---  
**Funciones completadas en** `motor.py`  
  
|  |  |  |  
| - | - | - |  
| **Función** |  **Entrada principal** |  **Implementación y requisito cumplido** |  
| `crear_perfil` |  `intereses: str`, `vectorizador: TfidfVectorizer` |  Aplica `vectorizador.transform([intereses]).toarray()[0]` para proyectar el texto de intereses en el espacio TF-IDF de 17 términos ya aprendido del catálogo, retornando un array 1D sin volver a ajustar el vocabulario. |  
| `calcular_puntuaciones` |  `perfil: np.ndarray`, `matriz: np.ndarray` |  Calcula `cosine_similarity([perfil], matriz)[0]`, retornando un vector 1D con las 12 puntuaciones en el mismo orden de filas del catálogo (incluso cuando el perfil es el vector nulo). |  
| `recomendar` |  `perfil`, `recursos`, `matriz`, `vectorizador`, `top_k=3` |  Obtiene las puntuaciones con `calcular_puntuaciones`, filtra solo aquellas con `puntuacion > 0`, ordena por `(-puntuacion, id)` (mayor a menor puntuación y desempate por `id` ascendente) y devuelve hasta `top_k` diccionarios generados con `construir_resultado`. |  
| `actualizar_perfil` |  `perfil: np.ndarray`, `vector_recurso: np.ndarray` |  Calcula la combinación lineal `mezcla = 0.4 * perfil + 0.6 * vector_recurso` (`alpha = 0.6`) y retorna `normalize([mezcla])[0]`, conservando norma L2 unitaria en un vector 1D. |  
  
   
  
---  
**Pruebas funcionales y evidencias (**`T1`**–**`T4`**)**  
A continuación se documentan los resultados reales obtenidos al ejecutar `python app.py --demo`, respaldados por los archivos generados en `evidencias/reporte_demo.json` y `evidencias/puntuaciones_antes_despues.csv`.  
  
|  |  |  |  |  |  
| - | - | - | - | - |  
| **Caso** |  **Entrada o acción** |  **Resultado esperado** |  **Resultado observado y evidencia** |  **Estado** |  
| **T1** |  Solicitar recomendaciones iniciales para `u1` (`"python datos pandas"`) y `u2` (`"angular componentes interfaces"`) sin interacciones previas. |  Rankings coherentes con intereses distintos, hasta 3 resultados con puntuación positiva, orden descendente y desempate por `id` ascendente (`R02` primero para `u1`; `R07` primero para `u2`). |  **Para `u1`**: 1.º `R02` (`1.0000`, términos: `datos, pandas, python`), 2.º `R12` (`0.4965`, términos: `datos, python`), 3.º `R03` (`0.4326`, términos: `datos, python`).  **Para `u2`**: 1.º `R07` (`1.0000`, términos: `angular, componentes, interfaces`), 2.º `R08` (`0.5677`, términos: `angular, interfaces`), 3.º `R09` (`0.3087`, términos: `angular`).  *Interpretación*: Cada usuario recibe recomendaciones alineadas exclusivamente con las etiquetas de su perfil inicial dentro del mismo espacio vectorial. |  **Cumple** |  
| **T2** |  `u1` indica **«Me interesa»** sobre `R04` (*API web con Python*, etiquetas: `python web api`) y se recalculan las puntuaciones y el ranking en la misma ejecución. |  Cambia el vector de `u1`; se registra el evento en su historial; `R04` pasa al primer lugar; aumenta la puntuación de recursos no valorados afines (como `R05` y `R09`); se conservan los mismos 12 candidatos. |  El perfil de `u1` incorpora los términos `api` (`0.446269`) y `web` (`0.446269`) y refuerza `python` (`0.637116`). El historial de `u1` registra `[{"numero": 1, "accion": "Me interesa", "recurso": "R04"}]`.  **Nuevo ranking de `u1`**: 1.º `R04` (`0.8752`), 2.º `R02` (`0.6880`), 3.º `R09` (`0.4958`). La puntuación de `R05` sube de `0.0000` a `0.4378` (`+0.4378`) y la de `R09` sube de `0.0000` a `0.4958` (`+0.4958`), manteniendo los 12 candidatos. |  **Cumple** |  
| **T3** |  Consultar el perfil, historial y recomendaciones de `u2` después de la interacción de `u1` con `R04`. |  El vector del perfil, el historial y las puntuaciones de `u2` permanecen exactamente iguales a los obtenidos antes de modificar a `u1`. |  El perfil de `u2` se mantiene idéntico (`{"componentes": 0.657512, "interfaces": 0.564679, "angular": 0.498813}`), su historial sigue vacío (`[]`) y su ranking conserva `R07` (`1.0000`), `R08` (`0.5677`) y `R09` (`0.3087`).  *Interpretación*: Se comprueba el aislamiento de estado entre usuarios en memoria. |  **Cumple** |  
| **T4** |  Solicitar recomendaciones para `u3` (intereses vacíos `""`) y `u4` (término fuera del vocabulario `"zzxyz123"`). |  Ambos producen vectores nulos, `0` puntuaciones positivas, lista de recomendaciones vacía `[]` y mensaje informativo, sin fallos ni atribución de relevancia a recursos con puntuación cero. |  Tanto `u3` como `u4` obtienen `puntuaciones_cero: true` (las 12 puntuaciones son `0.0`), lista `resultados: []` y muestran el mensaje: `"No hay coincidencias positivas. Declare intereses presentes en el catálogo."` |  **Cumple** |  
  
   
  
---  
**Evidencias detalladas y comparación de la adaptación (Paso 8, Paso 10 y Paso 12)**  
***1. Rankings iniciales de*** *`u1`* ***y*** *`u2`* ***(antes de la interacción)***  
- **Ranking inicial de `u1` (`intereses`: `"python datos pandas"`)**:  
  1. **`R02` | Análisis de datos con Pandas** — `similitud = 1.0000` (`1.0000000000000002`)  
    - *Explicación*: `Términos compartidos: datos, pandas, python.`  
  2. **`R12` | Conectar Python con datos SQL** — `similitud = 0.4965` (`0.4965216471076184`)  
    - *Explicación*: `Términos compartidos: datos, python.`  
  3. **`R03` | Gráficos con Python** — `similitud = 0.4326` (`0.4326193652667901`)  
    - *Explicación*: `Términos compartidos: datos, python.`  
- **Ranking inicial de `u2` (`intereses`: `"angular componentes interfaces"`)**:  
  1. **`R07` | Componentes e interfaces en Angular** — `similitud = 1.0000` (`1.0`)  
    - *Explicación*: `Términos compartidos: angular, componentes, interfaces.`  
  2. **`R08` | Formularios e interfaces en Angular** — `similitud = 0.5677` (`0.5676773602861666`)  
    - *Explicación*: `Términos compartidos: angular, interfaces.`  
  3. **`R09` | Consumir una API desde Angular** — `similitud = 0.3087` (`0.3086813231374439`)  
    - *Explicación*: `Términos compartidos: angular.`  
***2. Evolución del perfil, historial y ranking de*** *`u1`* ***tras valorar*** *`R04`*  
- **Interacción registrada en el historial de `u1`**:  
  - Antes: `[]`  
  - Después: `[{"numero": 1, "accion": "Me interesa", "recurso": "R04"}]`  
- **Perfil de `u1` antes de valorar `R04`** (pesos no nulos):  
-   
```  
{  
   "pandas": 0.753247,  
   "datos": 0.465091,  
   "python": 0.465091  
 }  
   
```  
- **Perfil de `u1` después de valorar `R04`** (`mezcla = 0.4 * perfil_antes + 0.6 * vector_R04`, normalizada con L2):  
-   
```  
{  
   "python": 0.637116,  
   "api": 0.446269,  
   "web": 0.446269,  
   "pandas": 0.376477,  
   "datos": 0.232455  
 }  
   
```  
- **Ranking de `u1` después de valorar `R04`**:  
  1. **`R04` | API web con Python** — `similitud = 0.8752` (`0.8751781758698687`)  
    - *Explicación*: `Términos compartidos: api, python, web.`  
  2. **`R02` | Análisis de datos con Pandas** — `similitud = 0.6880` (`0.688010093616038`)  
    - *Explicación*: `Términos compartidos: datos, pandas, python.`  
  3. **`R09` | Consumir una API desde Angular** — `similitud = 0.4958` (`0.49575959368538536`)  
    - *Explicación*: `Términos compartidos: api, web.`  
***3. Tabla comparativa de puntuaciones para los 12 candidatos (****`evidencias/puntuaciones_antes_despues.csv`****)***  
Se conservan exactamente los mismos 12 recursos candidatos antes y después de la interacción de `u1` sobre `R04`:  
  
|  |  |  |  |  |  |  
| - | - | - | - | - | - |  
| **ID** |  **Título** |  **Etiquetas** |  **Puntuación antes** |  **Puntuación después** |  **Cambio (\Delta)** |  
| **R01** |  Fundamentos de Python |  `python variables funciones` |  `0.186096` (`0.1861`) |  `0.254928` (`0.2549`) |  `+0.068832` (`+0.0688`) |  
| **R02** |  Análisis de datos con Pandas |  `python datos pandas` |  `1.000000` (`1.0000`) |  `0.688010` (`0.6880`) |  `-0.311990` (`-0.3120`) |  
| **R03** |  Gráficos con Python |  `python datos graficos` |  `0.432619` (`0.4326`) |  `0.404430` (`0.4044`) |  `-0.028189` (`-0.0282`) |  
| **R04** |  API web con Python *(valorado)* |  `python web api` |  `0.251037` (`0.2510`) |  `0.875178` (`0.8752`) |  `+0.624141` (`+0.6241`) |  
| **R05** |  **Principios de una API HTTP** *(no valorado)* |  `web api http` |  `0.000000` (`0.0000`) |  `0.437777` (`0.4378`) |  **`+0.437777` (`+0.4378`)** |  
| **R06** |  **Validación de datos en una API** *(no valorado)* |  `web api validacion` |  `0.000000` (`0.0000`) |  `0.437777` (`0.4378`) |  **`+0.437777` (`+0.4378`)** |  
| **R07** |  Componentes e interfaces en Angular |  `angular componentes interfaces` |  `0.000000` (`0.0000`) |  `0.000000` (`0.0000`) |  `0.000000` (`0.0000`) |  
| **R08** |  Formularios e interfaces en Angular |  `angular formularios interfaces` |  `0.000000` (`0.0000`) |  `0.000000` (`0.0000`) |  `0.000000` (`0.0000`) |  
| **R09** |  **Consumir una API desde Angular** *(no valorado)* |  `angular web api` |  `0.000000` (`0.0000`) |  `0.495760` (`0.4958`) |  **`+0.495760` (`+0.4958`)** |  
| **R10** |  Consultas SQL sobre datos |  `sql datos consultas` |  `0.205290` (`0.2053`) |  `0.102605` (`0.1026`) |  `-0.102685` (`-0.1027`) |  
| **R11** |  Tablas de datos en SQL |  `sql datos tablas` |  `0.205290` (`0.2053`) |  `0.102605` (`0.1026`) |  `-0.102685` (`-0.1027`) |  
| **R12** |  Conectar Python con datos SQL |  `python sql datos` |  `0.496522` (`0.4965`) |  `0.464168` (`0.4642`) |  `-0.032353` (`-0.0324`) |  
  
   
*(Nota: en la prueba de desempate, recursos con idéntica puntuación como `R05` y `R06` (`0.437777`), o `R10` y `R11` (`0.205290` antes / `0.102605` después), se ordenan de manera determinista por su identificador ascendente `R05 < R06` y `R10 < R11`).*  
***4. Explicación del cambio de puntuación en recursos distintos del valorado (****`R05`* ***y*** *`R09`****)***  
- **Puntuación de `R05` (*Principios de una API HTTP*, etiquetas: `web api http`) antes y después**:  
  - **Antes**: `0.000000` (`0.0000`)  
  - **Después**: `0.437777` (`0.4378`), con un incremento de **`+0.437777`**.  
- **Puntuación de `R09` (*Consumir una API desde Angular*, etiquetas: `angular web api`) antes y después**:  
  - **Antes**: `0.000000` (`0.0000`)  
  - **Después**: `0.495760` (`0.4958`), con un incremento de **`+0.495760`**, lo que además le permite ingresar al **3.º puesto del ranking de recomendaciones de `u1`** desplazando a `R12` y `R03`.  
- **Por qué este cambio demuestra la adaptación del sistema**:  
   
 Inicialmente, el perfil de `u1` solo tenía pesos positivos en `pandas` (`0.753247`), `datos` (`0.465091`) y `python` (`0.465091`). Como ni `R05` (`web api http`) ni `R09` (`angular web api`) contienen esas tres palabras, su producto punto con el perfil inicial de `u1` era exactamente `0.0`.  
   
 Cuando `u1` señala «Me interesa» sobre `R04` (`python web api`), la función `actualizar_perfil` combina el 40 % del perfil previo con el 60 % del vector TF-IDF de `R04` y normaliza el vector resultante. Como consecuencia, el perfil de `u1` incorpora automáticamente nuevas características del recurso valorado (`api` con peso `0.446269` y `web` con peso `0.446269`) y refuerza el peso de `python` (de `0.465091` a `0.637116`), mientras que los pesos relativos de `pandas` (`0.376477`) y `datos` (`0.232455`) disminuyen proporcionalmente tras la normalización L2.  
   
 Al recalcular la similitud del coseno contra los mismos 12 recursos sin que el usuario haya interactuado directamente con `R05`, `R06` ni `R09`, estos tres recursos adquieren una similitud positiva significativa (`0.4378` para `R05` y `R06`, y `0.4958` para `R09`) porque comparten los nuevos términos `web` y `api` aprendidos en el perfil de `u1`. Del mismo modo, `R01` (Fundamentos de Python) también sube de `0.1861` a `0.2549` (`+0.0688`) debido al mayor peso de `python` en el perfil actualizado.  
***5. Problemas detectados y acciones realizadas***  
- No se detectaron fallos ni incumplimientos en los casos de prueba `T1`–`T4` tras completar las cuatro funciones en `motor.py`. Se verificó que:  
   
1. `crear_perfil` utilice exclusivamente `vectorizador.transform([intereses])` y nunca `fit_transform`, preservando la dimensión de 17 términos del vocabulario del catálogo.  
2. `calcular_puntuaciones` maneje correctamente vectores de ceros (`u3` y `u4`) devolviendo un vector 1D de 12 ceros sin errores de división por cero.  
3. `recomendar` excluya estrictamente recursos con puntuación `<= 0` y conserve las puntuaciones en punto flotante sin redondear durante el cálculo.  
  
---  
**Respuestas a las preguntas conceptuales**  
**Pregunta 1 (2 pts)**  
***¿Qué información describe el contenido de los recursos y qué información constituye el perfil del usuario? Identifique ambas en los archivos del proyecto.***  
- **Información que describe el contenido de los recursos:**  
   
 En el archivo `datos/recursos.csv`, cada recurso posee cuatro campos (`id`, `titulo`, `descripcion` y `etiquetas`), pero la información que representa matemáticamente su contenido para el motor de recomendación es exclusivamente la columna `etiquetas` (por ejemplo, `"python datos pandas"` en `R02` o `"python web api"` en `R04`). En `motor.py`, la función `construir_indice(recursos)` transforma `recursos["etiquetas"]` mediante `TfidfVectorizer().fit_transform(...)` en la matriz `matriz` de dimensión 12 \times 17, donde cada fila es el vector TF-IDF de un recurso. Las columnas `id`, `titulo` y `descripcion` se emplean únicamente para validación e interfaz (`mostrar_catalogo` y `construir_resultado`).  
- **Información que constituye el perfil del usuario:**  
   
 En `datos/perfiles.json`, cada usuario (`u1`, `u2`, `u3`, `u4`) tiene un objeto con su `nombre` descriptivo y la cadena `intereses` (los intereses iniciales declarados, p. ej., `"python datos pandas"` para `u1`). Durante la ejecución (`app.py` y `motor.py`), el estado del usuario está constituido por:  
1. Sus intereses iniciales en `datos/perfiles.json` (punto de partida al iniciar la aplicación).  
2. Su vector actual del perfil en memoria (`estado[usuario_id]`, creado inicialmente por `crear_perfil(datos["intereses"], vectorizador)` como un `np.ndarray` 1D de 17 pesos TF-IDF y actualizado dinámicamente por `actualizar_perfil` tras cada interacción).  
3. Su historial de interacciones en memoria (`historial[usuario_id]`, inicializado por `crear_historial(usuarios)` en `app.py`), que almacena los eventos `{"numero", "accion": "Me interesa", "recurso"}` asociados a ese usuario.  
**Pregunta 2 (2 pts)**  
***¿Por qué se utiliza `fit_transform` sobre el catálogo y `transform` sobre los intereses del usuario? ¿Qué problema causaría ajustar un vectorizador diferente para cada perfil?***  
- **Por qué se usa `fit_transform` en el catálogo y `transform` en el perfil:**  
 `fit_transform` realiza dos operaciones sobre `recursos["etiquetas"]`: primero aprende (`fit`) el vocabulario global del catálogo (las 17 palabras únicas y el índice de columna asignado a cada una) junto con los pesos IDF (Inverse Document Frequency) calculados a partir de la frecuencia de cada término en los 12 documentos del catálogo; segundo, transforma (`transform`) cada recurso en un vector de 17 dimensiones dentro de ese espacio. Por su parte, al representar los intereses de un usuario en `crear_perfil`, se utiliza únicamente `transform([intereses])` para proyectar las palabras del usuario en ese mismo espacio vectorial de 17 columnas y con los mismos pesos IDF del catálogo.  
- **Problema que causaría ajustar (`fit`) un vectorizador diferente para cada perfil:**  
   
 Si se llamara a `fit_transform` (o se instanciara un `TfidfVectorizer` nuevo) sobre el texto de intereses de cada usuario, el vectorizador aprendería únicamente el vocabulario presente en ese texto puntual:  
  1. Incompatibilidad dimensional: Para `u1` (`"python datos pandas"`), el vector tendría longitud 3 en lugar de 17 (y para `u3` con `""` lanzaría un `ValueError: empty vocabulary`), haciendo imposible calcular `cosine_similarity` contra la matriz del catálogo (12 \times 17) por incompatibilidad de dimensiones.  
  2. Desalineación semántica de índices e IDF inválido: Incluso si coincidiera alguna dimensión, el índice 0 de un vectorizador ajustado al perfil no correspondería al mismo término que el índice 0 del catálogo (`angular`), y los valores IDF calculados sobre un único texto de usuario carecerían de sentido estadístico.  
**Pregunta 3 (2 pts)**  
***¿Por qué el mismo catálogo puede producir recomendaciones distintas para `u1` y `u2`? ¿Una similitud de 0,80 significa que existe un 80 % de probabilidad de que el recurso guste? Justifique.***  
- **Por qué el mismo catálogo produce recomendaciones distintas para `u1` y `u2`:**  
   
 Aunque la matriz del catálogo (12 \times 17) es fija y común para todos, la recomendación basada en contenido calcula la similitud del coseno entre la matriz del catálogo y el vector de perfil específico de cada usuario (`calcular_puntuaciones(perfil, matriz)`). El vector de `u1` tiene pesos positivos únicamente en las dimensiones de `python`, `datos` y `pandas`, cuya dirección apunta hacia recursos como `R02` (`1.0000`), `R12` (`0.4965`) y `R03` (`0.4326`), mientras que es ortogonal (similitud `0.0`) a los recursos exclusivos de Angular. En cambio, el vector de `u2` tiene pesos positivos en `angular`, `componentes` e `interfaces`, apuntando en una dirección completamente distinta del espacio vectorial, donde los recursos más cercanos angularmente son `R07` (`1.0000`), `R08` (`0.5677`) y `R09` (`0.3087`).  
- **¿Una similitud de 0,80 significa un 80 % de probabilidad de que el recurso guste?:**  
 No. La similitud del coseno (\cos(\theta) = \frac{\mathbf{p} \cdot \mathbf{r}}{\\|\mathbf{p}\\|\_2 \\|\mathbf{r}\\|\_2}) es una medida geométrica de alineación direccional entre dos vectores de pesos léxicos TF-IDF, no una distribución de probabilidad ni la salida calibrada de un modelo probabilístico o supervisado. Un valor de `0,80` indica únicamente que el ángulo entre el vector de términos del perfil y el vector de etiquetas del recurso tiene un coseno de `0,80` (alta superposición de palabras clave ponderadas por TF-IDF), pero no modela la probabilidad real de agrado, ni evalúa la calidad pedagógica, la dificultad del recurso ni la comprensión semántica profunda del estudiante.  
**Pregunta 4 (2 pts)**  
***¿En qué se diferencia modificar manualmente los intereses iniciales de la adaptación implementada? Identifique el evento que provoca el cambio, el estado modificado y su efecto observable.***  
- Diferencia entre modificar manualmente los intereses iniciales y la adaptación implementada:  
   
 Modificar manualmente los intereses iniciales (por ejemplo, editar `datos/perfiles.json` o pedir al usuario que reescriba un formulario de palabras clave desde cero) es una configuración estática o declarativa explícita en la que el usuario debe conocer el vocabulario del dominio y reemplazar manualmente su perfil. En cambio, la adaptación implementada ocurre en tiempo de ejecución a partir del comportamiento e interacción del usuario con los recursos del catálogo, ajustando gradualmente los pesos de su representación vectorial mediante una regla matemática que combina la memoria de sus intereses previos (40 %) con las características del nuevo recurso de interés (60 %) sin perder por completo sus preferencias anteriores.  
   
- Identificación de los tres elementos del mecanismo adaptativo:  
- **Evento que provoca el cambio:** La acción de retroalimentación explícita «Me interesa» del usuario activo sobre un recurso del catálogo (por ejemplo, `u1` selecciona `R04` en la opción `m` de consola o durante `registrar_interes(estado, "u1", "R04", ...)`).  
- **Estado modificado:** El vector numérico actual del usuario activo en memoria (`estado["u1"]`), que se transforma mediante `actualizar_perfil(anterior, matriz[indice])` aplicando `normalize([0.4 * perfil + 0.6 * vector_recurso])[0]` (pasando de tener 3 términos activos a 5 términos ponderados: `python`, `api`, `web`, `pandas`, `datos`), además de añadirse el registro `{"numero": 1, "accion": "Me interesa", "recurso": "R04"}` a `historial["u1"]`.  
- **Efecto observable:** Sin modificar el catálogo de 12 candidatos ni los perfiles de otros usuarios (`u2`), al recalcular las puntuaciones de similitud cambian tanto los puntajes como el ranking mostrado a `u1`: el recurso valorado `R04` sube de `0.2510` a `0.8752` (1.º lugar) y recursos no valorados que comparten las nuevas características `web` y `api` (como `R09`, que pasa de `0.0000` a `0.4958`, y `R05`, que pasa de `0.0000` a `0.4378`) incrementan su puntuación, entrando `R09` al top 3 de recomendaciones.  
**Pregunta 5 (2 pts)**  
***Relacione su implementación con las etapas del ciclo MAPE-K estudiado en clase: Monitorizar, Analizar, Planificar, Ejecutar y Conocimiento. Indique una función, operación o dato de su programa para cada etapa.***  
  
|  |  |  |  
| - | - | - |  
| **Etapa MAPE-K** |  **Rol en el sistema adaptativo** |  **Función, operación o dato concreto en el programa (`app.py` / `motor.py`)** |  
| Monitorizar (Monitor) |  Capturar las interacciones y eventos del usuario sobre el entorno en tiempo de ejecución. |  Recolección de la opción `"m"` (Me interesa) y el `recurso_id` en `interactivo()` (`app.py`), y el registro del evento en `registrar_interes(...)` añadiendo `{"numero": ..., "accion": "Me interesa", "recurso": recurso_id}` a `historial[usuario_id]`. |  
| Analizar (Analyze) |  Validar el evento recibido y determinar cómo impactan las características del recurso seleccionado sobre el estado actual del usuario y su afinidad con el catálogo. |  Validación del identificador mediante `obtener_indice(recursos, recurso_id)` y el cálculo de la combinación ponderada `mezcla = (1.0 - alpha) * perfil + alpha * vector_recurso` junto con la evaluación de similitud en `calcular_puntuaciones(perfil, matriz)`. |  
| Planificar (Plan) |  Decidir la nueva estructura del perfil normalizado y seleccionar qué recursos formarán la nueva recomendación adaptada según las reglas de negocio. |  Normalización L2 en `actualizar_perfil(...)` (`normalize([mezcla])[0]`) y selección/ordenamiento en `recomendar(...)`, que filtra puntuaciones `> 0`, ordena por `(-puntuacion, id)` y selecciona hasta `top_k = 3` candidatos junto con sus explicaciones (`explicar_coincidencias`). |  
| Ejecutar (Execute) |  Aplicar efectivamente el cambio de estado en el sistema y presentar la adaptación resultante al usuario. |  Actualización efectiva en memoria `estado[usuario_id] = nuevo.copy()` dentro de `registrar_interes(...)` y visualización del nuevo perfil y del nuevo ranking adaptado en consola mediante `mostrar_resultados(...)` y `describir_perfil(...)`. |  
| Conocimiento (Knowledge) |  Repositorio compartido de datos, modelos vectoriales, perfiles e historiales que todas las etapas consultan y actualizan. |  El DataFrame `recursos` (`datos/recursos.csv`), los perfiles iniciales `usuarios` (`datos/perfiles.json`), el índice `vectorizador` y la `matriz` TF-IDF construidos por `construir_indice`, así como los diccionarios de estado en memoria `estado` e `historial`. |  
  
   
**Pregunta 6 (2 pts)**  
***Indique una limitación del mecanismo implementado y proponga una mejora concreta. Explique qué dato adicional o qué cambio del algoritmo exigiría esa mejora.***  
- **Limitación identificada**:  
   
 El sistema presenta un problema de arranque en frío (cold-start) y coincidencia estrictamente léxica (exactitud de tokens) combinada con un único tipo de retroalimentación positiva fija (\alpha = 0.6). Por un lado, si un usuario declara términos sinónimos o afines que no aparecen literalmente en la columna `etiquetas` (como en `u4`, o si alguien escribiera `" gráficas "` o `"frontend"` en lugar de `"graficos"` o `"angular"`), el vector TF-IDF resulta completamente nulo (`0.0`) y el sistema no puede recomendar nada. Por otro lado, si un usuario pulsa repetidamente «Me interesa» sobre recursos con las mismas etiquetas o se equivoca al valorar un recurso, no existe retroalimentación negativa («No me interesa») ni control de diversidad/saturación, y además el recurso ya consumido/valorado (`R04`) pasa a ocupar permanentemente el primer puesto de la recomendación.  
   
- **Mejora concreta propuesta**:  
   
 Incorporar **retroalimentación negativa («No me interesa» / penalización tipo Rocchio)** y **distinción entre recursos ya consumidos/valorados y recursos por descubrir**, o bien enriquecer la representación con *n-gramas de caracteres / embeddings semánticos*.  
- **Dato adicional y cambio algorítmico requerido**:  
  1. **Dato adicional**:  
- En cada evento de `historial[usuario_id]`, almacenar el tipo de valoración (`accion: "Me interesa" | "No me interesa"`) o una puntuación explícita, además de conservar el conjunto de `ids_valorados` por el usuario.  
  2. **Cambio en el algoritmo (motor.py)**:  
- En `actualizar_perfil`: Implementar la fórmula de retroalimentación de Rocchio modificada, recibiendo el tipo de acción:  
 \mathbf{p}\_{\text{nuevo}} = \text{normalize}\left(\alpha \mathbf{p}\_{\text{actual}} + \beta \mathbf{r}\_{\text{positivo}} - \gamma \mathbf{r}\_{\text{negativo}}\right)  
   
 donde tras restar \gamma \mathbf{r}\_{\text{negativo}} se aplica `np.maximum(mezcla, 0)` (recorte a valores no negativos) antes de normalizar con L2, evitando pesos negativos incompatibles con la interpretación TF-IDF.  
- En `recomendar`: Añadir un parámetro opcional `excluir_vistos: bool = False` (o un factor de penalización de novedad) que consulte `historial[usuario_id]` para que, una vez demostrada la actualización de las 12 puntuaciones, la interfaz pueda priorizar recursos nuevos afines (como R09 y R05) por encima de recursos que el estudiante ya marcó como conocidos.  
  
---  
**Archivos de la entrega**  
El repositorio contiene todos los archivos obligatorios solicitados:  
- **`motor.py`:** Las cuatro funciones completadas (`crear_perfil`, `calcular_puntuaciones`, `recomendar`, `actualizar_perfil`) y documentadas.  
- **`app.py`:** Aplicación de consola y ejecución de las pruebas `--comprobar` y `--demo`.  
- **`datos/recursos.csv`:** Catálogo original de 12 recursos utilizado en las pruebas.  
- **`datos/perfiles.json`:** Perfiles iniciales de los usuarios (`u1`, `u2`, `u3`, `u4`).  
- **`evidencias/reporte_demo.json`:** Registro JSON generado por `python app.py --demo` con los perfiles, recomendaciones, interacciones y comprobaciones `T1`–`T4`.  
- **`evidencias/puntuaciones_antes_despues.csv`:** Comparación de las puntuaciones de los 12 candidatos antes y después de actualizar el perfil de `u1`.  
- **`README.md`:** Identificación, instrucciones de reproducción, respuestas a las 6 preguntas conceptuales y análisis completo de las pruebas.  
- **`requirements.txt` y `.gitignore`:** Dependencias congeladas del proyecto y exclusiones (`.venv/`, `__pycache__/`, `*.py[cod]`, `.DS_Store`, `node_modules/`).  
**Reproducción desde una copia nueva**  
Cualquier persona que clone este repositorio en una copia limpia puede reproducir exactamente los resultados ejecutando:  
```  
python3 -m venv .venv  
 source .venv/bin/activate   # En Windows: .venv\Scripts\activate  
 python -m pip install -r requirements.txt  
 python app.py --comprobar  
 python app.py --demo  
   
```  
