# 🔥 Setup Firebase + Google Calendar — Paso a paso

Esta guía te lleva de cero a un sistema funcionando en ~15 minutos. Todo gratis.

> Si en cualquier paso te pierdes, manda screenshot y te desbloqueo.

---

## 📋 Lo que vas a hacer

1. Crear un proyecto en Firebase
2. Activar **Authentication** con Google
3. Activar **Firestore Database**
4. Activar la **Google Calendar API**
5. Pegar tus credenciales en `src/firebase-config.js`
6. Pegar las reglas de seguridad en Firestore
7. Probar

---

## 1. Crear proyecto en Firebase

1. Ve a https://console.firebase.google.com
2. Inicia sesión con tu cuenta Google.
3. Click **"Crear un proyecto"** (o "Añadir proyecto").
4. Nombre del proyecto: `sistema-citas` (o el que quieras).
5. **DESACTIVA** Google Analytics (no lo necesitas y simplifica).
6. Click **"Crear proyecto"** → espera ~30 seg → "Continuar".

---

## 2. Activar Authentication (Login con Google)

1. En el menú izquierdo, abre **"Compilación" → "Authentication"**.
2. Click **"Comenzar"**.
3. En la pestaña **"Sign-in method"**, click sobre **"Google"**.
4. Activa el toggle **"Habilitar"**.
5. Selecciona tu **email de soporte** (el tuyo).
6. Click **"Guardar"**.

✅ Listo — tu app ahora puede recibir logins con Google.

---

## 3. Activar Firestore Database

1. En el menú izquierdo, abre **"Compilación" → "Firestore Database"**.
2. Click **"Crear base de datos"**.
3. **Modo de inicio**: elige **"Comenzar en modo de prueba"** (luego pondremos las reglas reales).
4. **Ubicación**: elige una cercana a tus clientes (ej. `us-east1` o `europe-west1`).
5. Click **"Habilitar"**.

⚠️ El **modo de prueba** caduca en 30 días. Antes debemos pegar las reglas del paso 6.

---

## 4. Activar Google Calendar API

Firebase y Google Cloud comparten el mismo proyecto. Sigue estos pasos:

1. Ve a https://console.cloud.google.com/apis/library/calendar-json.googleapis.com
2. Arriba, asegúrate de que el **selector de proyecto** muestre el mismo nombre que creaste en Firebase (`sistema-citas`).
3. Click **"Habilitar"**.

✅ Esperas ~30 seg.

---

## 5. Obtener las credenciales y pegarlas

1. Vuelve a Firebase Console (https://console.firebase.google.com).
2. Click el **engranaje ⚙️** arriba a la izquierda → **"Configuración del proyecto"**.
3. Baja hasta **"Tus aplicaciones"**.
4. Click el icono **`</>`** (Web app).
5. Apodo: `sistema-citas-web` (o lo que quieras). NO marques "Hosting".
6. Click **"Registrar aplicación"**.
7. Te muestra un bloque de código. Te interesa solo el objeto:

   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "tu-proyecto.firebaseapp.com",
     projectId: "tu-proyecto",
     storageBucket: "tu-proyecto.appspot.com",
     messagingSenderId: "123456789012",
     appId: "1:123456789012:web:abc123..."
   };
   ```

8. **Copia ese objeto** y pégalo en `sistema-citas/src/firebase-config.js`, reemplazando los valores `"TU_API_KEY"` etc.

9. Guarda el archivo.

---

## 6. Pegar reglas de seguridad en Firestore

1. En Firebase Console → **Firestore Database** → pestaña **"Reglas"**.
2. **Borra** lo que haya ahí.
3. Abre el archivo `sistema-citas/firestore.rules` de tu proyecto.
4. **Copia** todo su contenido y pégalo en el editor de reglas.
5. Click **"Publicar"**.

✅ Esto hace que:
- Cualquiera pueda leer la configuración de un negocio (para el formulario público).
- Cualquiera pueda CREAR citas pendientes (cliente que agenda).
- Solo el admin (logueado con su Google) puede leer, aceptar o rechazar SUS citas.

---

## 7. Probar

1. En tu terminal:
   ```bash
   cd sistema-citas
   npm run dev
   ```
2. Abre http://localhost:5173
3. Verás la pantalla "Iniciar sesión con Google" — clica.
4. Aparece una pantalla de Google pidiendo permisos:
   - "Ver y acceder a tu información de Google"
   - **"Ver, modificar y eliminar eventos en tus calendarios de Google"** ← ESTO es Calendar
5. Acepta.
6. Vuelves a la app, ya logueado. Ves un mensaje **"Tu link público:"** con una URL tipo:
   ```
   http://localhost:5173/?biz=AbCdEf123...
   ```
7. **Copia esa URL** y ábrela en una pestaña incógnita (o en otro navegador) → ahí va el cliente a agendar.
8. Crea una cita de prueba como cliente.
9. Vuelve a tu pestaña admin → la cita aparece automáticamente.
10. Click **"Aceptar"** → la cita se sincroniza con tu Google Calendar (revisa https://calendar.google.com).

---

## 🎯 Cómo funciona el flujo completo

```
┌──────────────┐                  ┌─────────────────┐
│ TÚ (admin)   │ login Google ──→ │  Firebase Auth  │
└──────┬───────┘                  └─────────────────┘
       │
       │ comparte link
       ▼
┌──────────────┐                  ┌─────────────────┐
│ CLIENTE      │ rellena form ──→ │   Firestore     │
└──────────────┘                  └────────┬────────┘
                                           │
                                           │ tiempo real
                                           ▼
                                  ┌─────────────────┐
                                  │ TÚ ves la cita  │
                                  │ en tu admin     │
                                  └────────┬────────┘
                                           │ "Aceptar"
                                           ▼
                                  ┌─────────────────┐
                                  │ Google Calendar │ ← evento creado
                                  └────────┬────────┘
                                           │ envía invite
                                           ▼
                                  ┌─────────────────┐
                                  │ CLIENTE recibe  │
                                  │ email con la    │
                                  │ cita en su      │
                                  │ calendario      │
                                  └─────────────────┘
```

---

## ❓ Problemas comunes

### "Cannot find module 'firebase'"
```bash
cd sistema-citas
npm install firebase
```

### "Falta configurar Firebase"
No has pegado tu config en `src/firebase-config.js`. Vuelve al paso 5.

### "permission-denied" al guardar/leer
No has pegado las reglas en Firestore o las pegaste mal. Paso 6.

### Login funciona pero no se crea evento en Calendar
- Verifica que activaste **Google Calendar API** en Cloud Console (paso 4).
- Cierra sesión y vuelve a entrar — al re-loguearte aparece otra vez la pantalla de permisos. Acepta el de Calendar.

### "auth/unauthorized-domain"
Si despliegas en Vercel/Netlify, Firebase necesita autorizar ese dominio:
- Authentication → Settings → **Authorized domains** → Add → `tu-proyecto.vercel.app`

### El link del cliente no muestra nada / "Negocio no encontrado"
La URL debe ser `?biz=<uid>` con tu UID. Cópialo desde el banner azul del panel admin — no lo escribas a mano.

---

## 💰 ¿Esto cuesta dinero?

**Plan gratis (Spark) cubre cómodamente:**
- 50,000 lecturas/día
- 20,000 escrituras/día
- 1 GB de datos
- 10K logins/mes

Para un negocio normal con cientos de citas al mes, **es gratis para siempre**. Si llegas a millones, es ~$0.06 por 100k operaciones.

Google Calendar API: **gratis sin límite** para uso normal.

---

## 🚀 Despliegue a producción (Vercel)

Cuando todo funcione local:

1. Sube el repo a GitHub.
2. Ve a https://vercel.com → "Import Project" → conecta el repo.
3. Build command: `npm run build` · Output: `dist`
4. Deploy.
5. **Importante**: añade el dominio de Vercel a "Authorized domains" en Firebase Auth (ver problemas comunes).

URL final: `https://tu-app.vercel.app`
URL para clientes: `https://tu-app.vercel.app/?biz=<tu-uid>`

---

## ✅ Checklist final

- [ ] Proyecto creado en Firebase
- [ ] Authentication con Google activado
- [ ] Firestore Database creada
- [ ] Google Calendar API habilitada en Cloud Console
- [ ] Config pegada en `src/firebase-config.js`
- [ ] Reglas pegadas en Firestore
- [ ] Login con Google funciona
- [ ] Link público funciona
- [ ] Crear cita como cliente → llega al admin
- [ ] Aceptar cita → aparece en Google Calendar
- [ ] Cliente recibe invite por email
