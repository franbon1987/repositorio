# Laboratorio: Código Seguro y Modelo OWASP

## 1. Datos generales

- **Nombre completo:** JOSE FRANCISCO BONILLA LOPEZ
- **Carné:** 
- **Correo institucional:**  franbon1987@hotmail.com
- **Asignatura:** Lenguajes de Programación con Aspectos de Seguridad 03-2026
- **Fecha:** 08/11/2026

## 2. Descripción

Este laboratorio consiste en desarrollar una aplicación web utilizando Python, implementando controles básicos de seguridad basados en buenas prácticas y el modelo OWASP. Se aplican mecanismos de validación de entradas, consultas parametrizadas, protección de contraseñas, registro de eventos de autenticación y detección de incidentes, incluyendo el cálculo de las métricas MTTD y MTTR.

## 3. Estructura del proyecto

```text
repositorio/
├── proyecto/
│   ├── app.py
│   ├── crearDB.py
│   ├── detector.py
│   ├── requirements.txt
│   ├── usuarios.db
│   ├── eventos.log
│   ├── respuesta.log
│   └── templates/
│       ├── login.html
│       └── registro.html
└── README.md
```

| Archivo | Función |
|---|---|
| `app.py` | Aplicación Flask: registro, inicio de sesión, validaciones y registro de eventos |
| `crearDB.py` | Crea la base de datos SQLite `usuarios.db` y la tabla `usuarios` |
| `detector.py` | Lee `eventos.log`, detecta intentos fallidos, aplica la respuesta y calcula MTTD y MTTR |
| `eventos.log` | Eventos de autenticación (`LOGIN_CORRECTO` y `LOGIN_FALLIDO`) |
| `respuesta.log` | Acciones de respuesta aplicadas (`USUARIO_BLOQUEADO`) |
| `templates/` | Formularios HTML de registro e inicio de sesión |

## 4. Ejecución del proyecto

**Paso 1.** Clonar el repositorio.

```bash
git clone https://github.com/franbon1987/repositorio.git
```

**Paso 2.** Ingresar al proyecto.

```bash
cd repositorio/proyecto
```

**Paso 3.** Crear y activar el entorno virtual (Windows).

```bash
python -m venv venv
venv\Scripts\activate
```

**Paso 4.** Instalar las dependencias.

```bash
pip install -r requirements.txt
```

**Paso 5.** Crear la base de datos (solo si no existe `usuarios.db`).

```bash
python crearDB.py
```

**Paso 6.** Ejecutar la aplicación.

```bash
python app.py
```

La aplicación queda disponible en `http://127.0.0.1:5000`. Las rutas son `/registro` y `/login`.

**Paso 7.** Ejecutar el detector de incidentes (en otra terminal, dentro de la misma carpeta).

```bash
python detector.py
```

## 5. Controles de seguridad implementados

| Control | Implementación |
|---|---|
| Validación de usuario | Entre 4 y 20 caracteres: letras, números y guion bajo |
| Validación de correo | Formato de correo electrónico válido (expresión regular) |
| Validación de contraseña | Mínimo 8 caracteres |
| Saneamiento de datos | Se eliminan espacios al inicio y al final del usuario y del correo; la contraseña no se modifica |
| Usuarios duplicados | Se rechaza el registro si el usuario o el correo ya existen |
| Consultas parametrizadas | Todas las consultas SQL usan `?` para evitar inyección SQL |
| Protección de contraseñas | Se almacena un hash con `generate_password_hash` y se verifica con `check_password_hash` |
| Registro de eventos | Cada inicio de sesión, correcto o fallido, se guarda en `eventos.log` con fecha, tipo y usuario |
| Protección del log | En el nombre de usuario se reemplazan comas y saltos de línea para que no alteren el formato del log |
| Detección | El detector genera una alerta cuando un usuario acumula 3 o más intentos fallidos |
| Respuesta | Bloqueo simulado del usuario, registrado en `respuesta.log` |

## 6. Resultados de MTTD y MTTR

- **MTTD** (tiempo medio de detección): tiempo entre el primer intento fallido y la detección (tercer intento fallido).
- **MTTR** (tiempo medio de respuesta): tiempo entre la detección y la acción de respuesta.

Se realizaron tres escenarios, cada uno con un usuario distinto y ejecutando `detector.py` justo después de los intentos fallidos.

| Prueba | Usuario | Primer intento | Detección | Respuesta | MTTD | MTTR |
|---|---|---|---|---|---|---|
| 1 | `usuario_03` | 11:04:14 | 11:04:47 | 11:06:58 | 33.0 s | 131.0 s |
| 2 | `usuario_04` | 11:11:10 | 11:11:48 | 11:12:26 | 38.0 s | 38.0 s |
| 3 | `usuario_05` | 11:15:55 | 11:16:28 | 11:17:01 | 33.0 s | 33.0 s |

Todas las fechas corresponden al 2026-11-08.

**Cálculo de los promedios**

- MTTD promedio = (33 + 38 + 33) / 3 = 104 / 3 = **34.67 segundos**
- MTTR promedio = (131 + 38 + 33) / 3 = 202 / 3 = **67.33 segundos**

**Observaciones**

- El MTTD depende del tiempo que transcurre entre los intentos fallidos: a mayor pausa entre intentos, mayor MTTD.
- El MTTR de la prueba 1 es mayor porque pasó más tiempo entre la detección y la ejecución del detector. En este laboratorio la respuesta es simulada y se registra en el momento en que se ejecuta `detector.py`.
- El detector toma el tercer intento fallido como momento de detección; si hubo intentos adicionales, solo aumentan el contador de intentos fallidos.
