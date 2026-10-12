# Guía de Pandas - Limpieza y Análisis de Datos

Esto es un recopilatorio de las funciones que más uso en pandas para trabajar con datos. Lo voy actualizando según necesito cosas nuevas.

## 1. Inspección básica
```python
df.head()           # Primeras filas
df.tail()           # Últimas filas
df.info()           # Info general + tipos de datos
df.describe()       # Estadísticas básicas
df.columns.tolist() # Listar columnas, sin el "tolist()" también se puede visualizar
df.shape            # Número de filas y columnas
len(df)             # Total de filas
```

## 2. Valores nulos
```python
# Ver cuántos nulos hay
df.isnull().sum()
df['columna'].isna().sum()

# Eliminar nulos
df.dropna()  # Elimina filas con cualquier nulo
df.dropna(subset=['col1', 'col2'])  # Solo si hay nulos en estas columnas
df.dropna(how='all')  # Solo si TODA la fila es nula
df.dropna(thresh=3)   # Mantener filas con al menos 3 valores válidos

# Rellenar nulos
df.fillna(0)  # Rellenar con cero
df.fillna(df['col'].mean())  # Rellenar con la media
```

## 3. Eliminar duplicados y columnas
```python

# Ver cuantos duplicados hay
df.duplicated().sum()
df.duplicated(subset=['col1']).sum()
df.duplicated(subset=['col1','col2']).sum()

# Duplicados 
df.drop_duplicates()
df.drop_duplicates(subset=['numero_serie'])  # Por columna específica
df.drop_duplicates(subset=['col1'], keep='first')  # Mantiene el primero
df.drop_duplicates(subset=['col1'], keep='last')   # Mantiene el último
df.drop_duplicates(subset=['col2'], keep=False)    # Elimina TODOS (cuidado con esto)

# Ver duplicados antes de borrar
df[df.duplicated(subset=['columna'], keep=False)]

# Columnas
df.drop('columna', axis=1, inplace=True)
df.drop(['col1', 'col2'], axis=1, inplace=True)
```

## 4. Cambiar nombres y reordenar
```python
# Renombrar
df.rename(columns={'nombre_antiguo': 'nombre_nuevo'})

# Reordenar columnas
df = df[['col1', 'col2', 'col3']]  # Orden específico
df = df[sorted(df.columns)]  # Alfabéticamente

# Mover columnas al principio
cols_primero = ['id', 'nombre']
otras = [c for c in df.columns if c not in cols_primero]
df = df[cols_primero + otras]
```

## 5. Conversión de tipos

**Nota:** Para tratar con problemas como códigos de referencia que pueden tratarse como fechas, enteros o strings. Por ejemplo valores que empiezan por 0...
```python
# Básico
df['col'].astype(int)
df['col'].astype(str)
df['col'].astype('Int64')  # Permite nulos (con mayúscula!)

# Numérico con manejo de errores
pd.to_numeric(df['col'], errors='coerce')  # Invalidos → NaN

# Fechas
pd.to_datetime(df['fecha'], errors='coerce')
pd.to_datetime(df['fecha'], format='%d/%m/%Y', errors='coerce')
```

**Truco para códigos de referencia:**
```python
# Si están en formato float con .0
df['codigoPostal'] = df['codigoPostal'].astype(int).astype(str).str.zfill(5)
# Primero int (quita el .0), luego string, luego rellena con ceros
```

## 6. Trabajar con texto (strings)
```python
df['col'].str.upper()     # MAYÚSCULAS
df['col'].str.lower()     # minúsculas
df['col'].str.strip()     # Quitar espacios

# Reemplazar
df['col'].str.replace('viejo', 'nuevo', regex=False)
df['col'].str.replace(r'^http://', 'https://', regex=True)

# Validaciones
df['col'].str.isnumeric()  # ¿Es número?
df['col'].str.startswith('A', na=False)  # ¿Empieza por A?
df['col'].str.len()  # Longitud

# Rellenar con ceros (típico para referencias)
df['ref'].str.zfill(5)  # 8302 → 08302
```

**Uso de Expresiones Regulares (Regex):**
```python
df['num_serie'] = (
    df['num_serie']
    .astype(str)
    .str.replace(r'^REF-', '', regex=True)  # Quitar prefijo
)
df = df[df['num_serie'].str.startswith('2024', na=False)]  # Solo de este año
```

## 7. Filtrar datos

### Filtrado básico
```python
df[df['precio'] > 1000]
df[df['marca'] == 'Toyota']
df[(df['precio'] > 1000) & (df['marca'] == 'Toyota')]  # AND
df[(df['año'] < 2010) | (df['año'] > 2023)]  # OR
```

### Filtrar por múltiples valores

Esto lo uso muchísimo cuando necesito filtrar por varias marcas, categorías, etc:
```python
# Opción 1: Usando isin() - LA MÁS USADA
marcas = ['Toyota', 'Honda', 'Ford']
df = df[df['marca'].isin(marcas)]

# Opción 2: Con múltiples OR (no recomendado, muy largo)
df = df[(df['marca'] == 'Toyota') | (df['marca'] == 'Honda') | (df['marca'] == 'Ford')]

# Ejemplos prácticos:
# Filtrar por varias categorías
categorias_interes = ['SUV', 'Sedan', 'Deportivo', 'Familiar']
df_filtrado = df[df['categoria'].isin(categorias_interes)]

# Filtrar referencias que empiecen por A, B o C
df = df[df['referencia'].str[0].isin(['A', 'B', 'C'])]

# Excluir valores (lo contrario con ~)
excluir = ['Toyota', 'Honda']
df = df[~df['marca'].isin(excluir)]  # El ~ es negación
```

### Otros filtros útiles
```python
df[df['col'].notna()]  # No nulos
df[df['col'].isna()]   # Nulos
df[df['precio'].between(10000, 50000)]  # Rango
```

## 8. Conteos y agrupaciones

Para analizar datos:
```python
# Contar valores únicos
df['marca'].value_counts()
df['marca'].nunique()  # Solo el número

# Crear DataFrame del conteo (muy útil para exportar)
conteo = df['marca'].value_counts().reset_index()
conteo.columns = ['marca', 'cantidad']

# Agrupar y contar
df.groupby('categoria').size().reset_index(name='cantidad')
df.groupby(['categoria', 'marca']).size().reset_index(name='cantidad')

# Ordenar
df.sort_values('cantidad', ascending=False)
```

**Mi patrón típico de análisis:**
```python
# Conteo con porcentaje
conteo = df['columna'].value_counts().reset_index()
conteo.columns = ['columna', 'cantidad']
total = conteo['cantidad'].sum()
conteo['porcentaje'] = (conteo['cantidad'] / total * 100).round(2)
conteo = conteo.sort_values('cantidad', ascending=False)
```

## 9. Cruzar tablas (merge)

Para juntar dos tablas que tienen una columna en común (lo mismo que un BUSCARV de Excel o un JOIN de SQL):
```python
# Añadir a cada venta los datos de su cliente
df = pd.merge(df_ventas, df_clientes, on='id_cliente', how='left')

# Lo mismo escrito desde el DataFrame
df = df_ventas.merge(df_clientes, on='id_cliente', how='left')
```

### Parámetros principales
- `on`: Columna (o lista de columnas) por la que se cruza. Tiene que llamarse igual en las dos tablas
- `how`: Tipo de cruce (ver abajo). Si no se pone nada es `'inner'`
- `left_on` / `right_on`: Si la columna se llama distinto en cada tabla (se usan en vez de `on`)
- `suffixes`: Sufijos para las columnas que se repiten en las dos tablas. Por defecto `_x` e `_y`
- `indicator=True`: Añade la columna `_merge` que dice de qué tabla viene cada fila
- `validate`: Da error si la clave está duplicada donde no debería (ver más abajo)

### Tipos de cruce (how)
La tabla de la izquierda es la primera que se pone y la de la derecha la segunda.
- `how='left'`: Todas las filas de la izquierda + lo que encuentre en la derecha. Lo que no encuentra → NaN. **LA MÁS USADA**, no pierdes ninguna fila de tu tabla principal
- `how='inner'`: Solo las filas que están en las dos tablas (cuidado, puedes perder filas sin darte cuenta)
- `how='right'`: Lo contrario que left
- `how='outer'`: Todas las filas de las dos tablas, con NaN donde no cruza
```python
# df_ventas tiene los clientes 1, 2, 3
# df_clientes tiene los clientes 1, 2, 4
pd.merge(df_ventas, df_clientes, on='id_cliente', how='left')   # 1, 2, 3 (el 3 sin datos de cliente)
pd.merge(df_ventas, df_clientes, on='id_cliente', how='inner')  # 1, 2
pd.merge(df_ventas, df_clientes, on='id_cliente', how='right')  # 1, 2, 4 (el 4 sin datos de venta)
pd.merge(df_ventas, df_clientes, on='id_cliente', how='outer')  # 1, 2, 3, 4
```

### Casos típicos
```python
# La columna se llama distinto en cada tabla
pd.merge(df_ventas, df_clientes, left_on='cod_cliente', right_on='id', how='left')
# Se quedan las dos columnas (cod_cliente e id), se puede borrar una con drop

# Cruzar por varias columnas a la vez
pd.merge(df_stock, df_precios, on=['marca', 'modelo'], how='left')

# Columnas con el mismo nombre en las dos tablas
pd.merge(df_2024, df_2025, on='referencia', suffixes=('_2024', '_2025'))
# precio_2024 y precio_2025 en vez de precio_x y precio_y

# Traer solo algunas columnas de la otra tabla
pd.merge(df_ventas, df_clientes[['id_cliente', 'nombre']], on='id_cliente', how='left')
```

### Ver qué ha cruzado y qué no
```python
df = pd.merge(df_ventas, df_clientes, on='id_cliente', how='left', indicator=True)
df['_merge'].value_counts()  # both / left_only / right_only

# Ventas cuyo cliente no está en la tabla de clientes
sin_cliente = df[df['_merge'] == 'left_only']

# Borrar la columna cuando ya no hace falta
df = df.drop('_merge', axis=1)
```

### Antes de cruzar (lo que más falla)

**1. La clave tiene que ser del mismo tipo en las dos tablas.** Si en una es número y en la otra texto da error, y si las dos son texto pero en una pone `8302` y en la otra `08302` no cruzan. Pasa mucho con los códigos que empiezan por 0 (ver sección 5):
```python
df_ventas['codigo_ref'].dtype     # Comprobar el tipo en las dos
df_productos['codigo_ref'].dtype

# Dejar las dos como texto de 5 dígitos
df_ventas['codigo_ref'] = df_ventas['codigo_ref'].astype(str).str.zfill(5)
df_productos['codigo_ref'] = df_productos['codigo_ref'].astype(str).str.zfill(5)
# Si viene como float con .0, primero astype(int) (ver truco de la sección 5)
```

**2. Limpiar espacios y mayúsculas.** `'Toyota '` y `'TOYOTA'` no cruzan con `'Toyota'`:
```python
df_stock['marca'] = df_stock['marca'].str.strip().str.upper()
df_precios['marca'] = df_precios['marca'].str.strip().str.upper()
```

**3. Claves duplicadas en la tabla de la derecha → se multiplican filas.** Si un cliente aparece dos veces en `df_clientes`, cada venta suya sale duplicada:
```python
df_clientes['id_cliente'].duplicated().sum()  # Debería ser 0

# Comprobar filas antes y después (con left deberían ser las mismas)
len(df_ventas)
df = pd.merge(df_ventas, df_clientes, on='id_cliente', how='left')
len(df)

# O que pandas avise directamente
pd.merge(df_ventas, df_clientes, on='id_cliente', how='left', validate='many_to_one')
# many_to_one → muchas ventas por cliente, pero cada cliente una sola vez en df_clientes
# Si no se cumple → MergeError
```

### Juntar tablas una debajo de otra (concat)
Para apilar tablas con las mismas columnas, por ejemplo un Excel por mes:
```python
df_total = pd.concat([df_enero, df_febrero, df_marzo], ignore_index=True)
# ignore_index=True → renumera el índice (si no, se repite 0, 1, 2... de cada tabla)
# Si una columna no está en todas las tablas → NaN en las que falta
```

### join (cruzar por índice)
Casi siempre uso merge, pero join es un atajo cuando la clave está en el índice:
```python
df_clientes_idx = df_clientes.set_index('id_cliente')
df_ventas.join(df_clientes_idx, on='id_cliente')  # how='left' por defecto
```

## 10. Fechas
Posibles soluciones a incidencias con fechas...
```python
# Convertir a fecha
pd.to_datetime(df['fecha'], format='%d/%m/%Y', errors='coerce')

# Calcular antigüedad en años
from datetime import datetime
hoy = datetime.now()
df['antiguedad'] = ((hoy - df['fecha_compra']).dt.days / 365.25).astype(int)

# Filtrar fechas válidas (evita errores de overflow)
fecha_min = pd.Timestamp('2000-01-01')
fecha_max = pd.Timestamp.now()
df = df[(df['fecha'] >= fecha_min) & (df['fecha'] <= fecha_max)]

# Extraer partes de la fecha
df['fecha'].dt.year
df['fecha'].dt.month
df['fecha'].dt.day
```

## 11. Aplicar funciones
```python
# Con función normal
df['categoria'] = df['codigo_producto'].apply(obtener_categoria)

# Con lambda
df['nueva_col'] = df['col'].apply(lambda x: x * 2)

# Map con diccionario
df['col'] = df['col'].map(diccionario)
```

**Función ejemplo para categorías:**
```python
def obtener_categoria(codigo):
    mapeo = {
        '01': 'Electrónica', '02': 'Hogar', '03': 'Deportes', '04': 'Ropa',
        '05': 'Libros', '06': 'Juguetes', '07': 'Alimentos', '08': 'Belleza',
        '09': 'Herramientas', '10': 'Jardín', '11': 'Mascotas', '12': 'Música',
        '13': 'Videojuegos', '14': 'Oficina', '15': 'Bebés', '16': 'Salud',
        '17': 'Automóvil', '18': 'Industrial', '19': 'Arte', '20': 'Joyería'
    }
    codigo_str = str(codigo).zfill(5)  # Importante para códigos que empiezan por 0
    prefijo = codigo_str[:2]
    return mapeo.get(prefijo, 'Sin categoría')
```

## 12. Exportar a Excel
```python
# Simple
df.to_excel('datos.xlsx', index=False)

# Múltiples pestañas
with pd.ExcelWriter('informe.xlsx') as writer:
    df_categorias.to_excel(writer, sheet_name='Por Categoría', index=False)
    df_marcas.to_excel(writer, sheet_name='Por Marca', index=False)
    df_resumen.to_excel(writer, sheet_name='Resumen', index=False)
```

## 13. Preparar los df antes de analizar

Lo que reviso con cada Excel nuevo antes de cruzar o analizar nada.

Para probar los snippets: [`tiendas_prueba.xlsx`](tiendas_prueba.xlsx) (`RUTA = 'tiendas_prueba.xlsx'`). Son tiendas que pertenecen a cadenas: `id_tienda` identifica una tienda e `id_cadena` una cadena, que tiene varias tiendas. Por eso `id_cadena` se repite en las hojas de tiendas y es único en la hoja `cadenas`.

### 13.1 Leer todas las hojas y recorrerlas
```python
pd.ExcelFile(RUTA).sheet_names                             # Nombres de las hojas
hojas = pd.read_excel(RUTA, sheet_name=None)               # Todas → diccionario {nombre: df}
hojas = pd.read_excel(RUTA, sheet_name=['2025', '2026'])   # Solo algunas

for nombre, df in hojas.items():
    print(nombre, df.shape)
    print(df.columns.tolist())
```

### 13.2 Cabecera que no empieza en la fila 1
Se nota porque salen columnas `Unnamed: 1`, `Unnamed: 2`...
```python
for nombre, df in hojas.items():
    if df.columns.astype(str).str.contains('Unnamed').any():
        print(nombre)

pd.read_excel(RUTA, sheet_name='externa', header=None).head(10)          # Ver en qué fila empieza
hojas['externa'] = pd.read_excel(RUTA, sheet_name='externa', header=2)   # Fila 3 de Excel = header=2
```

### 13.3 Espacios en los nombres de columna
```python
'Ventas' in df.columns   # False aunque se vea → hay espacios
df.columns.tolist()      # Con tolist se ven: 'Ventas '

for df in hojas.values():
    df.columns = df.columns.str.strip()
```

### 13.4 La misma columna con nombres distintos en cada hoja
```python
set(hojas['2025'].columns) & set(hojas['2026'].columns)   # Comunes
set(hojas['2025'].columns) ^ set(hojas['2026'].columns)   # Solo en una de las dos

nombres = {'ID Tienda': 'id_tienda', 'Cadena ID': 'id_cadena', 'Cadena': 'cadena',
           'Ciudad': 'ciudad', 'Ventas': 'ventas', 'Estado': 'estado'}
for nombre in hojas:
    hojas[nombre] = hojas[nombre].rename(columns=nombres)   # Ignora los nombres que no encuentra
```
**Ojo:** dentro del bucle, `df = df.rename(...)` no cambia el diccionario. Hay que guardarlo en `hojas[nombre]`.

### 13.5 El mismo nombre de columna dos veces en una hoja
Por ejemplo, una columna creada aparte con una fórmula que trae el dato de otra hoja. Al leer, pandas le pone `.1` a la segunda.
```python
df.columns[df.columns.str.contains(r'\.\d+$')]   # Las que ha renombrado pandas: 'estado.1'
df.columns.duplicated().sum()                    # Da 0 aunque en Excel estén repetidas (por el .1)

df = df.rename(columns={'estado.1': 'estado_externa'})   # Que el nombre diga de dónde viene
df[df['estado'] != df['estado_externa']]                 # Filas donde no coinciden
```
**Ojo:**
- El `strip` puede crear repetidas: `'estado'` y `'estado '` pasan a ser dos `'estado'`, y entonces `df['estado']` devuelve dos columnas. Comprueba `df.columns.duplicated().sum()` después del `strip` y, si sale algo, renombra antes de hacerlo.
- `read_excel` lee el último valor que guardó Excel, no la fórmula. Si una columna con fórmula sale vacía, abre el archivo en Excel, guárdalo y vuelve a leerlo.

### 13.6 Tipos: qué es número y qué no
```python
df.dtypes                                                  # object en una columna de números → hay texto mezclado
df['ventas'].map(type).value_counts()                      # Tipos que hay en una columna
df.map(type).astype(str).apply(pd.Series.value_counts).T   # En todas las columnas

num = pd.to_numeric(df['ventas'], errors='coerce')
df.loc[num.isna() & df['ventas'].notna(), 'ventas']        # Los que no se pueden convertir

df['ventas_nd'] = df['ventas'] == 'n/d'                    # Guardar los textos con significado antes de convertir
df['ventas'] = pd.to_numeric(df['ventas'].astype(str).str.replace(' ', ''), errors='coerce')
```
**Ojo:** sin `astype(str)`, `.str.replace` deja en NaN los valores que ya eran número.

### 13.7 Ficha rápida de cada hoja
```python
for nombre, df in hojas.items():
    print(nombre)
    display(pd.DataFrame({'tipo': df.dtypes, 'con dato': df.count(), 'distintos': df.nunique()}))
```
Si `distintos` = `con dato` = nº de filas, esa columna no se repite y puede servir de clave.

### 13.8 Limpiar ids y comprobar que la clave es única
```python
df['id_tienda'] = df['id_tienda'].str.strip().str.upper()
df['id_tienda'].str.len().value_counts()     # Todos deberían tener la misma longitud

df['id_tienda'].is_unique                    # ¿Una fila por tienda?
df['id_tienda'].dropna().is_unique           # Los vacíos también cuentan como repetidos
df[df['id_tienda'].duplicated(keep=False)]   # Ver todas las filas repetidas
```

### 13.9 Funciones para reutilizar y preparar todas las hojas de golpe
```python
def limpiar_columnas(df):
    df.columns = df.columns.str.strip()
    return df

def limpiar_id(s):
    return s.str.strip().str.upper()

def a_numero(s):
    return pd.to_numeric(s.astype(str).str.replace(' ', ''), errors='coerce')

def ficha(df):
    return pd.DataFrame({'tipo': df.dtypes, 'con dato': df.count(), 'distintos': df.nunique(),
                         'ejemplo': df.apply(lambda s: s.dropna().iloc[0] if s.notna().any() else None)})

def buscar(hojas, col, valor):
    for nombre, df in hojas.items():
        if col in df.columns:
            print(nombre)
            display(df[df[col] == valor].T)   # .T: en vertical, se lee mejor
```
Con las funciones, todo lo anterior en un bucle:
```python
nombres = {'ID Tienda': 'id_tienda', 'Cadena ID': 'id_cadena', 'Cadena': 'cadena', 'Ciudad': 'ciudad',
           'Ventas': 'ventas', 'Estado': 'estado', 'estado.1': 'estado_externa'}

hojas = pd.read_excel(RUTA, sheet_name=None)
hojas['externa'] = pd.read_excel(RUTA, sheet_name='externa', header=2)

for nombre in hojas:
    df = limpiar_columnas(hojas[nombre]).rename(columns=nombres)
    for col in ['id_tienda', 'id_cadena']:
        if col in df.columns:
            df[col] = limpiar_id(df[col])
    if 'ventas' in df.columns:
        df['ventas'] = a_numero(df['ventas'])
    hojas[nombre] = df
    print(nombre)
    display(ficha(df))
```

### 13.10 Buscar un id en todas las hojas
```python
buscar(hojas, 'id_tienda', 'T005')
buscar(hojas, 'id_cadena', 'C02')
```

### 13.11 Comparar ids entre hojas (sets)

| Operación | Resultado |
|---|---|
| `a & b` | En las dos |
| `a - b` | En `a` pero no en `b` |
| `a ^ b` | En una sola de las dos |
| `a \| b` | En cualquiera de las dos |
| `'T005' in a` | ¿Está ese id? |

**Entre dos hojas**
```python
ids25 = set(hojas['2025']['id_tienda'].dropna())
ids26 = set(hojas['2026']['id_tienda'].dropna())

ids25 & ids26   # En las dos
ids25 - ids26   # Solo en 2025
ids26 - ids25   # Solo en 2026

hojas['2025'][~hojas['2025']['id_tienda'].isin(ids26)]                 # Las filas, no solo los ids
set(hojas['2026']['id_cadena']) - set(hojas['cadenas']['id_cadena'])   # Cadenas sin ficha → set() si todo bien
```

**Entre todas las hojas: un set por hoja**
```python
ids = {}
for nombre, df in hojas.items():
    if 'id_tienda' in df.columns:
        ids[nombre] = set(df['id_tienda'].dropna())

for nombre, s in ids.items():
    print(nombre, 'T005' in s)    # En qué hojas está una tienda concreta

set.intersection(*ids.values())   # En todas las hojas
set.union(*ids.values())          # En alguna hoja
```

**Tabla de presencia: en qué hojas está cada id y cuántas veces**
```python
partes = []
for nombre, df in hojas.items():
    if 'id_tienda' in df.columns:
        partes.append(df[['id_tienda']].assign(hoja=nombre))
largo = pd.concat(partes)

presencia = pd.crosstab(largo['id_tienda'], largo['hoja'])   # 0 = no está, 2 = repetido en esa hoja
presencia.loc['T005']                                        # Una tienda concreta
presencia[(presencia == 0).any(axis=1)]                      # Las que faltan en alguna hoja
presencia[(presencia > 1).any(axis=1)]                       # Las repetidas en alguna hoja
```
**Ojo:** quita los vacíos con `dropna()` antes de crear el set, y limpia los ids antes (13.8): si no, `'t003 '` y `'T003'` salen como tiendas distintas.

### 13.12 Df aparte con las columnas que quiero
```python
d25, d26, ext, cad = hojas['2025'], hojas['2026'], hojas['externa'], hojas['cadenas']
cols = ['id_tienda', 'id_cadena', 'ventas']

t25 = d25[cols].copy()                                 # Df aparte con esas columnas
t26 = d26.loc[d26['canal'] == 'Tienda', cols].copy()   # Filtrando filas a la vez
ext.filter(items=cols)                                 # Ignora las que no existan (ext no tiene id_cadena)
d26.filter(like='id_')                                 # Columnas cuyo nombre contiene 'id_'
```
**Ojo:** con `.copy()`, al cambiar después el df aparte no sale el `SettingWithCopyWarning`.

### 13.13 concat, merge o join

| Quiero... | Uso |
|---|---|
| Poner tablas una debajo de otra | `concat` |
| Añadir columnas de otra tabla cruzando por una columna | `merge` |
| Añadir columnas de una tabla cuyo índice es un id único | `join` |

Parámetros y trampas de `merge`: sección 9.

**concat: apilar años y marcar de dónde viene cada fila**
```python
largo = pd.concat([t25.assign(año=2025), t26.assign(año=2026)], ignore_index=True)
largo.groupby('año')['ventas'].sum()
```

**merge: qué tiendas cambian entre años**
```python
m = t25.merge(t26, on=['id_tienda', 'id_cadena'], how='outer',
              suffixes=('_2025', '_2026'), indicator=True, validate='one_to_one')
m['_merge'].value_counts()   # both / left_only (solo 2025) / right_only (solo 2026)
m[m['_merge'] != 'both']
m['dif'] = m['ventas_2026'] - m['ventas_2025']
```

**merge: misma columna, distinta fuente**
```python
ext1 = ext.groupby('id_tienda', as_index=False)['ventas'].sum(min_count=1)   # Una fila por tienda antes de cruzar
comp = t26.merge(ext1, on='id_tienda', suffixes=('_nuestras', '_externa'), validate='one_to_one')
comp[comp['ventas_nuestras'] != comp['ventas_externa']]
```

**join: cruzar con un id único como índice**
```python
cad_idx = cad.set_index('id_cadena', verify_integrity=True)   # Da error si algún id se repite
cad_idx.loc['C02']                                            # Buscar por id: una fila
t26 = t26.join(cad_idx, on='id_cadena')                       # Cada tienda con los datos de su cadena

total = t26.groupby('id_cadena')['ventas'].sum().rename('ventas_cadena')   # El índice ya es id_cadena, único
t26 = t26.join(total, on='id_cadena')
t26['peso'] = t26['ventas'] / t26['ventas_cadena']            # Parte de las ventas de su cadena
cad_idx.join(total)                                           # Las dos con id_cadena de índice: sin on
```
**Ojo:**
- `validate` da `MergeError` si la clave se repite donde no debería: mejor un error que filas multiplicadas sin avisar.
- `merge` empareja los vacíos entre sí: quita antes las filas sin id.
- `join` no comprueba nada: si el índice se repite, multiplica filas. Por eso `verify_integrity=True`.

## Diferencias importantes

### map() vs replace()
- `map()`: Transforma TODOS los valores. Si no está en el diccionario → NaN
- `replace()`: Solo cambia los que se indican, el resto igual
```python
df['estado'].map({'N': 'Nuevo', 'U': 'Usado'})  # 'R' → NaN
df['estado'].replace({'N': 'Nuevo', 'U': 'Usado'})  # 'R' → 'R'
```

### astype(int) vs astype('Int64')
- `astype(int)`: No permite nulos → error si hay NaN
- `astype('Int64')`: Permite nulos → aparecen como <NA>

### inplace=True
```python
df.dropna(inplace=True)  # Modifica df directamente
df = df.dropna()  # Crea nuevo DataFrame
```

### merge() vs concat() vs join()
- `merge()`: Cruza por columnas en común → añade columnas (el BUSCARV)
- `concat()`: Pone tablas una debajo de otra → añade filas
- `join()`: Como merge pero cruzando por el índice. Por defecto es `left` (merge es `inner`)

### map() vs merge()
- `map()`: Para traer **una** columna desde un diccionario o una Serie
- `merge()`: Para traer **varias** columnas de otra tabla
```python
# Las dos hacen lo mismo para una sola columna
df_ventas['nombre'] = df_ventas['id_cliente'].map(df_clientes.set_index('id_cliente')['nombre'])
df_ventas = pd.merge(df_ventas, df_clientes[['id_cliente', 'nombre']], on='id_cliente', how='left')
```

---

## Git (comandos básicos)
```bash
git status
git add .
git commit -m "actualizar datos"
git push origin main

# Ver ramas
git branch

# Ver remoto
git remote -v
```

---

**Última actualización:** Octubre 2026
