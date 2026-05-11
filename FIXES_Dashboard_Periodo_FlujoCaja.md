# Correcciones Dashboard — Período y Flujo de Caja

**Archivo:** `index.html` en el repo `guillermosemola/saludgest`  
**Método:** GitHub web editor → editar `index.html` → buscar y reemplazar los dos fragmentos a continuación → hacer commit → Vercel redeploy automático.

---

## FIX 1 — Período dinámico (ya no queda fijo en "Abril 2026")

### Buscar (una sola línea, cerca del inicio del `<script>`):
```js
const periodoActual = 'Abr 2026';
```

### Reemplazar por:
```js
const _now = new Date();
const periodoActual = ['','Ene','Feb','Mar','Abr','May','Jun','Jul','Ago','Sep','Oct','Nov','Dic'][_now.getMonth()+1] + ' ' + _now.getFullYear();
```

---

## FIX 2 — Flujo de Caja en dashboard (deja de quedarse cargando)

### Buscar este fragmento (está al final del bloque de "Pendiente de Cobro" dentro de `loadDashboard`):
```js
  } else { document.getElementById('dbPend').innerHTML='<div class="empty">✅ Sin saldos deudores</div>'; }

  // Dashboard comisiones pendientes
```

### Reemplazar por (inserta el bloque de flujo de caja entre los dos):
```js
  } else { document.getElementById('dbPend').innerHTML='<div class="empty">✅ Sin saldos deudores</div>'; }

  // Flujo de caja — últimos 7 días (cobros acreditados)
  const hace7 = new Date(); hace7.setDate(hace7.getDate()-6);
  const hace7Str = hace7.toISOString().split('T')[0];
  const cobsFlujo = cobs.filter(c => c.estado === 'acreditado' && c.fecha_cobro >= hace7Str);
  const flujoMap = {};
  cobsFlujo.forEach(c => {
    const d = c.fecha_cobro || c.fecha;
    if(!d) return;
    if(!flujoMap[d]) flujoMap[d] = 0;
    flujoMap[d] += Number(c.importe || 0);
  });
  const flujoRows = Object.entries(flujoMap).sort((a,b) => a[0].localeCompare(b[0]));
  if(flujoRows.length) {
    document.getElementById('dbFlujo').innerHTML = `
      <table><thead><tr><th>Fecha</th><th style="text-align:right">Cobros del día</th></tr></thead>
      <tbody>${flujoRows.map(([d,v]) => `<tr>
        <td>${fmtD(d)}</td>
        <td style="text-align:right;font-weight:600;color:var(--success)">${fmt(v)}</td>
      </tr>`).join('')}
      </tbody>
      <tfoot><tr>
        <td style="font-weight:700;color:var(--gray);padding:8px 13px">TOTAL 7 DÍAS</td>
        <td style="text-align:right;font-weight:800;color:var(--teal);font-family:'Cormorant Garamond',serif;font-size:18px;padding:8px 13px">${fmt(flujoRows.reduce((s,[,v])=>s+v,0))}</td>
      </tr></tfoot></table>`;
  } else {
    document.getElementById('dbFlujo').innerHTML = '<div class="empty">Sin cobros acreditados en los últimos 7 días</div>';
  }

  // Dashboard comisiones pendientes
```

---

## Notas

- **Fix 1** calcula el período usando `new Date()` del navegador del usuario, por lo que siempre mostrará el mes y año actuales (ej: "May 2026", "Jun 2026", etc.).
- **Fix 2** usa los datos de `cobros` que ya se cargan en `loadDashboard()` (variable `cobs`), por lo que no hace ninguna consulta extra a Supabase. Filtra cobros acreditados de los últimos 7 días y los agrupa por fecha de cobro. Si no hay cobros en el período, muestra un mensaje vacío en lugar del spinner infinito.
- El campo usado para la fecha es `fecha_cobro` (con fallback a `fecha`), que es el campo real en la tabla `cobros`.
