## Objetivo
Estandarizar el nombre de los notebooks para que sea fácil:
- identificar **workspace**, **flujo entre capas** (Medallion), **entidad** y **tipo de proceso**,
- ordenar por versión,
- buscar y referenciar notebooks sin ambigüedad.

---

## Formato oficial
**Formato**
`NB_<WS>_<ORIG2DEST>_<ENTIDAD>_<PROCESO>_vNN`

**Ejemplo**
`NB_Catalogos_B2S_Productos_MergeUpsert_v01`

### Componentes
- `NB`: prefijo fijo para notebooks.
- `<WS>`: workspace lógico (ej. `Catalogos`, `Venta`, `Almacen`, `Finanzas`).
- `<ORIG2DEST>`: movimiento entre capas o puede tratarse de refinamiento también.
- `<ENTIDAD>`: entidad principal del notebook (tabla/objeto de negocio).
- `<PROCESO>`: acción principal (ver catálogo de procesos).
- `_vNN`: versión de 2 dígitos (`v01`, `v02`, ...).

---

## Reglas de estilo
- Usar **PascalCase** en `<WS>`, `<ENTIDAD>`, `<PROCESO>` (sin espacios).
- Evitar acentos, ñ y caracteres especiales: `Catalogos`, `Descripcion`, `Anio`.
- Separador fijo: `_` entre componentes.
- Mantener `<ENTIDAD>` en **plural** cuando represente conjunto (`Productos`, `Clientes`).
- Mantener nombres **cortos pero claros** (ideal: 2–4 palabras máximo en `<ENTIDAD>`).
- Solo una “acción principal” por notebook (si hay dos, separarlo en dos notebooks o usar `Refine`/`Stage` y `MergeUpsert`).

---

## Valores estándar para `<ORIG2DEST>`
- `B2S`: Bronze → Silver
- `S2G`: Silver → Gold
- `B2G`: Bronze → Gold (solo con justificación)
- `SilverRefine`: transformaciones internas en Silver (stage → final silver, o refinamientos)
- `GoldRefine`: transformaciones internas en Gold (agregados, snapshots, ajustes)

---

## Catálogo estándar para `<PROCESO>`
### Ingesta / transformación / carga
- `BronzeToSilver`: transformación de Bronze a Silver (alternativa explícita a `B2S`; usar solo si lo prefieres, no mezclar)
- `Stage`: carga intermedia para deduplicación, preparación de llaves y validaciones previas al MERGE
- `MergeUpsert`: MERGE/UPSERT incremental al destino (SCD1 típico)
- `MergeSCD1`: variante explícita de SCD Tipo 1 (slow change dimension)
- `FullLoad`: recarga completa (overwrite) del destino
- `Incremental`: carga incremental basada en watermark (si no es específicamente un MERGE)

### Curación / analítica (común en Gold)
- `BuildDim`: construcción de dimensión
- `BuildFact`: construcción de hecho
- `ConformDim`: conformado/alineación de dimensiones a estándar común
- `Bridge`: tabla puente (many-to-many, jerarquías, múltiples categorías)
- `Snapshot`: corte/estado a una fecha (ej. inventario al cierre)
- `Aggregate`: agregaciones para rendimiento (diario/mensual/sucursal)

### Calidad / operación
- `DQChecks`: pruebas de calidad (duplicados, nulos, rangos, reconciliación)
- `Backfill`: reproceso histórico por rango/partición
- `Optimize`: mantenimiento Delta (`OPTIMIZE`, `ZORDER`, `VACUUM`)
- `Publish`: preparación/publicación de salidas listas para consumo (tablas finales/vistas/artefactos)

> Recomendación: usar **un conjunto reducido** de procesos y no inventar etiquetas nuevas sin necesidad.

---

## Ejemplos recomendados
- `NB_Catalogos_B2S_Productos_Stage_v01`
- `NB_Catalogos_B2S_Productos_MergeUpsert_v01`
- `NB_Catalogos_S2G_Productos_BuildDim_v01`
- `NB_Almacen_S2G_InventarioActual_BuildFact_v01`
- `NB_Venta_S2G_Facturacion_Aggregate_v01`
- `NB_Catalogos_SilverRefine_Productos_DQChecks_v01`
- `NB_Catalogos_GoldRefine_Producto360Snapshot_Snapshot_v01`

---

## Versionado
- Incrementar `_vNN` cuando haya cambios funcionales (reglas de negocio, llaves, lógica incremental, nuevos campos, etc.).
- No incrementar versión por cambios cosméticos (comentarios, formato), a menos que se requiera.
- Mantener un `CHANGELOG.md` a nivel repo para registrar: fecha, notebook, cambio, impacto y validaciones.
