# Entrega Final: Ecosistema de Automatización IA Autónomo para Negocios

**Caso:** Roots Garden, vivero y tienda online de plantas de interior en zona norte de Buenos Aires (negocio ficticio).
**Curso:** IA y Automatización – Coderhouse

## ¿Qué hace?

Sistema de atención de consultas por email con IA y aprobación humana:

1. Un cliente escribe un email a Roots Garden.
2. n8n lo detecta, lo filtra y lo guarda en Airtable (cliente + consulta).
3. Gemini lee las directrices de la marca, **clasifica** la consulta (categoría y prioridad) y **redacta un borrador** de respuesta.
4. El dueño recibe un **aviso por email**, revisa la respuesta en Airtable, puede editarla y la aprueba o la rechaza.
5. Si la aprueba, la respuesta se envía al cliente **en el mismo hilo** de su email.

## Herramientas

| Herramienta | Rol |
|---|---|
| **Airtable** | Base de datos: clientes, consultas, estados, registro de errores y directrices de marca |
| **n8n** | Orquestador del flujo y manejo de errores |
| **Google Gemini 3.5 Flash** | Clasificación y redacción (reemplazable por Claude u OpenAI) |
| **Gmail** | Entrada de consultas, aviso al dueño y respuesta al cliente |

## Estados de una consulta

`Pendiente` → `Procesado por IA` → `Aprobado por Humano` → `Enviado`
También puede terminar en `Rechazado` o `Error`.

## Puntos clave

- **Filtro anti-bucle:** se ignoran los emails enviados por la propia casilla del sistema.
- **Filtro de email vacío:** usa `trim()` para que un salto de línea no cuente como texto.
- **Prompt dinámico:** se arma con las directrices de Airtable y los datos de cada cliente.
- **Límite de tokens:** máximo 10.000 tokens en Gemini.
- **Resiliencia:** 5 reintentos en Gemini y salida de error hacia la tabla *Registro de errores*; la consulta queda en estado `Error` sin romper el flujo.
- **Human-in-the-Loop:** nada se envía al cliente sin aprobación del dueño.

## Test de estrés

| # | Prueba | Resultado |
|---|---|---|
| 1 | Venta | Gemini falló ("Service unavailable"): el error se registró y la consulta quedó en `Error` |
| 2 | Reclamo | Clasificado Alta, el dueño editó la respuesta y se envió en el mismo hilo |
| 3 | Cuidado de plantas | Se detectó y corrigió un error de Airtable; el dueño la rechazó y no se envió nada |
| 4 | Email vacío | Se corrigió el filtro y ahora se frena antes de llegar a la IA |
| 5 | Bucle (email del propio sistema) | El filtro anti-bucle lo frenó |

## Archivos del repositorio

| Archivo | Contenido |
|---|---|
| `Entrega_Final_Roots_Garden.docx` | Informe con explicación y capturas del proyecto |
| `Diagrama_Arquitectura_Roots_Garden.pdf` | Diagrama de arquitectura del sistema |
| `atencion_consultas_roots_garden.json` | Workflow de n8n (sin credenciales ni claves) |

## Base de datos (solo lectura)

https://airtable.com/appFyGcatjW84rmKb/shrwN8C6B8mkn1iX9

## Cómo importar el workflow

1. En n8n: **Create Workflow → ⋯ → Import from File** y elegir el `.json`.
2. Conectar las credenciales propias de Gmail, Airtable y Google Gemini.
3. Apuntar los nodos de Airtable a una base con las mismas tablas.

## Mejoras futuras

- Usar las correcciones del dueño como ejemplos en el prompt para que la IA aprenda su estilo.
- Agregar una regla en las directrices para pedir disculpas siempre en los reclamos.
- Pasar a un modelo pago (Claude u OpenAI) para evitar los límites de la capa gratuita.
