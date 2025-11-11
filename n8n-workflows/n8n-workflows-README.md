# 🔧 n8n Workflow - Patas y Rutas

Este directorio contiene el workflow de automatización completo de Patas y Rutas.

---

## 📋 Descripción

El workflow orquesta todo el proceso de generación de guías personalizadas:

1. **Recibe datos** del formulario web vía webhook
2. **Consulta 3 APIs** en paralelo (clima, hoteles, restaurantes)
3. **Envía contexto a Claude** para generar guía personalizada
4. **Envía email** con la guía en HTML
5. **Responde** al formulario confirmando el envío

---

## ⚙️ Configuración de API Keys

Este workflow requiere configurar las siguientes credenciales en n8n:

### 1. Anthropic Claude API
- **Dónde obtenerla:** https://console.anthropic.com
- **Coste:** Pay-as-you-go (aproximadamente $0.015 por guía)
- **Configuración en n8n:**
  - Nodo: "Claude API - Generar Itinerario"
  - Busca: `TU_ANTHROPIC_API_KEY_AQUI`
  - Reemplaza con tu key real: `sk-ant-api03-...`

### 2. OpenWeather API
- **Dónde obtenerla:** https://openweathermap.org/api
- **Plan:** Free (60 requests/min)
- **Configuración en n8n:**
  - Nodo: "OpenWeather API - Clima Real"
  - Busca: `TU_OPENWEATHER_API_KEY_AQUI`
  - Reemplaza con tu key real

### 3. Geoapify API
- **Dónde obtenerla:** https://www.geoapify.com
- **Plan:** Free (3,000 requests/día)
- **Configuración en n8n:**
  - Nodos: "Geoapify - Hoteles Pet-Friendly" y "Geoapify - Restaurantes Pet-Friendly"
  - Busca: `TU_GEOAPIFY_API_KEY_AQUI` (aparece 2 veces)
  - Reemplaza con tu key real

### 4. Gmail OAuth2
- **Configuración en n8n:**
  - Nodo: "Send a message"
  - Credentials → "New" → "Gmail OAuth2"
  - Sigue el asistente de configuración de n8n
  - Autoriza tu cuenta de Gmail

---

## 📥 Cómo Importar el Workflow

### Método 1: Interfaz Web (Recomendado)

1. Abre tu instancia de n8n (n8n.cloud o self-hosted)
2. Click en el menú **"Workflows"** (sidebar izquierdo)
3. Click en el botón **"+"** (arriba derecha)
4. Selecciona **"Import from file"**
5. Carga el archivo `patas-rutas-workflow.json`
6. Click en **"Import"**

### Método 2: URL (si tienes el archivo en un servidor)

1. Workflows → "+" → "Import from URL"
2. Pega la URL del archivo JSON
3. Import

---

## 🔐 Seguridad: Configurar Credenciales

Una vez importado el workflow, verás que algunos nodos tienen **triángulos rojos** ⚠️ indicando que faltan credenciales.

### ✅ Configuración paso a paso:

1. **Click en cada nodo con error**
2. **Busca el campo de API key o credenciales**
3. **Reemplaza el placeholder** por tu key real
4. **Guarda el nodo** (botón verde en la esquina)
5. **Repite para todos los nodos**

⚠️ **IMPORTANTE:** 
- Nunca compartas tus API keys reales
- n8n guarda las credenciales de forma segura
- Este archivo usa placeholders para proteger tus keys

---

## 🚀 Activar el Workflow

Una vez configuradas todas las credenciales:

1. **Test el workflow:**
   - Click en **"Execute Workflow"** (abajo)
   - Revisa que todos los nodos se ejecuten correctamente

2. **Obtén la URL del webhook:**
   - Click en el nodo "Webhook - Formulario Web"
   - Copia la **Production URL**
   - Ejemplo: `https://[tu-instancia].app.n8n.cloud/webhook/patasyrutas-webhook`

3. **Actualiza tu formulario web:**
   - En tu archivo `form.html` o `form.js`
   - Busca la línea del `fetch()` que envía los datos
   - Reemplaza la URL por la de tu webhook

4. **Activa el workflow:**
   - Toggle en la esquina superior derecha: OFF → **ON** ✅
   - El workflow ahora está escuchando requests 24/7

---

## 🧪 Testing del Workflow

### Test Manual:

1. En n8n, con el workflow abierto
2. Click en "Execute Workflow"
3. En el nodo "Webhook", click en "Listen for test event"
4. Ve a tu formulario web y envía datos de prueba
5. Vuelve a n8n y revisa cada nodo para ver los outputs

### Test Completo:

1. Workflow activado (ON)
2. Formulario web funcionando
3. Envía datos reales
4. Revisa tu email en 30-60 segundos
5. Verifica que la guía sea correcta

---

## 📊 Monitoreo

### Ver ejecuciones:

1. En n8n, ve a **"Executions"** (menú superior)
2. Verás todas las ejecuciones recientes
3. Click en cualquiera para ver detalles:
   - Datos de entrada
   - Output de cada nodo
   - Errores (si los hay)
   - Tiempo de ejecución

### Errores comunes:

| Error | Causa | Solución |
|-------|-------|----------|
| "401 Unauthorized" | API key inválida | Verifica que la key sea correcta |
| "429 Too Many Requests" | Límite de API excedido | Espera o upgrade plan |
| "500 Internal Server Error" | Error en Claude o APIs | Reintenta en unos minutos |
| "No Gmail credentials" | OAuth no configurado | Configura Gmail OAuth2 |

---

## 🔄 Actualizaciones

Si actualizas el workflow:

1. **Exporta la nueva versión:**
   - Workflows → Tu workflow → "..." → **"Download"**
2. **Reemplaza el archivo** en GitHub
3. **Asegúrate de limpiar las keys** antes de subirlo

---

## 📚 Recursos Adicionales

- [Documentación de n8n](https://docs.n8n.io/)
- [Claude API Docs](https://docs.anthropic.com/)
- [OpenWeather API Docs](https://openweathermap.org/api)
- [Geoapify API Docs](https://www.geoapify.com/api-documentation)

---

## ⚠️ Notas Importantes

- **Este archivo es una plantilla** con placeholders en lugar de keys reales
- **Nunca subas tu workflow real** con las keys a GitHub
- **Las credenciales de Gmail** se manejan via OAuth2 en n8n (no están en el JSON)
- **El workflow funciona en n8n Cloud y self-hosted** (versión 1.0+)

---

## 💬 Soporte

Si tienes problemas configurando el workflow:

1. Revisa los logs en **Executions**
2. Verifica que todas las keys sean válidas
3. Asegúrate de que el webhook esté activo
4. Contacta: irpadilla95@gmail.com

---

**Última actualización:** Noviembre 2025  
**Versión del workflow:** 1.0  
**Autor:** Natalia Ramírez Padilla
