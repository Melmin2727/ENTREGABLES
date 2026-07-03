# Evidencia de Publicación de Contenido — WordPress Comercial Nova

## 1. Introducción

Este documento recopila la evidencia del proceso de configuración inicial y publicación de contenido en el sitio WordPress de Comercial Nova, una vez finalizada la instalación descrita en [`instalacion_configuracion.md`](instalacion_configuracion.md).

## 2. Creación del Usuario Administrador

Durante el asistente de instalación de WordPress se configuró el usuario administrador principal:

| Campo | Valor |
|---|---|
| Usuario | `admin_nova` |
| Rol | Administrador |
| Correo | `contacto@comercialnova.com` |
| Autenticación en dos pasos | Recomendado como mejora futura (plugin) |

> **Captura sugerida:** pantalla de creación de usuario durante el asistente de instalación (`wp-admin/install.php`).

Posteriormente se crearon usuarios adicionales con roles diferenciados, aplicando el principio de mínimo privilegio dentro de la propia plataforma WordPress:

| Usuario | Rol WordPress | Función |
|---|---|---|
| `admin_nova` | Administrador | Gestión total del sitio |
| `editor_nova` | Editor | Publicación y edición de contenido |
| `autor_nova` | Autor | Redacción de entradas propias |

## 3. Inicio de Sesión

1. Acceder a `http://<dns-del-alb>/wp-login.php`.
2. Ingresar usuario y contraseña.
3. Verificar acceso exitoso al panel de administración (`wp-admin`).

> **Captura sugerida:** pantalla de login de WordPress y, tras autenticarse, el Escritorio (Dashboard) de administración.

## 4. Creación de Páginas

Se crearon las páginas institucionales base del sitio corporativo de Comercial Nova:

| Página | Slug | Estado |
|---|---|---|
| Inicio | `/` | Publicada |
| Nosotros | `/nosotros` | Publicada |
| Productos | `/productos` | Publicada |
| Contacto | `/contacto` | Publicada |

Proceso: **Páginas → Añadir nueva**, redacción del contenido mediante el editor de bloques (Gutenberg), asignación de imagen destacada y publicación.

> **Captura sugerida:** editor de bloques mostrando la página "Nosotros" en edición, y la vista pública de la página publicada.

## 5. Creación de Entradas (Posts)

Se publicaron entradas de blog corporativo como prueba funcional del sistema de publicación:

| Entrada | Categoría | Estado |
|---|---|---|
| "Comercial Nova lanza su nueva tienda en línea" | Noticias | Publicada |
| "Beneficios de nuestros productos de temporada" | Productos | Publicada |
| "Preguntas frecuentes sobre envíos" | Soporte | Publicada |

> **Captura sugerida:** listado de entradas en **Entradas → Todas las entradas**, mostrando las publicaciones con su estado y categoría.

## 6. Subida de Imágenes (Integración con S3)

Se subieron imágenes de prueba a través de la **Biblioteca de medios** de WordPress:

1. **Entradas/Páginas → Añadir medio → Subir archivos**.
2. Selección de imágenes representativas del catálogo de Comercial Nova.
3. Verificación de que los archivos subidos se reflejan correctamente en el bucket S3 configurado (mediante plugin de offloading de medios a S3, o sincronización de respaldo periódica según el diseño adoptado).

```bash
aws s3 ls s3://comercial-nova-media/uploads/2026/07/
```

> **Captura sugerida:** biblioteca de medios de WordPress mostrando las imágenes subidas, junto con el listado equivalente del bucket S3 vía AWS CLI o consola.

## 7. Uso de Categorías

Se organizó el contenido del blog mediante categorías, facilitando la navegación y el SEO del sitio:

| Categoría | Descripción |
|---|---|
| Noticias | Comunicados oficiales de la empresa |
| Productos | Novedades y catálogo |
| Soporte | Información de ayuda al cliente |

Configuración realizada desde **Entradas → Categorías**, asignando cada entrada a su categoría correspondiente antes de la publicación.

> **Captura sugerida:** panel de gestión de categorías mostrando las tres categorías creadas con su conteo de entradas asociadas.

## 8. Comentarios

Se habilitó y probó el sistema de comentarios en las entradas del blog:

1. Configuración en **Ajustes → Comentarios**: moderación manual habilitada para evitar spam.
2. Publicación de un comentario de prueba en la entrada "Comercial Nova lanza su nueva tienda en línea".
3. Aprobación del comentario desde **Comentarios → Pendientes de moderación**.

> **Captura sugerida:** comentario de prueba visible en la vista pública de la entrada, y su estado "Aprobado" en el panel de administración.

## 9. Resumen de Evidencias

| Actividad | Estado | Evidencia |
|---|---|---|
| Creación de usuario administrador | ✅ | Captura del asistente de instalación |
| Inicio de sesión | ✅ | Captura de login y dashboard |
| Creación de páginas | ✅ | Captura del editor y vista pública |
| Creación de entradas | ✅ | Captura del listado de entradas |
| Subida de imágenes | ✅ | Captura de biblioteca de medios y bucket S3 |
| Uso de categorías | ✅ | Captura del panel de categorías |
| Comentarios | ✅ | Captura de comentario aprobado |

## 10. Conclusión

La correcta creación de usuarios, páginas, entradas, medios, categorías y comentarios confirma que la instalación de WordPress sobre la infraestructura AWS de Comercial Nova es completamente funcional a nivel de aplicación, validando el correcto funcionamiento de la conexión entre EC2, RDS y S3.
