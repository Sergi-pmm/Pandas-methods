# Guía de Pandas - Limpieza y Análisis de Datos

Esto es un recopilatorio de las funciones que más uso en pandas para trabajar con datos. Lo voy actualizando según necesito cosas nuevas.

## 1. Inspección básica
```python
df.head()           # Primeras filas
df.tail()           # Últimas filas
df.info()           # Info general + tipos de datos
df.describe()       # Estadísticas básicas
df.columns.tolist() # Listar columnas
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

## 3. Eliminar columnas y duplicados
```python
# Columnas
df.drop('columna', axis=1, inplace=True)
df.drop(['col1', 'col2'], axis=1, inplace=True)

# Duplicados 
df.drop_duplicates()
df.drop_duplicates(subset=['numero_serie'])  # Por columna específica
df.drop_duplicates(subset=['col1'], keep='first')  # Mantiene el primero
df.drop_duplicates(subset=['col1'], keep='last')   # Mantiene el último
df.drop_duplicates(subset=['col2'], keep=False)    # Elimina TODOS (cuidado con esto)

# Ver duplicados antes de borrar
df[df.duplicated(subset=['columna'], keep=False)]
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
df['codigo_ref'] = df['codigo_ref'].astype(int).astype(str).str.zfill(5)
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

## 9. Fechas
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

## 10. Aplicar funciones
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

## 11. Exportar a Excel
```python
# Simple
df.to_excel('datos.xlsx', index=False)

# Múltiples pestañas
with pd.ExcelWriter('informe.xlsx') as writer:
    df_categorias.to_excel(writer, sheet_name='Por Categoría', index=False)
    df_marcas.to_excel(writer, sheet_name='Por Marca', index=False)
    df_resumen.to_excel(writer, sheet_name='Resumen', index=False)
```

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

**Última actualización:** Octubre 2025
