---
name: limpiar-skill
description: Convierte una skill tuya en una versión PORTABLE que funcione en el ordenador de otra persona, sin tocar la original. Trabaja sobre una copia dentro del repo skills-library, detecta todo lo que depende de tu entorno (API keys, credenciales, MCPs, CLIs, rutas, IDs de Notion/Drive/ClickUp, cuentas), lo cambia por variables de entorno o configuración, y genera un SETUP.md con la lista de requisitos y el paso a paso para conectarlos. Úsala cuando el usuario diga "limpia mi skill X", "haz portable esta skill", "prepara esta skill para compartirla", "/limpiar-skill", o como paso previo a subir-skill.
---

# Limpiar una skill (hacerla portable)

Objetivo: que cualquiera que instale la skill en otro ordenador sepa **qué necesita** y **cómo conectarlo**, y que la skill funcione en cuanto lo tenga.

## Regla de oro: la original NO se toca

- Nunca edites, muevas ni borres nada en la carpeta original (`~/.claude/skills/<slug>/` o donde esté).
- Todo el trabajo se hace sobre una **copia** en `~/skills-library/skills/<slug>/`.
- Al empezar, toma la huella de la original y compruébala al terminar:
  ```bash
  ORIG=~/.claude/skills/<slug>
  find "$ORIG" -type f -not -name .DS_Store -print0 | sort -z | xargs -0 shasum | shasum   # antes
  # … trabajo …
  find "$ORIG" -type f -not -name .DS_Store -print0 | sort -z | xargs -0 shasum | shasum   # después: debe ser idéntica
  ```
  Si cambia, para y avisa al usuario.

## 1. Prepara la copia

1. Repo: si el directorio actual no tiene `schema/skill.schema.json`, usa `~/skills-library`. Si no existe, clónalo con `gh repo clone branddydev/skills-library ~/skills-library`. Después, `git pull --rebase` y `npm install` si falta `node_modules`.
2. Si **no existe** `skills/<slug>/`, crea la copia con `npm run new -- <slug> --from <ruta-original>`.
3. Si **ya existe**, es una actualización. Copia la original a `/tmp/limpiar-<slug>/` y lleva los cambios a `skills/<slug>/`, conservando su `skill.yaml`, su `SETUP.md` y su carpeta `examples/`.

## 2. Inventario de dependencias

Lee **todos** los archivos de la copia: `SKILL.md`, `references/`, `scripts/` y cualquier config. Busca:

| Qué | Pistas |
|---|---|
| **Secretos escritos a mano** | `sk-…`, `ghp_…`, `AIza…`, `xox…`, `ntn_…`, `Bearer …`, `password=`, cookies, URLs con `?key=` o `token=` |
| **Variables de entorno** | `$VAR`, `${VAR}`, `process.env.X`, `os.environ["X"]`, `getenv`, `.env` |
| **MCPs / conectores** | `mcp__<servidor>__<tool>`, "conector de…", "MCP de…". Ojo: los prefijos con UUID (`mcp__1a59c906-…__`) **cambian en cada instalación** |
| **CLIs y programas** | `gh`, `ffmpeg`, `yt-dlp`, `rclone`, `railway`, `python3`, `node`, `brew install`, `pip install`, `npm i`, imports de librerías |
| **Rutas personales** | `/Users/<nombre>/…`, `~/Documents/…`, carpetas de Drive/Dropbox, rutas absolutas en general |
| **IDs y cuentas personales** | IDs de bases de datos o páginas de Notion, listas de ClickUp, carpetas de Drive, webhooks, emails, teléfonos, nombres de clientes, workspaces, dominios propios |
| **Otras skills** | Menciones a otras skills que llama o de las que depende |
| **Servicios de pago** | APIs que cuestan dinero (APImart, OpenAI, Apify…). Apúntalo para avisar en el SETUP |

Haz una tabla: **dependencia · tipo · dónde aparece (archivo:línea) · qué harás con ella**.

## 3. Hazla portable (solo en la copia)

- **Secretos** → bórralos. El código los lee de una variable de entorno con un nombre claro (`APIMART_API_KEY`, `NOTION_TOKEN`), y nunca imprime su valor.
- **IDs, cuentas y rutas personales** → variables de entorno con el prefijo de la skill (`<SLUG_EN_MAYUS>_NOTION_DB_ID`, `<SLUG>_DRIVE_FOLDER_ID`). Si el valor solo aparece en las instrucciones, usa un marcador (`<NOTION_DB_ID>`) y explica en el SETUP cómo obtenerlo. Las rutas absolutas pasan a ser relativas a la carpeta de la skill o a `~`.
- **MCPs** → quita los prefijos con UUID y nombra el servicio: "usa la herramienta de búsqueda del MCP de Notion (`notion-search`)". Indica qué herramientas necesita de cada MCP.
- **Datos de clientes o de negocio** que no hacen falta para que funcione → generalízalos o quítalos.
- **Añade al principio del `SKILL.md`** (justo después del título) esta sección, adaptada:
  ```markdown
  ## Antes de empezar: comprueba los requisitos

  Comprueba en silencio que existe todo esto. Si falta algo, PARA, dile al usuario qué falta y guíale con el SETUP.md de esta skill antes de seguir:
  - Variables de entorno: `NOMBRE_1`, `NOMBRE_2` (comprueba que no estén vacías, sin mostrar su valor)
  - MCPs: <servicio> (herramientas: …)
  - Programas: `gh`, `ffmpeg` (`command -v …`)
  ```
- No cambies la lógica de la skill. Si para hacerla portable hay que cambiar cómo funciona, pregúntalo antes.

## 4. Escribe el SETUP.md

Crea `skills/<slug>/SETUP.md` a partir de `templates/skill/SETUP.md`:

1. **Lista de requisitos** con casillas, agrupados por tipo: API keys, MCPs, programas, cuentas o IDs, otras skills. Marca cuáles son de pago.
2. **Paso a paso para cada uno**, pensado para alguien que no sabe nada de tu entorno:
   - **API key:** dónde se consigue (URL exacta del panel) y cómo guardarla. Por defecto, en `~/.claude/settings.json` dentro de `"env": { "NOMBRE": "valor" }`; como alternativa, `export NOMBRE=…` en `~/.zshrc`.
   - **MCP:** cómo conectarlo. Si es un conector de claude.ai: Ajustes → Conectores. Si es local: el comando `claude mcp add …` exacto, o el bloque de `.mcp.json`. Añade qué permisos o páginas hay que compartir (por ejemplo, compartir la base de datos de Notion con la integración).
   - **Programa:** el comando de instalación (`brew install …`) y cómo comprobar que funciona.
   - **ID o configuración personal:** dónde encontrarlo (por ejemplo, el ID de Notion es la parte de la URL tras el nombre de la página) y en qué variable va.
3. **Comprobación final:** unos comandos para verificar que todo está listo (`echo ${NOMBRE:+ok}`, `command -v ffmpeg`, una llamada de prueba al MCP).

## 5. Rellena los requisitos en skill.yaml

En `requirements`:
- `tools`: MCPs y programas en lenguaje humano (`"MCP de Notion"`, `"ffmpeg"`).
- `env`: **solo los nombres** de las variables.
- `skills`: otras skills de las que depende.
- `notes`: `"Sigue el SETUP.md para configurarla."`, más cualquier aviso de coste.

## 6. Verifica

- Vuelve a buscar en la copia secretos, rutas `/Users/`, emails e IDs largos. No debe quedar nada personal.
- `npm run validate` debe salir con ✓.
- Comprueba la huella de la original: tiene que ser idéntica.

## 7. Enséñaselo al usuario

Resume en el chat:
- La **tabla de requisitos** (qué necesita alguien para usarla).
- Los **cambios** de la copia respecto a la original.
- Que su skill original **no se ha tocado** (huella idéntica).

Espera su OK. Después, si quiere subirla, continúa con la skill `subir-skill` desde el paso 3: rellena el resto de la ficha, valida y súbela.
