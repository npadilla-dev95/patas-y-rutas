# 🏗️ Arquitectura Técnica - Patas y Rutas

## 📋 Índice

1. [Visión General](#visión-general)
2. [Arquitectura del Sistema](#arquitectura-del-sistema)
3. [Componentes Principales](#componentes-principales)
4. [Flujo de Datos](#flujo-de-datos)
5. [APIs Utilizadas](#apis-utilizadas)
6. [Seguridad](#seguridad)
7. [Manejo de Errores](#manejo-de-errores)
8. [Optimizaciones](#optimizaciones)

---

## 🎯 Visión General

**Patas y Rutas** es una aplicación web serverless que combina frontend estático, automatización con n8n, IA generativa y múltiples APIs para crear guías de viaje personalizadas.

### Características Arquitectónicas

- **Serverless**: Sin backend tradicional, todo funciona con webhooks y servicios cloud
- **Event-driven**: Basado en eventos (formulario → webhook → procesamiento → email)
- **Microservicios**: Cada API es independiente y puede fallar sin tumbar el sistema
- **Real-time**: Datos actualizados en tiempo real de múltiples fuentes

---

## 🏛️ Arquitectura del Sistema

### Diagrama de Componentes

```
┌─────────────────────────────────────────────────────────────┐
│                         USUARIO                              │
│                    (Navegador Web)                           │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                    FRONTEND (Netlify)                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │  index.html  │  │  form.html   │  │  styles.css  │      │
│  │ (Landing)    │  │ (Formulario) │  │  main.js     │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└────────────────────────┬────────────────────────────────────┘
                         │ HTTP POST
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                   N8N WORKFLOW (n8n.cloud)                   │
│                                                               │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  1. Webhook Trigger                                   │   │
│  │     ↓                                                 │   │
│  │  2. Procesar Datos Formulario                        │   │
│  │     ↓                                                 │   │
│  │  3. APIs en Paralelo                                 │   │
│  │     ├─→ OpenWeather API    (clima)                   │   │
│  │     ├─→ Geoapify API       (hoteles)                 │   │
│  │     └─→ Geoapify API       (restaurantes)            │   │
│  │     ↓                                                 │   │
│  │  4. Combinar Datos de APIs                           │   │
│  │     ↓                                                 │   │
│  │  5. Claude API - Generar Itinerario                  │   │
│  │     ↓                                                 │   │
│  │  6. Gmail - Enviar Email                             │   │
│  └──────────────────────────────────────────────────────┘   │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│               SERVICIOS EXTERNOS                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │ Anthropic    │  │ OpenWeather  │  │  Geoapify    │      │
│  │ Claude API   │  │     API      │  │     API      │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
                         │
                         ▼
                    ┌─────────┐
                    │  Gmail  │
                    │  SMTP   │
                    └─────────┘
                         │
                         ▼
                   [Usuario recibe email]
```

---

## 🧩 Componentes Principales

### 1. Frontend (Netlify)

**Tecnologías**: HTML5, CSS3, JavaScript vanilla

**Archivos principales**:
- `index.html`: Landing page con propuesta de valor
- `form.html`: Formulario de captura de datos
- `styles.css`: Estilos responsive
- `main.js`: Validación de formulario y envío a webhook

**Responsabilidades**:
- Capturar datos del usuario
- Validar campos antes del envío
- Mostrar mensajes de confirmación/error
- Enviar datos al webhook de n8n vía POST

**Código clave del envío**:
```javascript
// main.js - Envío de datos al webhook
fetch('https://[TU-INSTANCIA].app.n8n.cloud/webhook/patas-rutas', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify(formData)
})
.then(response => response.json())
.then(data => {
    // Mostrar confirmación
    alert('¡Guía generada! Revisa tu email en 1 minuto.');
})
.catch(error => {
    console.error('Error:', error);
    alert('Hubo un error. Por favor, intenta de nuevo.');
});
```

---

### 2. N8N Workflow (Cerebro del Sistema)

**Tecnología**: n8n (automatización low-code/no-code)

**Nodos principales**:

#### 🔹 Nodo 1: Webhook Trigger
- **Tipo**: Webhook
- **Método**: POST
- **Función**: Recibe datos del formulario
- **Output**: JSON con datos del usuario

```json
{
  "nombre": "María",
  "email": "maria@example.com",
  "destino": "Asturias",
  "fecha_inicio": "2025-12-01",
  "fecha_fin": "2025-12-05",
  "tipo_mascota": "perro",
  "nombre_mascota": "Max",
  "tamano_mascota": "mediano",
  "info_mascota": "muy sociable",
  "preferencias": "playas y montaña"
}
```

#### 🔹 Nodo 2: Procesar Datos Formulario
- **Tipo**: Set
- **Función**: Normalizar y estructurar datos
- **Transformaciones**:
  - Convertir fechas a formato ISO
  - Sanitizar inputs (eliminar caracteres especiales)
  - Crear variables para uso posterior

#### 🔹 Nodo 3a: OpenWeather API
- **Tipo**: HTTP Request
- **Endpoint**: `api.openweathermap.org/data/2.5/weather`
- **Parámetros**:
  ```
  q: {{ $json.destino }}
  appid: [API_KEY]
  units: metric
  lang: es
  ```
- **Output esperado**:
  ```json
  {
    "main": {
      "temp": 14.53,
      "feels_like": 13.2,
      "humidity": 72
    },
    "weather": [{
      "description": "nubes"
    }],
    "name": "Asturias"
  }
  ```

#### 🔹 Nodo 3b: Geoapify - Hoteles Pet-Friendly
- **Tipo**: HTTP Request
- **Endpoint**: `api.geoapify.com/v2/places`
- **Parámetros**:
  ```
  categories: accommodation
  filter: circle:[lon],[lat],5000
  limit: 10
  apiKey: [API_KEY]
  ```
- **Filtrado adicional**: Se procesan solo hoteles con "pet" o "dog friendly" en descripción

#### 🔹 Nodo 3c: Geoapify - Restaurantes
- **Tipo**: HTTP Request  
- **Endpoint**: `api.geoapify.com/v2/places`
- **Parámetros**:
  ```
  categories: catering.restaurant
  filter: circle:[lon],[lat],3000
  limit: 10
  apiKey: [API_KEY]
  ```

#### 🔹 Nodo 4: Combinar Datos de APIs
- **Tipo**: Merge
- **Función**: Une todos los outputs en un solo objeto JSON
- **Output**:
  ```json
  {
    "clima": { ... },
    "hoteles": [ ... ],
    "restaurantes": [ ... ],
    "formulario": { ... }
  }
  ```

#### 🔹 Nodo 5: Claude API - Generar Itinerario
- **Tipo**: HTTP Request
- **Endpoint**: `api.anthropic.com/v1/messages`
- **Headers**:
  ```
  x-api-key: [ANTHROPIC_API_KEY]
  anthropic-version: 2023-06-01
  content-type: application/json
  ```
- **Body**:
  ```json
  {
    "model": "claude-sonnet-4-20250514",
    "max_tokens": 4096,
    "messages": [{
      "role": "user",
      "content": "[PROMPT CON TODOS LOS DATOS]"
    }]
  }
  ```

**Prompt Structure**:
```
Eres un experto en viajes pet-friendly de España.

=== DATOS DEL CLIENTE ===
Nombre: {{ nombre }}
Destino: {{ destino }}
Mascota: {{ tipo }} llamada {{ nombre_mascota }}
...

=== DATOS DEL CLIMA ===
Temperatura: {{ temp }}°C
Clima: {{ description }}
...

=== TU TAREA ===
Genera un email en HTML profesional con:
1. Saludo personalizado
2. Sección de clima
3. Hoteles pet-friendly (usa datos reales o inventa basándote en el destino)
4. Restaurantes con terraza
5. Actividades recomendadas
6. Consejos para viajar con la mascota
...
```

#### 🔹 Nodo 6: Gmail - Enviar Email
- **Tipo**: Gmail
- **Operación**: Send Email
- **Configuración**:
  ```
  To: {{ $json.email }}
  From: patasyrutas@gmail.com
  Subject: 🐾 Tu guía personalizada para {{ destino }}
  Body Type: HTML
  Body: {{ $('Claude API').item.json.content[0].text }}
  ```

---

## 🔄 Flujo de Datos Detallado

### Secuencia Temporal

```
T=0s    Usuario envía formulario
        ↓
T=0.1s  Webhook recibe datos en n8n
        ↓
T=0.2s  Procesa y normaliza datos
        ↓
T=0.3s  Lanza 3 llamadas en paralelo:
        ├─→ OpenWeather API    (responde en 0.5s)
        ├─→ Geoapify Hoteles   (responde en 0.8s)
        └─→ Geoapify Rest.     (responde en 0.7s)
        ↓
T=1.1s  Todas las APIs respondieron
        Combina datos
        ↓
T=1.2s  Envía prompt completo a Claude
        ↓
T=8.5s  Claude genera HTML (tarda ~7s)
        ↓
T=8.6s  Gmail envía email
        ↓
T=9.0s  Usuario recibe email

TIEMPO TOTAL: ~9 segundos
```

---

## 🔌 APIs Utilizadas

### 1. Anthropic Claude API

**Modelo**: claude-sonnet-4-20250514  
**Propósito**: Generar contenido personalizado en HTML

**Ventajas**:
- Excelente comprensión de contexto
- Genera HTML válido y bien formateado
- Personalización basada en datos estructurados
- Consistencia en outputs

**Limitaciones**:
- Latencia de ~7 segundos por request
- Coste por tokens (input + output)
- Requiere prompts bien estructurados

**Configuración óptima**:
```json
{
  "model": "claude-sonnet-4-20250514",
  "max_tokens": 4096,
  "temperature": 0.7
}
```

---

### 2. OpenWeather API

**Endpoint**: `api.openweathermap.org/data/2.5/weather`  
**Propósito**: Obtener clima actual del destino

**Datos utilizados**:
- Temperatura actual
- Sensación térmica
- Humedad
- Descripción del clima
- Nombre de la ciudad

**Manejo de errores**:
- Si la ciudad no existe → Usa clima genérico
- Si API falla → Continúa sin datos de clima

---

### 3. Geoapify Places API

**Endpoint**: `api.geoapify.com/v2/places`  
**Propósito**: Buscar hoteles y restaurantes pet-friendly

**Categorías usadas**:
- `accommodation` → Hoteles
- `catering.restaurant` → Restaurantes

**Filtros aplicados**:
- Radio de 3-5km del destino
- Límite de 10 resultados
- Ordenado por relevancia

**Desafío**: Geoapify no filtra específicamente "pet-friendly", por lo que:
1. Se obtienen resultados generales
2. Claude recibe la lista
3. Claude decide cuáles son aptos para mascotas basándose en su conocimiento

---

## 🔒 Seguridad

### Protección de API Keys

- ✅ Todas las keys están en variables de entorno de n8n
- ✅ Nunca se exponen en el frontend
- ✅ El webhook de n8n es la única puerta de entrada

### Validación de Datos

**Frontend**:
```javascript
// Validación de email
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
if (!emailRegex.test(email)) {
    alert('Email inválido');
    return;
}

// Validación de fechas
if (new Date(fecha_inicio) < new Date()) {
    alert('La fecha de inicio no puede ser en el pasado');
    return;
}
```

**Backend (n8n)**:
- Sanitización de inputs
- Límite de caracteres por campo
- Validación de tipos de datos

### Rate Limiting

- Webhook de n8n: Sin límite (controlado por n8n Cloud)
- Claude API: 5 requests/min (límite de Anthropic)
- OpenWeather: 60 requests/min (plan free)
- Geoapify: 3,000 requests/day (plan free)

---

## ⚠️ Manejo de Errores

### Estrategia de Fallbacks

```
Si OpenWeather falla:
  ↓
Continúa sin datos de clima
Claude genera guía sin sección de clima

Si Geoapify falla:
  ↓
Claude inventa hoteles/restaurantes conocidos del destino
basándose en su conocimiento previo

Si Claude falla:
  ↓
Envía email con mensaje de error
Notifica al admin

Si Gmail falla:
  ↓
Reintenta 3 veces con delay exponencial
Si sigue fallando → Notifica al admin
```

### Logging y Debugging

n8n guarda automáticamente:
- Todos los inputs/outputs de cada nodo
- Errores con stack trace
- Tiempo de ejecución de cada paso

Acceso a logs: `n8n.cloud → Executions → [Seleccionar ejecución]`

---

## ⚡ Optimizaciones

### 1. Llamadas en Paralelo

Las 3 APIs externas se llaman **simultáneamente**, no secuencialmente:
- Sin paralelización: 0.5s + 0.8s + 0.7s = 2.0s
- Con paralelización: max(0.5s, 0.8s, 0.7s) = 0.8s
- **Ahorro: 1.2 segundos**

### 2. Caching (Futuro)

Ideas para optimizar:
- Cachear respuestas de OpenWeather (válidas por 30 min)
- Cachear hoteles/restaurantes por ciudad (válidas por 24h)
- Reducir llamadas a Claude reutilizando estructuras

### 3. Prompt Optimizado

El prompt de Claude está optimizado para:
- **Reducir tokens**: Solo se envían datos relevantes
- **Mejorar precisión**: Instrucciones claras y estructuradas
- **Consistencia**: Mismo formato HTML siempre

---

## 📈 Escalabilidad

### Límites Actuales

| Componente | Límite | Solución si se supera |
|------------|--------|-----------------------|
| n8n Cloud  | 2,500 ejecuciones/mes | Migrar a n8n self-hosted |
| Claude API | 5 req/min | Implementar cola de requests |
| OpenWeather | 60 req/min | Upgrade a plan de pago |
| Geoapify | 3,000 req/día | Implementar caching |
| Netlify | 100GB bandwidth/mes | Upgrade a plan Pro |

### Estrategia de Escalado

**Si llega a 100 usuarios/día**:
1. Implementar caching de APIs
2. Upgrade a plan de pago de OpenWeather y Geoapify

**Si llega a 1,000 usuarios/día**:
1. Migrar n8n a self-hosted (AWS/DigitalOcean)
2. Implementar cola con Redis
3. Múltiples workers para procesar requests

**Si llega a 10,000 usuarios/día**:
1. Backend propio (Node.js + Express)
2. Base de datos para histórico de guías
3. CDN para assets estáticos
4. Load balancer

---

## 🛠️ Herramientas de Desarrollo

- **IDE**: VS Code
- **Control de versiones**: Git + GitHub
- **Testing**: Manual (futuro: Jest para frontend)
- **Debugging**: n8n execution logs + Chrome DevTools
- **Deploy**: Netlify CLI para frontend, n8n Cloud para workflows

---

## 📚 Referencias

- [Documentación n8n](https://docs.n8n.io/)
- [Claude API Documentation](https://docs.anthropic.com/)
- [OpenWeather API Docs](https://openweathermap.org/api)
- [Geoapify Places API](https://www.geoapify.com/places-api)

---

**Última actualización**: Noviembre 2025  
**Versión del documento**: 1.0  
**Autor**: Natalia Ramírez Padilla
