# JobHunter AI — arquitectura y hoja de ruta

**Estado:** diseño inicial, 4 de noviembre de 2026  
**Repositorio:** `in0cybe/jobhunter-ai`

## 1. Objetivo

Construir un asistente personal para organizar y mejorar candidaturas a puestos remotos que acepten candidatos desde España, con foco inicial en desarrollo PHP/Symfony, JavaScript, backend, APIs y Full Stack.

La aplicación complementa el agente de búsqueda que ya utiliza el usuario en Claude Pro. No debe duplicar ese buscador ni interferir con sus automatizaciones existentes.

## 2. Reparto de responsabilidades

- **Agente existente de Claude Pro:** continúa localizando ofertas en los portales que ya tiene configurados y mantiene únicamente las acciones de subida de CV que el usuario ya autorizó.
- **JobHunter AI:** recibe ofertas y resultados del agente, los normaliza, elimina duplicados, conserva la trazabilidad y organiza el proceso.
- **ChatGPT/Codex:** ayuda a desarrollar el producto; en el flujo de candidaturas puede analizar ofertas, explicar el encaje, detectar señales de fraude, proponer palabras clave y preparar borradores de CV y cartas.
- **Usuario:** decide qué candidaturas seguir y revisa/aprueba cualquier envío o acción irreversible.

**Límite inicial:** JobHunter AI no inicia sesión en portales ni envía candidaturas automáticamente. Tampoco vuelve a subir un CV que ya haya gestionado el agente existente. Cualquier automatización posterior necesitará autorización explícita y controles por acción.

## 3. Arquitectura propuesta

Empezar con un **monolito modular**, sencillo de mantener y desplegar, en lugar de microservicios.

- **Backend:** Symfony (versión LTS soportada al iniciar el desarrollo), PHP y Doctrine.
- **Interfaz:** Vue 3.
- **Base de datos:** PostgreSQL.
- **Procesos en segundo plano:** Symfony Messenger para importaciones, análisis y recordatorios.
- **Entorno local:** Docker Compose.
- **Almacenamiento documental:** directorio privado fuera del repositorio, con rutas y permisos controlados.
- **IA:** adaptadores independientes para Anthropic y OpenAI. La aplicación debe funcionar en modo manual/sin API y no llamar a modelos externos hasta configurar credenciales, límites de gasto y consentimiento.

### Flujo de datos

1. Claude Pro localiza ofertas con su configuración actual.
2. El usuario o una exportación autorizada entrega los resultados a JobHunter AI en CSV/JSON.
3. El importador valida campos y normaliza empresa, puesto, ubicación, modalidad, URL y fecha.
4. El sistema detecta duplicados por URL canónica y por huella de empresa + puesto.
5. El usuario revisa la oferta y solicita un análisis.
6. El motor de IA devuelve una evaluación estructurada, con razones y evidencias; nunca inventa experiencia ni métricas.
7. JobHunter prepara, si se solicita, borradores adaptados a la oferta.
8. El usuario revisa y presenta la candidatura en el portal.
9. JobHunter registra el estado, la versión documental utilizada y los siguientes pasos.

## 4. Módulos y entidades iniciales

- **Oferta:** empresa, título, descripción, URL, fuente, fecha detectada, ubicación, modalidad remota, salario y estado de revisión.
- **Empresa:** nombre normalizado, dominio y notas.
- **Importación:** origen, fecha, fichero, filas procesadas, errores y duplicados.
- **Evaluación:** puntuación orientativa, criterios cumplidos, carencias, palabras clave y señales de riesgo.
- **Versión de CV:** identificador, idioma, especialidad, fecha y ubicación privada del fichero; nunca el contenido del CV dentro de Git.
- **Candidatura:** oferta asociada, estado, fecha, canal, versión de CV y registro de acciones.
- **Seguimiento:** fecha prevista, tipo de recordatorio, resultado y notas.
- **Borrador:** carta, respuestas a preguntas y versión adaptada del CV, siempre pendiente de revisión.

## 5. Criterios de evaluación de ofertas

La puntuación debe ser explicable y configurable. Como primera propuesta:

- Compatibilidad técnica y experiencia demostrable.
- Aceptación explícita de candidatos residentes en España.
- Modalidad realmente remota y requisitos de zona horaria.
- Idioma requerido y nivel que puede acreditarse.
- Seniority y responsabilidades.
- Salario, contrato y condiciones, cuando estén publicados.
- Calidad de la fuente y señales de fraude.

La puntuación no sustituye la decisión humana. Las señales de riesgo deben mostrarse por separado y explicar qué dato las provoca.

## 6. Privacidad y seguridad

- Este repositorio es **público**: no subir CV, teléfono, correo, direcciones, cartas, datos de candidaturas, cookies, tokens ni claves.
- Mantener secretos en variables de entorno locales; incluir solo un `.env.example` con nombres de variables y valores ficticios.
- Añadir CV y documentos a almacenamiento privado y al `.gitignore`.
- No guardar contraseñas de portales.
- Limitar el acceso a los datos personales y permitir exportación y borrado.
- No enviar datos a Anthropic u OpenAI hasta que el usuario haya configurado el proveedor y conozca el coste.
- Registrar qué modelo/proveedor procesó cada evaluación y cuándo.
- Toda acción de envío, subida o modificación en un portal requiere revisión y aprobación explícitas.

## 7. Hoja de ruta por fases

### Fase 0 — definición y revisión del punto de partida
- [x] Localizar el repositorio y comprobar su estado.
- [x] Confirmar que el repositorio parte prácticamente vacío.
- [x] Documentar la arquitectura y las reglas de seguridad.
- [ ] Confirmar el formato real de exportación del agente de Claude.
- [ ] Verificar las versiones de PHP, Symfony, Node y Docker disponibles en el equipo de desarrollo.

### Fase 1 — base funcional sin coste de IA
- Crear el esqueleto Symfony + Vue.
- Configurar PostgreSQL y Docker Compose.
- Crear migraciones y entidades.
- Implementar importación CSV/JSON con validación.
- Crear listado, búsqueda, filtros y ficha de oferta.
- Probar con datos ficticios, sin subir CV ni información personal al repositorio.

**Criterio de salida:** importar un lote de ofertas, detectar duplicados y revisarlas desde el panel.

### Fase 2 — análisis asistido
- Definir un esquema JSON estable para las evaluaciones.
- Implementar puntuación explicable, palabras clave, encaje y señales de riesgo.
- Añadir adaptadores de proveedor detrás de una interfaz común.
- Mantener un modo simulado para desarrollo y pruebas.

**Criterio de salida:** analizar una oferta de prueba y obtener un resultado reproducible, justificable y editable.

### Fase 3 — documentos de candidatura
- Seleccionar una versión de CV por oferta.
- Preparar borradores de CV y carta en español o inglés.
- Verificar que cada afirmación procede de la información profesional confirmada.
- No inventar métricas, tecnologías, niveles de idioma ni responsabilidades.

**Criterio de salida:** generar un paquete de candidatura revisable, sin enviarlo.

### Fase 4 — seguimiento
- Registrar candidatura, estado, fechas y versión documental.
- Añadir recordatorios de seguimiento y entrevistas.
- Incorporar estadísticas básicas.

### Fase 5 — integraciones controladas
- Automatizar la importación desde el formato que realmente produzca el agente de Claude.
- Valorar APIs de Anthropic/OpenAI solo después de fijar presupuesto y límites.
- Añadir integraciones con portales únicamente cuando exista un método permitido y estable.
- Mantener aprobación humana antes de cualquier envío.

## 8. Decisiones pendientes antes de programar la Fase 1

1. Confirmar que Symfony + Vue 3 sigue siendo la opción preferida.
2. Confirmar el formato de salida disponible del agente Claude Pro (CSV, JSON, hoja de cálculo o correo).
3. Comprobar el entorno de desarrollo del PC Windows y decidir si el Lenovo Linux se usará solo para tareas auxiliares.
4. Establecer un límite de gasto de 0 € para llamadas de IA hasta que el usuario decida lo contrario.

## 9. Nota sobre costes de proveedores

Las suscripciones de ChatGPT y Claude para sus interfaces de conversación no equivalen a saldo de API. Las llamadas de la aplicación a OpenAI o Anthropic deben tratarse como servicios con facturación separada, salvo que la cuenta muestre expresamente algún crédito promocional aplicable. Codex iniciado con la cuenta de ChatGPT utiliza los límites de Codex del plan; no es lo mismo que llamar a la API de OpenAI desde el backend.
