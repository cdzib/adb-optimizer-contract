# adb-optimizer-contract

Contrato de datos compartido por los dos clientes de ADB Optimizer:

- **Adb-Optimizer-Android** — app Android (Kotlin/Compose).
- **Adb-Optimizer-Desktop** — host de escritorio (Electron/React).

No contiene código: solo datos y los fixtures que mantienen a los dos parsers
alineados. Es la única fuente de verdad; ningún cliente guarda copia.

## Contenido

```
recipes/
├─ recipes.json      recetas estáticas: run + verify + undo
├─ actions.json      acciones parametrizadas por paquete
└─ schema.json       JSON Schema, validado en TS y en Kotlin
conformance/
├─ README.md
└─ cases/            dumpsys de referencia: ambos parsers deben dar lo mismo
i18n/
├─ es.json
└─ en.json
apps/
└─ protected-packages.json   paquetes que ninguna acción puede tocar
```

## Cómo lo consumen los clientes

Cada cliente monta este repo como submódulo en `shared/`:

```bash
git clone --recurse-submodules <url-del-cliente>
git submodule update --init --recursive   # si ya lo tenías clonado
```

- **Android**: `android/build.gradle.kts` busca `shared/recipes/recipes.json`
  (o `-PsharedDir=...`, o `ADB_OPTIMIZER_SHARED`).
- **Escritorio**: `findSharedDir()` sube desde el cwd hasta encontrar
  `shared/recipes/recipes.json`.
- **F-Droid**: el metadata del APK usa `submodules: true`.

## Versionado

Los clientes apuntan a un **commit** concreto (el submódulo fija el SHA). Los tags
marcan releases del contrato: `v1.0.0`, `v1.1.0`, …

Regla: los dos clientes se suben al **mismo tag** en la misma tanda. Un cambio de
receta que solo se aplica en un cliente es exactamente el desfase que este repo
existe para evitar.

## Cómo cambiar algo

1. Edita `recipes/*.json` y, si aplica, `schema.json`.
2. Añade `titleKey`/`summaryKey` en **ambos** `i18n/es.json` y `en.json`.
3. Declara `verify` y `undo` en cada paso; `undo: null` solo si es irreversible
   de forma explícita.
4. Ids únicos en kebab-case.
5. Si el cambio afecta a un parser, añade o ajusta un caso en `conformance/cases/`.
6. Tag nuevo y bump del submódulo en los dos clientes.

## Nota sobre URLs

`.gitmodules` de los clientes usa la URL `https://`, porque el buildserver de
F-Droid clona sin credenciales SSH. Para desarrollo local puedes sobreescribirla
sin tocar el repo versionado:

```bash
git config submodule.shared.url ../adb-optimizer-contract
```
