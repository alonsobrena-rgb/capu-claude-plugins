# Capu Inmobiliaria · Plugins para Claude

Herramientas de Claude para los agentes de Capu: búsqueda multiportal y publicación en grupos de Facebook.

## Búsqueda multiportal de inmuebles

Pegas el requerimiento de un cliente en Claude y Claude abre **Tokko (Red), RE/MAX, YoBusco y Babilonia** en tu Chrome con los filtros ya puestos (distritos, precio, dormitorios, cocheras, área) y comprueba que hayan quedado bien. Tú revisas los avisos y decides.

### Qué necesitas

- Una cuenta de Claude con plan pago (Pro o superior).
- La extensión **Claude in Chrome** instalada en el Chrome de tu computadora, con la misma cuenta de Claude.
- Tu sesión de **Tokko Broker** abierta en ese Chrome.
- Computadora: no funciona desde el celular.

### Instalación (una sola vez)

1. En Claude, entra a **Customize > Plugins**.
2. Toca **Add > Add marketplace > Add from a repository**.
3. Escribe `alonsobrena-rgb/capu-claude-plugins` y confirma.
4. En la lista que aparece, instala **busqueda-multiportal**.

En Claude Code, el equivalente es:

```
/plugin marketplace add alonsobrena-rgb/capu-claude-plugins
/plugin install busqueda-multiportal@capu-inmobiliaria
```

### Cómo usarlo

Abre un chat nuevo, pega el requerimiento del cliente tal como te llegó y escribe **"abre los portales"**. Claude deja las pestañas filtradas y te dice qué falta marcar a mano.

### Cuando haya una versión nueva

Si te avisan que hay cambios y no los ves, quita el marketplace **capu-inmobiliaria** en **Customize > Plugins** y vuelve a agregarlo con los pasos de instalación.
En Claude Code: `/plugin marketplace update capu-inmobiliaria`.

## Publicar propiedades en grupos de Facebook

Le pasas a Claude los links de las propiedades y de los grupos. Claude saca los datos de Umbral, arma el texto con una plantilla fija y publica en cada grupo desde tu Chrome con las **primeras 10 fotos** de la ficha. Antes de publicar te muestra la lista del lote y espera tu "sí".

### Qué necesitas

- Una cuenta de Claude con plan pago (Pro o superior), con el conector de **Umbral** activado.
- La extensión **Claude in Chrome** en el Chrome de tu computadora, con la misma cuenta de Claude.
- Tu sesión de **Facebook** abierta en ese Chrome, ya como miembro de los grupos.

### Instalación

Igual que la búsqueda multiportal, pero en la lista instala **publicar-grupos-facebook**.
En Claude Code: `/plugin install publicar-grupos-facebook@capu-inmobiliaria`.

### Cómo usarlo

Abre **un chat nuevo por cada lote** (así gasta mucho menos) y escribe, por ejemplo:

> publica en estos grupos: [link grupo 1], [link grupo 2], [link grupo 3]
> propiedades: [link propiedad A], [link propiedad B]

Claude reparte las propiedades entre los grupos, te muestra la lista y, cuando confirmas, publica.

**Mientras corre el lote:**
- No uses Ctrl+C en esa computadora. Las fotos pasan por el portapapeles.
- No minimices Chrome.

**Ritmo recomendado:** hasta 3 publicaciones por grupo al día, repartidas en la mañana, la tarde y la noche, y la misma propiedad en el mismo grupo no más de una vez por semana. Más que eso hace que Facebook restrinja la cuenta o que los admins te saquen del grupo.

## Sin Claude in Chrome

La página **Buscador Multiportal** genera los mismos links filtrados desde cualquier navegador, incluido el celular, y no consume tu plan. Pídele el link a Alonso.
