# Capu Inmobiliaria · Plugins para Claude

Herramientas de Claude para los agentes de Capu.

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

## Sin Claude in Chrome

La página **Buscador Multiportal** genera los mismos links filtrados desde cualquier navegador, incluido el celular, y no consume tu plan. Pídele el link a Alonso.
