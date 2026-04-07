# Symmetry News App — Informe del Proyecto
> 📎 **Capturas de pantalla:**git [Google Drive](https://drive.google.com/drive/folders/1RUH9Y1DCyITQG4JHFqew-GBecvyHOGVp?usp=drive_link)

## 1. Introducción

Este proyecto consiste en una aplicación de noticias multiplataforma desarrollada con Flutter, Clean Architecture y BLoC. Nunca había trabajado con Dart ni Flutter antes de esta prueba. Mi contexto es el de un desarrollador junior que partió de cero con estas tecnologías y tuvo que aprender la arquitectura, el lenguaje y el ecosistema en paralelo al desarrollo.

El objetivo: construir una app donde los periodistas puedan consultar noticias diarias de NewsAPI y publicar sus propios artículos con imagen a Firebase (Firestore + Cloud Storage), siguiendo los principios de Clean Architecture con BLoC como patrón de estado.

El resultado final incluye el cumplimiento completo de los requisitos base más 8 funcionalidades extra que describo en la sección de overdelivery.

## 2. Proceso de Aprendizaje

### Tecnologías aprendidas y aplicadas

- **Flutter y Dart** — Framework multiplataforma basado en widgets. Aprendí composición de widgets, gestión de estado y adaptaciones por plataforma.
- **Patrón BLoC** (`flutter_bloc ^9.1.1`) — Gestión de estado basada en eventos. La UI emite Events, el BLoC los procesa y emite States. Separa la lógica de negocio de la interfaz.
- **Clean Architecture** — Separación en tres capas según los principios de Robert C. Martin:
  - **Domain** (Dart puro) — Entidades, interfaces de repositorio y casos de uso sin dependencias externas
  - **Data** — Modelos, data sources (Firebase, REST API, SQLite) e implementaciones de repositorio
  - **Presentation** — Pantallas, widgets y BLoCs que nunca acceden directamente a los data sources
- **Firebase Firestore** — Base de datos NoSQL en la nube para almacenar artículos publicados
- **Firebase Cloud Storage** — Almacenamiento de imágenes en `media/articles/{articleId}/`
- **Retrofit + Dio** — Cliente HTTP tipado para consumir la API REST de NewsAPI con deserialización JSON automática
- **Floor (sqflite)** — ORM para SQLite local, utilizado para guardar artículos offline
- **GetIt** — Service locator para inyección de dependencias entre todas las capas
- **image_picker** — Selección de imagen multiplataforma usando `Uint8List` para compatibilidad web
- **Firebase Auth + Google Sign-In** — Sistema de autenticación completo con email/password y Google OAuth
- **API de OpenAI (GPT-4o-mini)** — Análisis de artículos con IA siguiendo Clean Architecture

### Recursos utilizados

- Documentación oficial de Flutter (flutter.dev)
- Documentación de Firebase para Flutter (firebase.google.com)
- Documentación de la librería BLoC (bloclibrary.dev)
- Principios de Clean Architecture de Robert C. Martin
- Documentación propia del proyecto (`APP_ARCHITECTURE.md`, `CODING_GUIDELINES.md`, `ARCHITECTURE_VIOLATIONS.md`)
- Tutorial de Clean Architecture en Flutter de Reso Coder (recomendado en el README del proyecto)

## 3. Retos encontrados

### 3.1 Compatibilidad multiplataforma (Web + Mobile)

**Problema:** `dart:io File` no está disponible en Flutter Web. Esto rompía todo el flujo de subida de imágenes — desde el widget picker hasta el upload a Firebase Storage.

**Solución:** Refactoricé el pipeline completo de thumbnails en todas las capas para usar `Uint8List` (bytes crudos) + `String fileName` en lugar de objetos `File`:
- `PublishArticleParams` → `thumbnailBytes: Uint8List` + `thumbnailFileName: String`
- `ThumbnailPickerWidget` → `MemoryImage(bytes)` en vez de `FileImage(file)`
- Firebase Storage → `putData(bytes)` en vez de `putFile(file)`

**Conclusión:** Los contratos de datos deben ser agnósticos a la plataforma desde el inicio. Usar bytes en lugar de File hace que el código sea multiplataforma sin imports condicionales.

### 3.2 CORS en Firebase Storage

**Problema:** Las imágenes subidas a Firebase Storage se cargaban en móvil pero aparecían rotas en web por restricciones CORS del navegador.

**Solución:** Configuración de política CORS en el bucket de Storage:
```bash
gsutil cors set cors.json gs://symmetry-news-app-5dcc3.firebasestorage.app
```

### 3.3 CORS en NewsAPI

**Problema:** NewsAPI bloquea peticiones directas desde el navegador.

**Solución:** Interceptor de Dio que redirige las peticiones a través de un proxy CORS (`corsproxy.io`) solo cuando `kIsWeb` es true. Las plataformas nativas no se ven afectadas.

### 3.4 Base de datos local en Web

**Problema:** sqflite/Floor falla en Flutter Web porque depende de bindings nativos de SQLite.

**Solución:** Clases fallback `_NoOpAppDatabase` y `_NoOpArticleDao` que devuelven resultados vacíos en web. La funcionalidad de artículos guardados se degrada de forma controlada sin afectar al resto de la app.

### 3.5 Code Signing en Xcode 26 (macOS/iOS)

**Problema:** Xcode 26 beta añade un atributo extendido `com.apple.provenance` que hace fallar la firma de código de los frameworks de Flutter.

**Solución:** Tras investigación exhaustiva (parcheado del SDK de Flutter, limpieza de xattrs, pruebas con `CODE_SIGNING_ALLOWED=NO`), se confirmó como un bug no resuelto de Apple beta. La app se ejecuta en web y Android mientras el código permanece preparado para iOS/macOS cuando Apple publique la corrección.

## 4. Reflexión y Direcciones Futuras

### Aprendizajes técnicos

- **Disciplina de Clean Architecture** — La separación estricta por capas resulta liosa al principio, pero se amortiza al añadir nuevas funcionalidades. Añadir edit/delete requirió cambios mínimos en el código existente porque cada capa tiene una única responsabilidad.

- **Predictibilidad de BLoC** — El flujo Event → State hace que el comportamiento sea determinista. Cada transición de estado es explícita, lo que facilita el debugging.

- **Pensamiento multiplataforma** — Desarrollar simultáneamente para web, móvil y desktop obliga a diseñar abstracciones correctas desde el inicio.

- **Firebase como backend rápido** — Con Firestore, Storage y security rules se obtiene un backend funcional sin escribir código de servidor.

### Crecimiento profesional

- Navegar codebases y patrones arquitectónicos desconocidos con rapidez
- Tomar decisiones pragmáticas ante bloqueos (bug de Xcode 26 → pivot a web/Android)
- Seguir documentación con rigor sin renunciar a la innovación
- Construir funcionalidades end-to-end a través de todas las capas arquitectónicas

### Mejoras futuras

- **Paginación** — Scroll infinito usando cursores de Firestore
- **Offline-first** — Persistencia local de Firestore para lectura sin internet
- **Tests** — Tests unitarios para BLoCs (transiciones event/state), use cases y repositorios. Tests de widgets para los flujos críticos de UI
- **Categorías de artículos** — Filtrado por tema usando parámetros de NewsAPI
- **Push Notifications** — Alertas de nuevos artículos vía Firebase Cloud Messaging
- **Control de acceso por roles** — Roles admin/periodista mediante custom claims de Firestore

## 5. Prueba del Proyecto

### Cómo ejecutar

1. Clonar el repositorio:
   ```bash
   git clone https://github.com/Guillegas/SymmetryPrueba.git
   ```
2. Navegar al frontend:
   ```bash
   cd starter-project/frontend
   ```
3. Instalar dependencias:
   ```bash
   flutter pub get
   ```
4. Crear el archivo de API keys (necesario para la funcionalidad de IA):
   ```bash
   # Crear lib/config/api_keys.dart con:
   class ApiKeys {
     static const openAiKey = 'TU_OPENAI_API_KEY';
   }
   ```
5. Ejecutar en Chrome:
   ```bash
   flutter run -d chrome
   ```
6. Ejecutar en Android (emulador o dispositivo):
   ```bash
   flutter run -d android
   ```

### Flujos principales a probar

1. **Pantalla de login** — Registro con email/password o Google Sign-In
2. **Feedback de bienvenida** — Snackbar verde con "Welcome, [nombre]" tras login exitoso
3. **Pantalla principal** — Carga artículos de NewsAPI + artículos propios de Firestore
4. **Barra de búsqueda** — Filtrado en tiempo real por título, descripción o autor
5. **Pull to refresh** — Recarga del feed completo
6. **Botón (+)** → Formulario de creación con validación (autor auto-rellenado desde la cuenta)
7. **Publicar** → Sube imagen a Storage + crea documento en Firestore
8. **Artículos publicados** — Aparecen en la parte superior del feed
9. **Detalle de artículo** — Vista completa con imagen, título, fecha relativa y contenido
10. **Asistente IA (icono ✨)** — Bottom sheet con Resumir, Sugerir Titular y Análisis de Sentimiento
11. **Artículos propios** — Menú (⋮) con opciones de Editar y Eliminar
12. **Compartir** — Botón de compartir con share sheet nativo (Android) o diálogo (web)
13. **Guardar artículo** — Bookmark que almacena en SQLite local (funciona offline)
14. **Dark mode** — Toggle sol/luna en la barra superior
15. **Artículos guardados** — Icono en la barra superior abre la lista de guardados
16. **Logout** — Icono de cierre de sesión con diálogo de confirmación

## 6. Overdelivery

### 6.1 Funcionalidades adicionales implementadas

Además de los requisitos base (consultar NewsAPI + publicar artículos), implementé **8 funcionalidades extra**:

#### 🔍 Búsqueda en tiempo real
- **Descripción:** Barra de búsqueda en la pantalla principal que filtra todos los artículos (NewsAPI y propios) mientras el usuario escribe.
- **Implementación:** Filtrado client-side por título, descripción y autor. Muestra mensaje "No articles found" cuando no hay coincidencias.
- **Justificación:** Funcionalidad básica de UX en cualquier app de noticias.
- **Ubicación:** `lib/features/daily_news/presentation/pages/home/daily_news.dart`

#### 🔄 Pull to Refresh
- **Descripción:** Gesto de arrastrar hacia abajo para recargar artículos de ambas fuentes.
- **Implementación:** `RefreshIndicator` de Flutter que dispara `GetArticles` en el `RemoteArticlesBloc`. También se auto-refresca al volver de la pantalla de crear artículo.
- **Justificación:** Los usuarios esperan poder actualizar el contenido manualmente.
- **Ubicación:** `lib/features/daily_news/presentation/pages/home/daily_news.dart`

#### ✏️ CRUD completo (Editar y Eliminar artículos propios)
- **Descripción:** Los periodistas pueden editar título/contenido de sus artículos o eliminarlos por completo (incluyendo la imagen de Firebase Storage).
- **Implementación:** Flujo arquitectónico completo siguiendo Clean Architecture:
  - **Domain:** `DeleteArticleUseCase`, `UpdateArticleUseCase` con sus params (Equatable)
  - **Data:** Métodos `deleteArticle()` y `updateArticle()` en `ArticlePublisherDataSource`
  - **Presentation:** `EditArticleScreen`, eventos `DeleteArticleEvent`/`UpdateArticleEvent` y estados `ArticlePublisherDeleted`/`ArticlePublisherUpdated`
  - Los artículos propios se identifican por el campo `firestoreId` en `ArticleEntity`
  - La vista de detalle muestra menú (⋮) con Editar/Eliminar solo para artículos propios
  - Eliminar muestra diálogo de confirmación
- **Justificación:** Una plataforma de publicación sin edición/eliminación está incompleta. Demuestra el dominio del patrón Clean Architecture al añadir una funcionalidad completa en todas las capas.
- **Ubicación:**
  - `lib/features/article_publisher/domain/use_cases/delete_article_usecase.dart`
  - `lib/features/article_publisher/domain/use_cases/update_article_usecase.dart`
  - `lib/features/article_publisher/presentation/screens/edit_article_screen.dart`

#### 📤 Compartir artículo
- **Descripción:** Botón de compartir en la pantalla de detalle que usa el share sheet nativo de Android o diálogo en web.
- **Implementación:** Paquete `share_plus`. Compone un mensaje con título, descripción y URL del artículo.
- **Justificación:** Funcionalidad estándar en apps de noticias.
- **Ubicación:** `lib/features/daily_news/presentation/pages/article_detail/article_detail.dart`

#### 🌙 Dark Mode
- **Descripción:** Toggle entre tema claro y oscuro con un toque en el icono sol/luna de la barra superior.
- **Implementación:** `ThemeCubit` (Cubit ligero) que gestiona `ThemeMode`. Definidos `lightTheme()` y `darkTheme()` con esquemas de color completos. `MaterialApp` reacciona a cambios de tema via `BlocBuilder`.
- **Justificación:** Funcionalidad estándar en apps modernas. Mejora la legibilidad en entornos oscuros y reduce consumo de batería en pantallas OLED.
- **Ubicación:**
  - `lib/config/theme/theme_cubit.dart`
  - `lib/config/theme/app_themes.dart`

#### 🔐 Autenticación con Firebase (Email/Password + Google Sign-In)
- **Descripción:** Sistema de autenticación completo que protege la app tras una pantalla de login/registro. Soporta email/password y Google OAuth.
- **Implementación:** Clean Architecture completa:
  - **Domain:** `UserEntity`, `AuthRepository` (abstract), `SignInUseCase`, `SignUpUseCase`, `SignOutUseCase`
  - **Data:** `FirebaseAuthDataSource` (wrapper de `FirebaseAuth` + `GoogleSignIn`), `AuthRepositoryImpl` con mensajes de error descriptivos
  - **Presentation:** `AuthBloc` (escucha `authStateChanges` en tiempo real), `LoginPage` con toggle Sign In/Sign Up, botón de Google y toggle de visibilidad de contraseña
  - `main.dart` muestra login (si no autenticado) o home (si autenticado) usando `BlocBuilder<AuthBloc, AuthState>`
  - Los artículos publicados usan el nombre/email del usuario autenticado como autor
  - Botón de logout con diálogo de confirmación
- **Justificación:** La autenticación es esencial en una plataforma multiusuario. Permite propiedad de artículos, protege la app y personaliza la experiencia.
- **Ubicación:**
  - `lib/features/auth/` (Clean Architecture completa: domain → data → presentation)
  - `lib/main.dart` (routing basado en estado de autenticación)

#### 🤖 Análisis de artículos con IA (OpenAI GPT-4o-mini)
- **Descripción:** Asistente IA accesible desde la pantalla de detalle con tres funciones: resumen del artículo, sugerencia de titular y análisis de sentimiento.
- **Implementación:** Clean Architecture completa:
  - **Domain:** `AiRepository` (abstract), `SummarizeArticleUseCase`, `SuggestHeadlineUseCase`, `AnalyzeSentimentUseCase`
  - **Data:** `AiService` implementa `AiRepository`, llama a la API de Chat de OpenAI via REST (Dio) con el modelo `gpt-4o-mini`
  - **Presentation:** `AiBottomSheet` — panel deslizable con tres botones de acción, estados de carga, resultado y manejo de errores
  - La API key se almacena en archivo gitignored (`lib/config/api_keys.dart`)
  - El icono ✨ aparece en la barra de la pantalla de detalle
- **Justificación:** La IA aporta valor real para periodistas — resúmenes rápidos, sugerencias de titulares y comprensión del tono del artículo. Demuestra integración con APIs externas de IA manteniendo la integridad arquitectónica.
- **Ubicación:**
  - `lib/features/ai/` (Clean Architecture completa: domain → data → presentation)
  - `lib/config/api_keys.dart` (gitignored)

### 6.2 Prototipos y patrones creados

#### Patrón de extensión CRUD
La funcionalidad de Edit/Delete demuestra cómo escala Clean Architecture. Añadir una nueva operación requiere tocar cada capa exactamente una vez:

```
Domain:       UseCase + Params        (Dart puro, sin dependencias)
   ↓
Data:         DataSource + Repository (implementación Firebase)
   ↓
Presentation: Event + State + Screen  (UI + BLoC)
   ↓
DI:           Registrar en GetIt      (injection_container.dart)
```

Este patrón se puede replicar para cualquier nueva funcionalidad (comentarios, likes, perfiles de usuario) sin modificar código existente — siguiendo el Principio Open/Closed.

#### Feed unificado de múltiples fuentes
El `RemoteArticlesBloc` implementa un patrón de feed combinado que fusiona artículos de dos fuentes completamente distintas (REST API + Firestore) en una sola lista:

```dart
final dataState = await _getArticleUseCase();
final publishedState = await _getPublishedArticlesUseCase();

allArticles.insertAll(0, publishedState.data!.map(_mapToArticleEntity));
allArticles.addAll(dataState.data!);
```

El patrón es extensible — se pueden añadir fuentes adicionales (RSS, otras APIs) registrando otro use case y concatenando sus resultados.

### 6.3 Posibles mejoras

1. **Actualizaciones en tiempo real** — Reemplazar `get()` de Firestore por un stream `snapshots()` para que los artículos publicados aparezcan al instante para todos los usuarios.
2. **Compresión de imágenes** — Compresión client-side antes de subir a Storage, y soporte para múltiples imágenes por artículo.
3. **Categorías y tags** — Etiquetado de artículos por categoría con chips de filtro en la pantalla principal.
4. **Arquitectura offline-first** — Persistencia local de Firestore para funcionamiento sin internet.
5. **Suite de tests** — Tests unitarios para todos los BLoCs, use cases y repositorios. Tests de widgets para flujos críticos.
6. **Pipeline CI/CD** — Workflow de GitHub Actions para ejecutar tests, analizar código y generar artefactos APK/web en cada push.

## 7. Secciones adicionales

### Nota sobre compatibilidad web

El proyecto contiene archivos JavaScript autogenerados (en el directorio `web/`) necesarios para que Firebase funcione en el navegador. Estos archivos **no son código fuente de la aplicación** — son archivos de configuración generados por `flutterfire configure`, necesarios para:
- Inicialización del SDK de Firebase en web
- SDKs web de Firebase Auth, Firestore y Storage
- Registro del service worker para soporte PWA

Toda la lógica de la aplicación está escrita **íntegramente en Dart/Flutter**. Los archivos JS son dependencias de infraestructura, equivalentes a `google-services.json` (Android) y `GoogleService-Info.plist` (iOS).

### Arquitectura de inyección de dependencias

`injection_container.dart` es el punto central donde se conectan todas las dependencias:

```
Instancias de Firebase (FirebaseAuth, Firestore, Storage)
    ↓
Data Sources (FirebaseAuthDataSource, NewsApiService, ArticlePublisherDataSourceImpl, AiService)
    ↓
Repositorios (AuthRepositoryImpl, ArticleRepositoryImpl, ArticlePublisherRepositoryImpl)
    ↓
Use Cases (SignIn, SignUp, SignOut, GetArticle, PublishArticle, DeleteArticle, UpdateArticle, Summarize...)
    ↓
BLoCs (AuthBloc, RemoteArticlesBloc, LocalArticleBloc, ArticlePublisherBloc)
```

Cada capa solo depende de la capa superior. Los BLoCs dependen de Use Cases, los Use Cases dependen de interfaces de repositorio (no de implementaciones), y solo el contenedor DI conoce las implementaciones concretas — inversión de control completa.

### Firebase Security Rules

Las reglas desplegadas en Firestore y Storage:
- **Firestore:** Lectura abierta; creación valida todos los campos obligatorios (`id`, `title`, `content`, `author`, `thumbnailUrl`, `thumbnailStoragePath`, `publishedAt`) con verificación de tipos; actualización valida `title` y `content`; eliminación permitida
- **Storage:** Tamaño máximo 5MB, solo tipos de contenido de imagen (`image/*`), organizado en `media/articles/{articleId}/`

### Arquitectura de autenticación

Firebase Auth está integrado con Google Sign-In y email/password, siguiendo el mismo patrón Clean Architecture que el resto de features:
- `AuthBloc` escucha el stream `authStateChanges` para gestión de sesión en tiempo real
- `main.dart` usa `BlocBuilder<AuthBloc, AuthState>` para enrutar entre login y home
- Los artículos publicados usan automáticamente el nombre o email del usuario autenticado como autor
- Las API keys (OpenAI) se almacenan en un archivo gitignored para evitar filtración de credenciales
