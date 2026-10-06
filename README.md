# RandoHub (Architecture & Overview)

*Note: The main source code for this project is currently in a private repository as it is in active development. This repository serves as a technical showcase of the architecture, database modeling, and problem-solving approach.*

## ¿Qué es RandoHub?
RandoHub (también conocido conceptualmente como Rando Atlas) es una aplicación web full-stack desarrollada en **Django** diseñada para gestionar la compleja logística del ecosistema del ciclismo de ultra-resistencia (Randonneurs). 
> Plataforma neutral, no oficial y para la agregación y visualización de datos del ciclismo Randonneur de ultra-distancia.

La plataforma digitaliza y centraliza la relación entre Organizaciones, Clubes ciclistas, Participantes, Resultados y los Eventos (Brevets/LRMs), sustituyendo procesos manuales por un sistema de gestión relacional robusto.

**RandoHub** es un proyecto de Trabajo Fin de Máster (TFM) que nace para solucionar la fragmentación histórica de los datos en el ciclismo de larga distancia no competitivo. La plataforma funciona como un archivo histórico público que centraliza los resultados y palmarés de ciclistas, a menudo dispersos en múltiples formatos y fuentes, respetando siempre la autoridad de las entidades oficiales como el Audax Club Parisien (ACP) y Les Randonneurs Mondiaux (LRM).

## Objetivos del Proyecto

-   **Centralización de Datos:** Unificar resultados de múltiples fuentes (webs nacionales, hojas de cálculo de retos) en una única base de datos relacional y estandarizada.
-   **Preservación del Historial:** Crear un palmarés consolidado para cada ciclista, resolviendo duplicidades mediante un pipeline de deduplicación de entidades.
-   **Comunidad y Privacidad (GDPR):** Ofrecer a los usuarios la capacidad de registrarse, reclamar sus perfiles históricos y gestionar su nivel de privacidad desde el diseño.
-   **Herramientas de Análisis:** Proveer funcionalidades de valor añadido, como el cálculo dinámico del progreso para la obtención de reconocimientos anuales e históricos.


## Características y Motores Implementados

El proyecto se ha desarrollado en fases incrementales, resultando en un conjunto de características robustas:

#### Fase A: MVP Nacional e Ingesta Core
-   **Directorio de Clubes Reales**: Base de datos unificada con 202 clubes oficiales de España indexados por códigos ACP.
-   **Blindaje e Integridad de Datos (Motor "Data Armor")**:
    -   Cálculo automático de estatus (*Finisher* vs *Over Time*) según la normativa de tiempos límites oficiales de ACP/LRM en el método `Result.save()`.
    -   Flexibilidad para ultra-distancias mediante un sistema de sobrescritura de límites de tiempo (`max_time_override`).

#### Fase B: Identidad, Cuentas y Privacidad GDPR
-   **Desacoplamiento de Identidad**: Separación arquitectónica entre la cuenta de usuario (`User`) y el perfil histórico del ciclista (`Randonneur`).
-   **Niveles de Privacidad por Diseño**: Tres niveles (`Público`, `Comunidad`, `Privado`) blindados a nivel de backend en las vistas.
-   **Flujo de Reclamación Voluntaria (Claiming Flow)**: Mecanismo seguro para que un usuario se asocie a su historial randonneur mediante consentimiento explícito.
-   **Sistema de Solicitud de Fusión (Merge Requests)**: Permite que la comunidad proponga fusiones de perfiles duplicados para revisión y ejecución por parte de un administrador.

#### Fase C: Automatización y Sincronización Avanzada
-   **Ingesta de Hojas de Cálculo Dinámica**: Comando de importación que procesa datos desde archivos locales o URLs de Google Sheets.
-   **Motor de Resolución de Entidades (Deduplicación)**: Script que limpia caracteres corruptos, normaliza nombres y los compara mediante conjuntos de tokens (*Token Set Matching*), resolviendo el 95% de las duplicidades.
-   **Calculador de Progreso en Tiempo Real**: Módulo visual que indica al ciclista en su perfil qué pruebas le faltan para obtener el título "Super Randonneur" en el año en curso.
-   **Sincronización de Logros**: Algoritmos que calculan y otorgan automáticamente reconocimientos acumulativos como los retos "10x, 20x, 30x SR Challenge".


## Arquitectura y Patrones de Diseño

-   **Patrón MTV (Model-Template-View):** Se sigue la arquitectura de Django con un desacoplamiento estricto de responsabilidades. La lógica de negocio y la integridad de los datos residen en el **Modelo**; la orquestación del flujo de datos en la **Vista**; y la presentación en las **Plantillas**.
-   **Modelo Relacional Normalizado (3FN):** El dominio está estructurado para erradicar redundancias e inconsistencias, asegurando la integridad referencial durante la ingesta de datos masivos.
-   **Principio DRY (Don't Repeat Yourself):** Se utiliza el patrón `TimeStampedModel` (una clase base abstracta) para la auditoría de marcas temporales, evitando la duplicidad de código en múltiples modelos.

---

## Stack Tecnológico

| Componente      | Tecnología Utilizada                                     | Justificación                                                                                                                               |
| --------------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **Backend**     | [Python 3.12](https://www.python.org/)                   | Ecosistema maduro para desarrollo web, sintaxis clara y amplia disponibilidad de librerías.                                                   |
| **Framework**   | [Django 6.1](https://www.djangoproject.com/)              | Filosofía "baterías incluidas", ORM potente que abstrae la base de datos, y robustas medidas de seguridad integradas.                     |
| **Base de Datos** | [PostgreSQL](https://www.postgresql.org/)                | Se migró de SQLite (prototipado) a PostgreSQL por ser un motor de producción robusto, transaccional (ACID) y el estándar para Django. |
| **Frontend**    | HTML5, CSS3 (Plantillas de Django)                       | Se utilizó el sistema de plantillas nativo, priorizando la lógica de backend sobre un framework de frontend complejo (React/Vue).    |
| **Dependencias**| `django-countries`, `python-decouple`, etc.              | Ver `requirements.txt` para el listado completo.                                                                                           |
| **Entorno Dev** | PyCharm, venv, Git                                       | Herramientas estándar para un flujo de desarrollo profesional y control de versiones.                                                        |


## Modelo de Datos y Arquitectura
El corazón de la aplicación es su base de datos relacional. El ORM de Django gestiona entidades con relaciones complejas (One-to-Many y Many-to-Many):
- `Randonneur`: Perfiles de usuarios/ciclistas con su historial y palmarés.
- `Club / Organización`: Entidades que alojan a los ciclistas y organizan pruebas.
- `Event`: Modelado de pruebas homologadas con atributos específicos (distancia, fecha, desnivel, tiempos de corte).

<img width="942" height="518" alt="Screenshot 2026-10-06 at 16 41 47" src="https://github.com/user-attachments/assets/1c8e2f37-c8a9-43ef-b498-c099a39a9276" />


