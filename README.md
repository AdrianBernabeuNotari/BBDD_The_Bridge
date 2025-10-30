# 🎓 Proyecto BBDD The Bridge

Sistema de gestión de base de datos para el seguimiento académico de estudiantes y profesores del bootcamp The Bridge.

## 📋 Descripción

Este proyecto implementa una base de datos relacional completa para gestionar la información académica de un bootcamp, incluyendo alumnos, profesores, clases, promociones y expedientes académicos. La base de datos está desplegada en Render y se gestiona mediante PostgreSQL.

## 🗂 Estructura de la Base de Datos

### Tablas Principales

- *Alumnos*: Información básica de los estudiantes (nombre, email)
- *Profesores*: Datos del claustro docente
- *Campus*: Ubicaciones físicas (Madrid, Valencia, etc.)
- *Vertical*: Especialidades (Data Science, Full Stack)
- *Promo*: Promociones por fecha de inicio
- *Clases*: Combinación de vertical, campus y promoción
- *Matriculas*: Relación entre alumnos y clases
- *Asignaciones*: Relación entre profesores y clases (incluye rol y modalidad)
- *Expediente*: Tabla denormalizada con toda la información académica

### Diagrama de Relaciones


Alumnos ─┐  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├─→ Matriculas ←─ Clases ─┬─→ Vertical  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;├─→ Campus  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;│&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;└─→ Promo  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;│  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;└─→ Expediente ←───────────┘  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;↑  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;│  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Profesores ─→ Asignaciones  


## 🚀 Instalación y Configuración

### Requisitos Previos

- Python 3.8+
- PostgreSQL 12+
- Cuenta en Render (para deployment)

### Librerías Necesarias

bash
pip install pandas psycopg2-binary jupyter


### Configuración de la Base de Datos

1. *Crear la base de datos en Render* o en tu servidor PostgreSQL local

2. *Ejecutar el script de creación de tablas* (disponible en Data/Bootcamp.sql)

3. *Configurar la conexión* en el notebook:

python
connection = psycopg2.connect(
    host='tu-host.render.com',
    database='tu_database',
    user='tu_usuario',
    password='tu_password',
    port=5432
)


## 📊 Datos de Entrada

El proyecto utiliza los siguientes archivos CSV ubicados en la carpeta Data/:

- Clase_1.csv - Alumnos de la primera clase
- Clase_2.csv - Alumnos de la segunda clase
- Clase_3.csv - Alumnos de la tercera clase
- Clase_4.csv - Alumnos de la cuarta clase
- Claustro.csv - Información del profesorado

### Formato de los CSV

*Estructura de Clase_X.csv:*

Nombre, Email, Promoción, Fecha_comienzo, Campus, Proyecto_HLF, Proyecto_EDA, ...


*Estructura de Claustro.csv:*

Nombre, Vertical, Campus, Promocion, Rol, Modalidad


## 🔄 Proceso de Carga de Datos

### 1. Importación de CSV
python
df_alumnos_1 = pd.read_csv('Data/Clase_1.csv', sep=";")
df_profesores = pd.read_csv('Data/Claustro.csv', sep=";")


### 2. Limpieza y Transformación
- Concatenación de todos los DataFrames de alumnos
- Normalización de nombres de verticales (DS → Data Science, FS → Full Stack)

### 3. Inserción en Base de Datos
El proceso sigue este orden:
1. Alumnos (nombre, email)
2. Profesores (nombre)
3. Asignaciones (relaciona profesores con clases mediante joins)
4. Expediente (tabla denormalizada con toda la información)

## 🛠 Uso del Notebook

### Ejecución Completa

bash
jupyter notebook Notebook.ipynb


### Ejecutar por Secciones

1. *Importar librerías*: Carga todas las dependencias necesarias
2. *Importar y limpiar CSV*: Lee y prepara los datos
3. *Conectar a la base de datos*: Establece la conexión con PostgreSQL
4. *Rellenar tablas*: Inserta los datos en el orden correcto

### Manejo de Errores

Si encuentras algún error durante la ejecución:

python
# Ejecutar rollback para deshacer cambios
connection.rollback()


## 📈 Características Destacadas

### Sistema de Asignaciones Inteligente
El código utiliza joins complejos para relacionar automáticamente profesores con sus clases basándose en:
- Vertical (Data Science / Full Stack)
- Campus (Madrid, Valencia, etc.)
- Promoción (Septiembre, Enero, etc.)

python
cursor.execute('''
    SELECT "ID_Clase"
    FROM "Clases"
    JOIN "Vertical" ON "Clases"."ID_Vertical" = "Vertical"."ID_Vertical"
    JOIN "Campus" ON "Clases"."ID_Campus" = "Campus"."ID_Campus"
    JOIN "Promo" ON "Clases"."ID_Promo" = "Promo"."ID_Promo"
    WHERE "Vertical"."Vertical" = %s 
      AND "Campus"."Ciudad" = %s 
      AND "Promo"."Promocion" = %s
''', (vertical, campus, promocion))


### Tabla de Expedientes
Tabla denormalizada que consolida toda la información para consultas rápidas:
- Datos del alumno
- Información de la clase
- Profesor asignado
- Notas de proyectos (HLF, Web_Dev, Data_Science)
- Observaciones

## 🔍 Validaciones Implementadas

### Verificación de Clases Existentes
El notebook incluye una validación para detectar combinaciones de profesor-clase inexistentes:

python
cursor.execute('''
    SELECT COUNT(*) FROM "Clases" c
    JOIN "Vertical" v ON c."ID_Vertical" = v."ID_Vertical"
    JOIN "Campus" ca ON c."ID_Campus" = ca."ID_Campus"
    JOIN "Promo" p ON c."ID_Promo" = p."ID_Promo"
    WHERE v."Vertical"=%s AND ca."Ciudad"=%s AND p."Promocion"=%s
''', (row['Vertical'], row['Campus'], row['Promocion']))


## 🌐 Deployment en Render

### Configuración del Servidor

Host: dpg-d41069hr0fns739r6gq0-a.oregon-postgres.render.com
Database: bootcamp_z1q7
User: bootcamp_z1q7_user
Port: 5432


### Herramientas de Administración
- *pgAdmin 4*: Para gestión visual de la base de datos
- *Render Dashboard*: Para monitoreo y configuración del servidor

## 📝 Notas Importantes

### Gestión de Transacciones
- Todas las inserciones usan connection.commit() para confirmar cambios
- En caso de error, usar connection.rollback() para deshacer cambios

### Codificación de Caracteres
Los CSV utilizan separador ; y pueden contener caracteres especiales. Asegúrate de que la codificación sea correcta al importar.

### Normalización de Datos
Las verticales se normalizan automáticamente:
- DS → Data Science
- FS → Full Stack

## 🔮 Próximas Mejoras

- [ ] Implementar sistema de try-except para manejo automático de errores
- [ ] Añadir validaciones de integridad referencial antes de inserciones
- [ ] Crear vistas SQL para consultas frecuentes
- [ ] Implementar sistema de respaldo automático
- [ ] Añadir logging de operaciones
- [ ] Desarrollar API REST para acceso a los datos
- [ ] Implementar sistema de roles y permisos

## 👥 Contribuciones

Para contribuir al proyecto:

1. Fork el repositorio
2. Crea una rama para tu feature (git checkout -b feature/AmazingFeature)
3. Commit tus cambios (git commit -m 'Add some AmazingFeature')
4. Push a la rama (git push origin feature/AmazingFeature)
5. Abre un Pull Request

## 📄 Licencia

Este proyecto es parte del bootcamp de The Bridge y está destinado a fines educativos.

## 🤝 Soporte

Si encuentras algún problema o tienes sugerencias:
- Abre un issue en GitHub
- Contacta con el equipo docente de The Bridge

---

*Desarrollado con* ❤ *para The Bridge Bootcamp*
