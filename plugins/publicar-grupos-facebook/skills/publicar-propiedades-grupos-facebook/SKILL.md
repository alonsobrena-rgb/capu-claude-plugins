---
name: publicar-propiedades-grupos-facebook
description: Publica propiedades de Umbral (texto + primeras 10 fotos) en grupos de Facebook desde Chrome, por lotes y gastando lo mínimo del plan. Úsala cuando pidan publicar una o varias propiedades en grupos de Facebook.
---

# Publicar propiedades en grupos de Facebook (Lima)

Úsala cuando el usuario pida publicar una o varias propiedades (links de la web del agente, links de Umbral o IDs) en uno o varios grupos de Facebook.

Esta skill está hecha para gastar poco. Las reglas que más ahorran son:
- **Un chat nuevo por lote.** Cada paso vuelve a procesar toda la conversación, así que el paso 40 de un chat largo cuesta mucho más que el paso 4 de uno nuevo.
- **Sin capturas para mirar:** solo las mínimas de escala 0.1 que Chrome necesita (Paso 5), y una de 0.4 si algo falla.
- **Nada de exploración:** se usan los scripts de abajo tal cual, en lotes (`browser_batch`).
- **Sin elegir fotos:** siempre las primeras 10 de la ficha.
- **Sin redactar:** el texto sale de la plantilla.
- **Sin narrar:** no se comenta entre pasos; se informa solo al final.

## Paso 1. Datos de cada propiedad (una llamada por propiedad)
1. Llama `get_property_full` del conector de Umbral con el link o el ID.
2. Toma solo estos campos:
   - `property`: `title`, `property_name`, `address`, `location`, `operation_type`, `price` o `price_alquiler` y su moneda, `area_total`, `bedrooms`, `bathrooms`, `half_bathrooms`, `garages`, `referencias` y las 2 primeras frases de `description`.
   - `public_url`.
   - Contacto: `contact.whatsapp_phone` (o `tenant.whatsapp_phone`) y `tenant.company_name`.
   - Fotos: las 10 primeras de `media.photos`.
3. No repitas la ficha en la respuesta.
4. Si la propiedad tiene menos de 10 fotos, usa las que haya. Si tiene 0, sáltala y avísalo al final.

## Paso 2. Texto (plantilla fija)
Rota la primera línea según el número de grupo (1→A, 2→B, 3→C, 4→A…). Así el mismo aviso no sale idéntico en todos los grupos.
- A: `🏙️ {VENTA|ALQUILER} – {tipo} en {zona}, {distrito}`
- B: `📍 {zona}, {distrito} | {tipo} en {venta|alquiler}`
- C: `✨ Oportunidad en {distrito}: {tipo} en {zona}`

Cuerpo:
```
{primera línea}

💰 {US$|S/} {precio con comas}

{1 frase gancho tomada de la descripción, máximo 25 palabras}

📐 {área} m² · 🛏️ {dorm} dorm. · 🛁 {baños} baños{ + 1 medio baño si hay} · 🚗 {cocheras} cocheras
✅ {3 atributos clave separados por " · " (vista, terraza, piscina, amenities…)}

📲 WhatsApp: +51 {número con espacios cada 3 dígitos}
🔗 {public_url}

{company_name} – Asesor inmobiliario
```
- Omite las líneas cuyo dato no exista. No inventes nada que no esté en la ficha.
- Nada de hashtags ni mayúsculas sostenidas.

## Paso 3. Confirmar el lote (un solo mensaje)
Antes de publicar, muestra una lista corta: `grupo → propiedad (precio)`. Pregunta una sola vez: "¿Publico estos N?".
- Con un sí, publica todo el lote sin volver a preguntar.
- No muestres los textos completos salvo que lo pidan.
- Si el usuario ya dijo en su mensaje "publica sin preguntar", igual muestra la lista. Para eso basta la confirmación de ese mismo mensaje.

Ritmo recomendado (dilo una vez si el pedido lo supera): máximo 3 publicaciones por grupo al día, la misma propiedad en el mismo grupo no más de una vez por semana, y los lotes repartidos en la mañana, la tarde y la noche. Pasarse de eso hace que Facebook restrinja la cuenta o que los admins la saquen.

## Paso 4. Preparar Chrome (una vez por lote)
1. Carga en UNA sola llamada de ToolSearch: `tabs_context_mcp`, `tabs_create_mcp`, `tabs_close_mcp`, `navigate`, `browser_batch`, `computer`, `javascript_tool`, `find`.
2. Llama `tabs_context_mcp` con `createIfEmpty: true`. Usa esa pestaña como **FB** y crea otra con `tabs_create_mcp` como **FOTOS**.
3. Si la extensión no está conectada, dilo y para. No uses otro navegador.

Las fotos pasan por el portapapeles de la computadora. Facebook no deja traerlas directo desde la web de fotos, y ese es el único camino que funciona. Avisa al usuario en una línea al empezar: **"No uses Ctrl+C en esta compu mientras publico."**

## Paso 5. Publicar cada post
Por cada post (grupo + propiedad) se hacen 4 llamadas. Todo lo que sea `javascript_tool` con un script de abajo se copia tal cual.

Dos cosas que hay que saber, comprobadas en pruebas:
- Chrome solo acepta clics y teclado en una pestaña después de una captura de esa pestaña. Por eso cada llamada A saca una captura a escala **0.1** de FOTOS y otra de FB. Casi no gastan y no hace falta mirarlas.
- La ventana de Chrome no puede estar minimizada durante el lote. Si una captura falla por "renderer frozen", pide al usuario que deje Chrome visible y reintenta una vez.

### Llamada A · `browser_batch`
Ejecuta en orden:
1. `navigate` FOTOS a la primera foto de la propiedad.
2. `javascript_tool` en FOTOS con el **script FOTOS**, cambiando `URLS` por el arreglo de las 10 URLs.
3. `computer` `screenshot` en FOTOS, escala 0.1.
4. `navigate` FB al link del grupo.
5. `computer` `wait` 3.
6. `computer` `screenshot` en FB, escala 0.1.
7. `javascript_tool` en FB con el **script ABRIR**.

Si la propiedad es la misma que la del post anterior, reemplaza los pasos 1 y 2 por un `javascript_tool` en FOTOS con `window.__i=0`. Mantén la captura del paso 3.

Según lo que devuelva ABRIR:
- `abierto`: sigue con la llamada A2.
- `sin-sesion`: para todo el lote y pide al usuario que inicie sesión en Facebook en ese Chrome. Nunca escribas contraseñas.
- `no-miembro`: salta ese grupo y anótalo. No pidas unirte por tu cuenta.
- `sin-compositor`: el grupo usa otro formulario, por ejemplo uno de compra y venta. Toma 1 captura a escala 0.4, salta ese grupo y anótalo.

### Llamada A2 · `find` en FB
Busca `cuadro de texto editable del diálogo Crear publicación` y guarda la referencia (`ref_N`). Si no aparece en 2 intentos, anota el grupo como fallido.

### Llamada B · `browser_batch`
Ejecuta en orden:
1. `computer` `left_click` en FB con `ref` = la referencia de A2.
2. `computer` `type` en FB con el texto del Paso 2.
3. Diez veces seguidas, un bloque por foto:
   - `computer` `left_click` en FOTOS en `[100, 60]`. Ese clic copia la foto siguiente.
   - `computer` `wait` 2.
   - `computer` `key` `ctrl+v` en FB.
   - `computer` `wait` 1.
4. `computer` `wait` 2.
5. `javascript_tool` en FOTOS: `document.getElementById('s').textContent`.
6. `javascript_tool` en FB con el **script VERIFICAR**.

Cómo leer el resultado:
- **Todo bien:** FOTOS muestra `ok0` … `ok9` (o tantos `ok` como fotos), VERIFICAR devuelve `fotos` igual a ese número y `fin` termina en "Asesor inmobiliario". Pasa a la llamada C.
- **FOTOS vacío:** los clics no llegaron porque faltó la captura de FOTOS. Corre el **script DESCARTAR**, anota el grupo como fallido y avísalo.
- **`fin` trae texto extra al final:** el usuario copió algo durante el lote. Borra solo ese texto con `key` `BackSpace` y `repeat` igual a su largo. Revisa que la cantidad de fotos sea la correcta. Si falta alguna, corre DESCARTAR, anota el grupo como fallido y sigue con el siguiente. No lo reintentes en este lote.
- **Algún `ERR` en FOTOS o `fotos` menor a lo esperado:** corre DESCARTAR y anótalo.

### Llamada C · `javascript_tool` en FB con el **script PUBLICAR**
- `publicado`: listo.
- `sigue-abierto`: toma 1 captura a escala 0.4 y anótalo.
- `sin-boton`: corre DESCARTAR y anótalo.
- Entre un post y el siguiente no hace falta esperar más. El espaciado se logra repartiendo los lotes en el día, no metiendo pausas dentro del lote.

## Scripts (copiar tal cual)

**FOTOS** (pestaña FOTOS; reemplaza `URLS`):
```js
window.__f=URLS; window.__i=0;
document.body.innerHTML='<button id=b style="font-size:40px;padding:40px">copiar</button><div id=s></div>';
document.getElementById('b').onclick=async()=>{const s=document.getElementById('s');const i=window.__i++;try{const b=await(await fetch(window.__f[i])).blob();const bm=await createImageBitmap(b);const k=Math.min(1,1600/bm.width);const c=document.createElement('canvas');c.width=Math.round(bm.width*k);c.height=Math.round(bm.height*k);c.getContext('2d').drawImage(bm,0,0,c.width,c.height);const p=await new Promise(r=>c.toBlob(r,'image/png'));await navigator.clipboard.write([new ClipboardItem({'image/png':p})]);s.textContent+='ok'+i+' ';}catch(e){s.textContent+='ERR'+i+' '+e.message+' ';}};
'listo'
```

**ABRIR** (pestaña FB):
```js
(()=>{const t=document.body.innerText;
if(/Iniciar sesión/.test(t)&&/Crear cuenta nueva/.test(t))return 'sin-sesion';
if([...document.querySelectorAll('[role=button]')].some(b=>/^Unirte al grupo$/.test(b.innerText.trim())))return 'no-miembro';
const e=[...document.querySelectorAll('[role=button],span')].find(x=>/^(Escribe algo|Write something)/.test(x.innerText.trim()));
if(!e)return 'sin-compositor';
(e.closest('[role=button]')||e).click();return 'abierto';})()
```

**VERIFICAR** (pestaña FB):
```js
(()=>{const ed=document.querySelector('[role=dialog] [contenteditable=true]');if(!ed)return 'sin-dialogo';const d=ed.closest('[role=dialog]');
const plus=[...d.querySelectorAll('div,span')].map(e=>(e.childElementCount===0?e.textContent.trim():'')).find(t=>/^\+\d+$/.test(t));
const tiles=new Set([...d.querySelectorAll('img')].map(i=>i.src).filter(s=>s.startsWith('blob:'))).size;
const fotos=plus?4+parseInt(plus.slice(1)):tiles;
return JSON.stringify({fotos,fin:ed.innerText.trim().slice(-45)});})()
```
Facebook muestra como máximo 5 miniaturas. Cuando hay más fotos, la quinta lleva un "+N" encima, y el total es 4 + N. Con 10 fotos se ve "+6".

**PUBLICAR** (pestaña FB; empieza con `await`, no lo quites):
```js
await (async()=>{const ed=document.querySelector('[role=dialog] [contenteditable=true]');const d=ed&&ed.closest('[role=dialog]');if(!d)return 'sin-dialogo';
const b=[...d.querySelectorAll('[role=button]')].find(x=>x.getAttribute('aria-label')==='Publicar'||x.innerText.trim()==='Publicar');if(!b)return 'sin-boton';
b.click();for(let k=0;k<24;k++){await new Promise(r=>setTimeout(r,500));if(!document.querySelector('[role=dialog] [contenteditable=true]'))return 'publicado';}return 'sigue-abierto';})()
```

**DESCARTAR** (pestaña FB; cierra el compositor sin publicar):
```js
await (async()=>{const ed=document.querySelector('[role=dialog] [contenteditable=true]');const d=ed&&ed.closest('[role=dialog]');if(!d)return 'nada';
const x=d.querySelector('[aria-label^="Cerrar"],[aria-label^="Close"]');if(x)x.click();await new Promise(r=>setTimeout(r,1500));
const y=[...document.querySelectorAll('[role=dialog] [role=button]')].find(b=>/^(Descartar|Discard)$/.test(b.innerText.trim()));if(y)y.click();await new Promise(r=>setTimeout(r,1000));
return document.querySelector('[role=dialog] [contenteditable=true]')?'sigue-abierto':'descartado';})()
```

**Ojo con la salida de `javascript_tool`:** si lo que devuelve trae `=`, `&` o `?`, la herramienta lo bloquea. Los scripts de arriba ya lo evitan; no agregues HTML ni URLs a lo que devuelven.

## Paso 6. Cerrar y entregar
1. Cierra las pestañas FB y FOTOS con `tabs_close_mcp`.
2. Responde con una tabla corta: `| Grupo | Propiedad | Estado |`. El estado es publicado, saltado (no miembro / otro formulario) o fallido (motivo en 3 palabras).
3. Cierra con una línea: "Ya puedes usar Ctrl+C."

## Reglas
- Nunca ingreses contraseñas ni inicies sesión. Nunca te unas a grupos ni aceptes reglas de grupos por tu cuenta.
- No comentes, no reacciones y no respondas mensajes en los grupos.
- Publica solo lo que el usuario confirmó en el Paso 3.
- Si algo falla 2 veces seguidas en el mismo paso, para el lote y cuenta qué pasó.
- Si Facebook muestra un aviso de restricción, de "actividad sospechosa" o de límite de publicaciones, para todo el lote de inmediato y avisa. No lo reintentes.
- Gasto: lotes, cero exploración, capturas de 0.1 para activar pestañas y de 0.4 solo si falla algo.
