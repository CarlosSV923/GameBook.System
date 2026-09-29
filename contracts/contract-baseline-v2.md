# GB-002-CONTRACTS-v2 — enmienda operativa de persistencia

Esta referencia conserva íntegra la línea base
[`GB-002-CONTRACTS-v1`](contract-baseline.md): sus seis artefactos y huellas
SHA-256 siguen siendo evidencia histórica de la revisión original. La única
enmienda es
[`persistence-access-amendment.md`](persistence-access-amendment.md), acordada
en `GB-003.02` tras comprobar las limitaciones de roles Neon.

Para implementar, leer los seis artefactos de v1 **y después** aplicar la
enmienda donde trate roles, conexiones y migraciones. Ante una discrepancia
en ese ámbito, prevalece la enmienda. Los contratos OpenAPI, JWT, RAWG,
modelos lógicos y errores de v1 no cambian.

La huella SHA-256 de `persistence-access-amendment.md` es
`884471DF75ADD16CA4A7628DCF609ADBE2F13CCCB5001F1AD53D7A2EBD2288FF`.
Las seis huellas de v1 fueron comprobadas de nuevo sin cambios.

**Endurecimiento pendiente:** v2 describe el estado comprobado en
`GB-003.02`, no el objetivo final de permisos migradores. `GB-003.06` debe
probar migradores SQL limitados antes de configurar los secretos de Actions.
Si se verifica, emitir una revisión contractual v3; si falla, solicitar una
decisión del autor. No interpretar v2 como autorización para colocar los
migradores antiguos con `neon_superuser` en GitHub Actions.

No se ejecutaron migraciones de aplicación ni se creó código por emitir esta
referencia. El estado y la evidencia están en [`../tasks/mvp.md`](../tasks/mvp.md).
