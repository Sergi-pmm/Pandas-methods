# Pandas-methods

# Limpieza de datos con Pandas

Este repositorio contiene un script de referencia con los comandos más utilizados para la limpieza y transformación de datos en Python mediante la librería `pandas`. El objetivo es proporcionar una guía práctica, legible y reproducible que facilite tanto el aprendizaje como la implementación en proyectos reales.

La estructura del script es directa y sin funciones personalizadas, permitiendo a cualquier usuario adaptar y ejecutar los comandos con facilidad. El enfoque está orientado a reforzar las operaciones fundamentales sin abstraer la lógica interna.

---

## Contenido del repositorio

- `data_cleaning_pandas_reference.py`: script con ejemplos organizados por temática.
- `ejemplo_dataset.csv`: archivo opcional para pruebas (no incluido por defecto).
- Este `README.md`: documentación de referencia.

---

## Secciones

### 1. Inspección de datos

- `df.head()`: muestra las primeras filas del DataFrame.
- `df.info()`: informa sobre tipos de datos y valores nulos.
- `df.describe()`: genera estadísticas descriptivas para columnas numéricas.

### 2. Manejo de valores faltantes

- `df.isnull().sum()`: cuenta los valores nulos por columna.
- `df.dropna()`: elimina filas que contienen al menos un valor nulo.
- `df.fillna(valor)`: reemplaza valores nulos por un valor específico.

### 3. Limpieza y transformación

- `df.drop_duplicates()`: elimina filas duplicadas.
- `df.rename(columns={'antiguo': 'nuevo'})`: cambia el nombre de una o varias columnas.
- `df['col'] = df['col'].astype(tipo)`: convierte el tipo de dato de una columna.
- `df['col'] = df['col'].replace({'a': 'b'})`: reemplaza valores específicos en una columna.
- `df.reset_index(drop=True)`: reinicia el índice del DataFrame.
- `df.drop('col', axis=1)`: elimina una columna.
- `df['col'] = df['col'].str.replace(r'\.0$', '', regex=True)`: elimina decimales artificiales (caso típico en códigos postales).

### 4. Selección y filtrado

- `df.loc[fila, 'col']`: selección por etiqueta.
- `df.iloc[fila, columna]`: selección por posición numérica.
- `df[df['col'] > valor]`: filtrado condicional por columna.

### 5. Agregación y análisis

- `df.groupby('col').agg({'otra_col': 'mean'})`: agrupación y cálculo de media (u otra función).
- `df.sort_values('col', ascending=False)`: ordena las filas por una columna.
- `df['col'].value_counts()`: cuenta los valores únicos de una columna.
- `df.apply(función)`: aplica una función a filas o columnas.
- `df.pivot_table(values, index, columns)`: genera una tabla dinámica.

### 6. Combinación de DataFrames

- `pd.concat([df1, df2])`: concatena DataFrames verticalmente o horizontalmente.
- `pd.merge(df1, df2, on='col')`: une DataFrames según una clave común.
- `df1.join(df2)`: une DataFrames por el índice.
- `df1.append(df2, ignore_index=True)`: añade filas de un DataFrame a otro (obsoleto a partir de pandas 2.0; usar `concat()`).

---

## Funciones adicionales útiles

- `df['col'].nunique()`: número de valores únicos en una columna.
- `df.duplicated()`: identifica filas duplicadas (retorna booleanos).
- `df[df['col'].isin(['a', 'b'])]`: filtra filas cuyo valor pertenece a una lista.
- `df['col'].map({'a': 'A', 'b': 'B'})`: transforma valores según un diccionario (puede generar `NaN` si el valor no está mapeado).
- `pd.crosstab(df['col1'], df['col2'])`: genera una tabla de contingencia.
- `df['fecha'] = pd.to_datetime(df['fecha'])`: convierte texto a fecha.
- `df.set_index('fecha').resample('M').sum()`: agrupa datos por frecuencia temporal (mensual en este caso).

---

## Aclaraciones frecuentes

### Diferencia entre `map()` y `replace()`

Ambas funciones permiten modificar valores en una columna, pero su comportamiento difiere:

- `map()` reemplaza o transforma valores según un diccionario o función. Si el valor no está incluido, se convierte en `NaN`.
- `replace()` cambia los valores indicados y deja intactos los demás, sin introducir `NaN`.

Ejemplo:

```python
df['sexo'].map({'M': 'Masculino', 'F': 'Femenino'})
# → 'X' se convierte en NaN

df['sexo'].replace({'M': 'Masculino', 'F': 'Femenino'})
# → 'X' permanece igual
