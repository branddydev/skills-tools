# Skills Tools · La Biblioteca de Skills

Las dos skills con las que se alimenta la **Biblioteca de Skills**. Están en un repo público para que cualquiera pueda instalarlas sin permisos.

- **`limpiar-skill`**: crea una copia portable de tu skill, sin tocar la original. Pasa las keys, los MCPs, las rutas y los IDs personales a variables de entorno y genera un `SETUP.md` con la lista de requisitos y el paso a paso para conectarlos.
- **`subir-skill`**: rellena la ficha de la skill, la valida y la sube a la biblioteca.

> Este repo se sincroniza solo desde la biblioteca. No lo edites aquí: los cambios se hacen en `branddydev/skills-library`.

## Instalar

```bash
rm -rf /tmp/skills-tools && mkdir -p /tmp/skills-tools \
  && curl -sL https://codeload.github.com/branddydev/skills-tools/tar.gz/main | tar -xz -C /tmp/skills-tools --strip-components=1 \
  && mkdir -p ~/.claude/skills && rm -rf ~/.claude/skills/limpiar-skill ~/.claude/skills/subir-skill \
  && cp -R /tmp/skills-tools/skills/limpiar-skill /tmp/skills-tools/skills/subir-skill ~/.claude/skills/
```
