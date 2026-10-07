---
name: busqueda-multiportal-inmuebles
description: Abre Tokko, RE/MAX, YoBusco y Babilonia en Chrome con los filtros del requerimiento de un cliente ya puestos y verificados, para que el agente revise los avisos uno por uno.
---

# Búsqueda multiportal de inmuebles (Lima)

Úsala cuando el usuario pegue el requerimiento de un cliente (ubicación, m², dormitorios, cocheras, presupuesto, extras) y pida buscar en portales o abrir las búsquedas.

El trabajo de Claude termina en dejar los portales abiertos con los filtros correctos y comprobados. La revisión de los avisos y la decisión las hace la persona: Claude NO lee los avisos, NO arma listas ni tablas de propiedades y NO opina sobre cuáles convienen, aunque se lo pidan de pasada. Si lo piden expresamente, recuérdales en una línea que esta skill solo deja los filtros listos.

Para gastar poco, Claude no explora los buscadores con clics: arma las URLs con las fórmulas de abajo y solo navega a ellas. Los únicos clics son los extras de Babilonia, que no viajan en la URL.

Alternativa gratis: la página "Buscador Multiportal" genera los mismos links sin consumir el plan. Si el usuario solo quiere los links, recuérdasela en una línea.

## Paso 1. Leer el requerimiento y armar la ficha
- **Datos personales**: nombre, DNI, correo y teléfono se ignoran. Nunca van en una URL, en la respuesta ni en archivos.
- **Operación**: venta por defecto; alquiler si dice alquiler/alquilar/mensual.
- **Tipo**: departamento por defecto (depa, dpto, flat, dúplex); casa, terreno, oficina, local comercial.
- **Distritos**: ver tablas de abajo. Las zonas se traducen a distritos y el agente afina en el mapa.
- **Área**: si da un número suelto ("150m2"), usa desde = redondeo a 5 de 93 % (150 → 140). Si dice "mínimo"/"desde", usa el número exacto. Si da rango, usa el rango.
- **Dormitorios / baños / cocheras**: el número indicado como mínimo. "Cocheras" en plural sin número = 2. "Paralelas" se anota aparte.
- **Presupuesto**: "US$350K" = 350000 USD. "S/" o "soles" = PEN. Es el precio máximo (sin margen salvo que el usuario lo pida).
- **Extras**: balcón/terraza, cuarto de servicio, baño de servicio, ascensor, piscina, vista al mar, frente a parque, jardín, mascotas, amoblado, depósito. Si la línea dice "no relevante" u "opcional", ese extra se ignora.

Muestra la ficha en una línea y sigue sin esperar confirmación, salvo que falte el distrito o el presupuesto.

## Paso 2. Armar las URLs

### Tokko Broker (Red Tokko, usa la sesión de Tokko de quien corre la skill)
`https://www.tokkobroker.com/properties/?go_network=True&search_options=` + encodeURIComponent(JSON) con este JSON:
```json
{"filters":[["suite_amount",">","DORM-1"],["parking_lot_amount",">","COCH-1"],["total_surface",">","AREA-1"]],
 "only_available":"checked","only_reserved":"undefined","only_to_be_cotized":"undefined","only_not_available":"undefined",
 "with_tags":[],"without_tags":[],"with_custom_tags":[],"with_or_custom_tags":[],"without_custom_tags":[],
 "listing_edition_review":"undefined","division_filters":[IDS_TOKKO],"state_filters":[],
 "current_localization_id":"0","current_localization_type":"","network":[3],"exclude_my_properties":false,
 "price_from":"0","price_to":PRECIO_MAX,"operation_types":[1],"property_types":[2],"currency":"USD","bounding_box":[]}
```
- operation_types: 1 venta, 2 alquiler. property_types: 2 depto, 3 casa, 1 terreno, 5 oficina, 7 local.
- Baños: `["bathroom_amount",">","N-1"]`. El operador `>=` NO funciona, siempre `>` con N-1.
- Abre directo en la pestaña "Red Tokko Broker"; al lado está la pestaña "Urbania" con los mismos filtros.
- Si Tokko muestra la pantalla de login, avisa que esa persona debe iniciar sesión en Tokko y sigue con los demás portales.

### RE/MAX Perú
`https://www.remax.pe/web/search/all/propertys/list/?` + parámetros:
`type__in` (departament | house | ground | office | local), `contract__type` (sale | rent), `contract__price_money__exclude` ($ o S/.), `contract__price__range=0,MAX`, `bedrooms__range=N,999999999`, `bathrooms__range=N,999999999`, `parking_lots__range=N,999999999`, `occupied_area__range=MIN,999999999`, `departament__in=15`, `province__in=128`, y un `district__in=ID` por distrito (se repite).
- Cocheras paralelas: `parking_lots_type__in=parallel_close&parking_lots_type__in=parallel_open` (solo si el usuario quiere exigirlo).
- `characteristics__in` combina con "o": usa como máximo un grupo, y solo si el usuario quiere exigir extras. Ids: 13 balcón + 8 patio/terraza, 33 cuarto de servicio, 25 ascensor, 17 piscina, 4 frente al mar, 6 frente al parque, 14 jardín, 26 mascotas, 3 amoblado, 34 almacén.
- Distritos del Callao: sin id mapeado; omite el distrito y avisa que se elige en el buscador.
- Puede salir una verificación de Cloudflare que pasa sola en unos segundos. Si pide un clic o captcha, para y pide al usuario que la resuelva.

### YoBusco
`https://yobusco.pe/buscar/{venta|alquiler}-de-{departamento|casa|terreno|oficina|local-comercial}-en-{SLUG1}-o-{SLUG2}`
+ `?bedroom=N&bathrooms=N&parking=N&currencyTypeId=2&priceMin=X&priceMax=Y&totalAreaMin=A&totalAreaMax=B`
- currencyTypeId: 2 = US$, 1 = soles.
- Extras (solo si el usuario quiere exigirlos): `balcony=true`, `service_room=true`, `elevator=true`, `pets=true`, `deposit=true`.
- Slug: nombre en minúsculas, sin tildes pero conservando la ñ, espacios a guiones, + `-lima-lima` (ej. `breña-lima-lima`, `san-isidro-lima-lima`). Cercado = `lima-lima-lima`. Callao = `{nombre}-prov.-const.-del-callao-prov.-const.-del-callao` (Carmen de la Legua = `carmen-de-la-legua-r-…`). Codifica la ñ en la URL.

### Babilonia (dominio correcto: babilonia.pe; babilonia.io redirige a un sitio sospechoso, nunca lo abras)
`https://babilonia.pe/inmuebles/{departamentos|casas|terrenos|oficinas|locales-comerciales}-en-{venta|alquiler}-en-{SLUG}/{N}-dormitorios/0:{MAX}-{usd|pen}/{MIN}:mas-m2-area`
- Un link por distrito. Slug: `lima-lima-` + nombre sin tildes, ñ→n, espacios a guiones (ej. `lima-lima-brena`, `lima-lima-magdalena-del-mar`, `lima-lima-cercado-de-lima`). Callao: `callao-callao-{nombre}` (`callao-callao-carmen-de-la-legua-reynoso`).
- La moneda en la URL es `usd` o `pen` (nunca `soles`).
- Cocheras, baños y extras NO van en la URL (ver Paso 3).

### Hol.pe
Requiere sesión y aún no está mapeado. Si el usuario ya inició sesión y lo pide, explora sus filtros una vez, anota cómo arma la URL y propone actualizar esta skill.

## Paso 3. Abrir en Chrome
1. Carga en UNA sola llamada de ToolSearch: tabs_context_mcp, tabs_create_mcp, navigate, browser_batch, computer, find, get_page_text, javascript_tool.
2. `tabs_context_mcp`, luego crea una pestaña por link y navega con `browser_batch` (todo en un solo lote, con esperas de 4 a 6 segundos).
3. **Babilonia, extras**: en cada pestaña abre el botón "Filtros" (el de las líneas de ajuste junto a "Precio"), usa `find` para obtener las referencias y haz clic real en: Estacionamientos N+, Baños N+ y las características pedidas ("Terraza en la unidad", "Cuarto de servicio", "Baño de servicio", "Estacionamiento paralelo", "Ascensores", "Piscina", "Vista al mar", "Frente a parque", "Jardín privado", "Pet-friendly", "Amoblado", "Almacén"). Luego "Aplicar". Nada más se toca.
4. **Tokko**: si aparece el popup "Ayuda rápida", ciérralo con la X. Ignora la encuesta "¿Qué tan probable…?" (no la respondas).

## Paso 4. Verificar que los filtros quedaron bien
Comprueba cada pestaña leyendo lo que el portal muestra (texto, no capturas, salvo que algo falle). Lee solo los indicadores de abajo, nunca los avisos.

- **Babilonia**: el encabezado debe decir algo como "N departamentos hasta USD 350,000 con 3 o más dormitorios en venta en San Isidro, Lima". Revisa que el distrito, el precio y los dormitorios coincidan con la ficha. Si dice "¡Lo sentimos! No encontramos…" con un distrito raro (por ejemplo "Soles"), el slug o la moneda están mal: corrige la URL. Si de verdad no hay resultados, déjalo anotado. Después de aplicar los extras, reabre "Filtros" y confirma con `find` que Estacionamientos, Baños y cada característica quedaron marcados; luego ciérralo.
- **YoBusco**: el encabezado "N Inmuebles en Venta en {distrito}" debe nombrar un distrito pedido. Si muestra San Isidro con unos 2,080 resultados cuando no se pidió San Isidro, el slug es inválido y cayó a la búsqueda por defecto: corrige el slug (ñ, Callao). Confirma que la URL final conserva `bedroom`, `parking`, `priceMax` y `totalAreaMin`.
- **RE/MAX**: debe aparecer "Resultado: N" y la URL final debe conservar los parámetros. Con `javascript_tool` confirma que el selector de distrito (`[name="district__in"]`) tiene seleccionados los distritos pedidos.
- **Tokko**: con `get_page_text` o `find` confirma los campos de arriba (Venta, Departamento, moneda y rango de precio, distritos), los chips de "Filtrado por" (Dormitorios, Cocheras, Área total) y que la pestaña activa es "Red Tokko Broker".
- Al devolver texto con `javascript_tool`, quita `?`, `&` y `=` del resultado (o devuelve solo `location.pathname`); si no, la herramienta lo bloquea como dato de query string.
- Si un filtro no quedó, corrige la URL o el clic una vez y vuelve a verificar. Si sigue fallando, déjalo anotado como "marcar a mano".

## Paso 5. Entregar
Deja todas las pestañas abiertas y responde con una tabla corta:

| Portal | Filtros verificados | Resultados | Marcar a mano |

- "Resultados" es solo el total que muestra el portal.
- En "Marcar a mano" van los extras que el portal no filtra (por ejemplo cocheras paralelas o cuarto de servicio en Tokko y YoBusco) y cualquier filtro que no se pudo verificar.
- Cierra con una línea: la persona revisa los avisos y decide.

## Reglas
- Nunca ingreses contraseñas ni inicies sesión. Si un portal pide login, avisa al usuario.
- Nunca contactes agentes ni hagas clic en WhatsApp, Correo, Contactar o Compartir.
- Si una acción falla 2 o 3 veces, para y cuenta qué pasó.
- Gasto: lotes de acciones, cero exploración y pocas capturas.

## Tabla de distritos (id RE/MAX | ids Tokko)
Ancón 1278 | 246233 · Ate 1279 | 240063 · Barranco 1280 | 246703 · Breña 1281 | 246722 · Carabayllo 1282 | 246975 · Chaclacayo 1283 | 247248 · Chorrillos 1284 | 247294 · Cieneguilla 1285 | 84154, 247471 · Comas 1286 | 84155, 247482 · El Agustino 1287 | 247633 · Independencia 1288 | 247721 · Jesús María 1289 | 247792 · La Molina 1290 | 247804 · La Victoria 1291 | 247936 · Cercado de Lima 1277 | 247978 · Lince 1292 | 248097 · Los Olivos 1293 | 248105 · Lurigancho 1294 | 240064 · Lurín 1295 | 84164, 248247 · Magdalena del Mar 1296 | 240065 · Miraflores 1298 | 248359 · Pachacámac 1299 | 248415 · Pucusana 1300 | 84168, 248507 · Pueblo Libre 1297 | 248538 · Puente Piedra 1301 | 248574 · Punta Hermosa 1302 | 249015 · Punta Negra 1303 | 249028 · Rímac 1304 | 249052 · San Bartolo 1305 | 84174 · San Borja 1306 | 249126 · San Isidro 1307 | 249162 · San Juan de Lurigancho 1308 | 249201 · San Juan de Miraflores 1309 | 249709 · San Luis 1310 | 249976 · San Martín de Porres 1311 | 250006 · San Miguel 1312 | 250468 · Santa Anita 1313 | 250524 · Santa María del Mar 1314 | 250602 · Santa Rosa 1315 | 250609 · Santiago de Surco 1316 | 250626 · Surquillo 1317 | 250965 · Villa El Salvador 1318 | 251230 · Villa María del Triunfo 1319 | 251004
Callao (sin id RE/MAX): Callao 83439, 243245 · Bellavista 83437, 243193 · La Perla 83440, 243246 · La Punta 83441, 243292 · Carmen de la Legua 83438, 243232 · Ventanilla 83442

## Zonas → distritos
- Mirasidro → Miraflores + San Isidro · Mirabarranco → Miraflores + Barranco
- Lima Top → San Isidro, Miraflores, Surco, San Borja, La Molina, Barranco
- Lima Moderna → Barranco, Jesús María, Lince, Magdalena, Miraflores, Pueblo Libre, San Borja, San Isidro, San Miguel, Surco, Surquillo
- Surco, Monterrico, Casuarinas, Higuereta, El Polo, Golf Los Incas, Valle Hermoso, La Encalada → Santiago de Surco
- Chacarilla → Surco + San Borja · Camacho → La Molina + Surco
- La Planicie, Rinconada, Sol de La Molina, Musa, Las Lagunas, Santa Patricia → La Molina
- Corpac, Orrantia, El Olivar, Golf de San Isidro, Country Club, Centro Financiero, Limatambo → San Isidro
- Santa Cruz, Aurora, Parque Kennedy, Larcomar, Reducto, Armendáriz, Malecón Cisneros / de la Reserva → Miraflores
- La Encantada, Country Club de Villa, Club Villa, Morro Solar → Chorrillos
- Campo de Marte → Jesús María · Maranga → San Miguel · Magdalena → Magdalena del Mar
- Cercado, Centro de Lima → Cercado de Lima · Chosica → Lurigancho
- SJM, SJL, SMP, VMT, VES → San Juan de Miraflores, San Juan de Lurigancho, San Martín de Porres, Villa María del Triunfo, Villa El Salvador
