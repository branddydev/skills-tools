---
name: subir-skill
description: Sube (o actualiza) una skill de Claude a la Biblioteca de Skills compartida (repo skills-library). Copia la skill desde ~/.claude/skills u otra carpeta, rellena su ficha skill.yaml (parámetros, output de ejemplo, requisitos), limpia secretos, valida y hace commit + push para que aparezca en la web. Úsala cuando el usuario diga "sube mi skill X a la biblioteca", "publica esta skill", "comparte esta skill con los demás", "actualiza mi skill en la biblioteca" o "/subir-skill".
---

# Subir una skill a la Biblioteca

Tu trabajo es dejar `skills/<slug>/` lista en el repo **skills-library**, validada y subida.

## 0. Sitúate en el repo

- Si el directorio actual no tiene `schema/skill.schema.json`, usa `~/skills-library`. Si no existe, clónalo: `gh repo clone branddydev/skills-library ~/skills-library` (el repo es privado; si gh da 404, el usuario aún no ha aceptado la invitación de GitHub).
- `git pull --rebase` para traer las skills nuevas de los demás.
- `npm install` si no hay `node_modules`.

## 1. Encuentra la skill de origen

- Por defecto: `~/.claude/skills/<slug>/`. También puede estar en `<proyecto>/.claude/skills/<slug>/` o en la ruta que te dé el usuario.
- Si el usuario no dice cuál, lista `~/.claude/skills/` y pregúntale.
- **Si ya existe `skills/<slug>/` en el repo**, es una actualización: copia los archivos nuevos encima (sin borrar `skill.yaml` ni `examples/`) y sube la `version` (patch si es un arreglo, minor si añade cosas, major si cambian los inputs). Si el `author` es otra persona, añade al usuario a `contributors`.
- No toques `updated`: la pone sola una GitHub Action en cada push.

## 2. Crea la carpeta

```bash
npm run new -- <slug> --from <ruta-de-origen>
```

Copia la skill (la original no se toca), ajusta el `name` del frontmatter y genera un `skill.yaml` desde la plantilla, con el autor sacado de `gh api user`.

## 3. Rellena skill.yaml

Lee **todo** el SKILL.md y sus `references/` y `scripts/`, y deduce:

| Campo | Cómo sacarlo |
|---|---|
| `title`, `emoji`, `summary` | Del título y la descripción. El `summary` es una frase que diga qué consigue el usuario. |
| `category`, `subcategory` | La categoría de la lista cerrada y una subcategoría corta y libre (ej. "meta ads", "copywriting"). Antes mira las subcategorías que ya existen (`grep -h "^subcategory:" skills/*/skill.yaml \| sort -u`) y reutiliza una si encaja. |
| `tags` | De 3 a 6 tags en kebab-case. |
| `triggers` | De la `description` del frontmatter: las frases entre comillas y el `/comando`. |
| `inputs` | Todo lo que la skill pide al usuario o necesita para arrancar. Cada uno lleva `name` en snake_case, `type`, `required` y `description`, y `example` siempre que puedas. |
| `output` | Qué entrega (`format`) y dónde lo deja (`description`). |
| `requirements` | MCPs, CLIs, APIs y variables de entorno (**solo nombres**) que use, y otras skills de las que dependa. |
| `status` | `estable` si el usuario la usa en producción, `beta` si funciona pero se está puliendo y `experimental` si no. Si dudas, pregunta. |

**Pide al usuario un output de ejemplo real** para `example.output`: lo último que generó la skill, aunque sea un extracto. Si tiene capturas o archivos de ejemplo, cópialos a `examples/` y añádelos a `example.files`. No inventes outputs: si no hay ninguno, pídele que la ejecute una vez o redacta uno marcado como *"Ejemplo ilustrativo"*.

Enséñale al usuario la ficha resumida (título, categoría / subcategoría, summary, inputs y output) y espera su OK antes de subirla.

## 4. Hazla portable

La web es pública y la skill se va a instalar en otros ordenadores. Sigue la skill `limpiar-skill` (`skills/limpiar-skill/SKILL.md`), de los pasos 2 al 6, sobre la copia que acabas de crear:
- Inventario de dependencias (keys, MCPs, CLIs, rutas, IDs).
- Secretos y datos personales pasados a variables de entorno.
- Bloque "Antes de empezar" en el SKILL.md.
- `SETUP.md` con los requisitos y el paso a paso.
- `requirements` rellenado en el skill.yaml.

**Nunca modifiques la skill original**: solo la copia del repo.

## 5. Valida

```bash
npm run validate
```

Corrige hasta que salga `✓`. Los errores dicen el archivo y el campo exactos. Opcional: `npm run dev` y abre `http://localhost:3000/s/<slug>/` para ver cómo queda.

## 6. Súbela

```bash
git add skills/<slug>
git commit -m "skill: <slug> v<version>"
git push
```

Si `main` está protegida o el usuario prefiere revisión: crea la rama `skill/<slug>`, haz push y abre un PR con `gh pr create`.

Termina diciéndole al usuario que la web se actualiza en uno o dos minutos, y dale la ruta `/s/<slug>/`.
