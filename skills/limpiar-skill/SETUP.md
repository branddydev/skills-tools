# Cómo configurar limpiar-skill

Necesitas acceso al repo de la biblioteca y tres programas. Hazlo una vez.

## Requisitos

- [ ] **Acceso al repo** `branddydev/skills-library` (privado)
- [ ] **git**
- [ ] **GitHub CLI (`gh`)** con sesión iniciada
- [ ] **Node.js 20 o superior**

## Paso a paso

### 1. Acceso al repo

Pide a Manuel que te invite con tu usuario de GitHub y acepta la invitación que te llega por email (o en https://github.com/notifications).

### 2. git, gh y Node

```bash
brew install git gh node
```

### 3. Inicia sesión en GitHub

```bash
gh auth login
```

Elige GitHub.com → HTTPS → Login with a web browser.

## Comprobación final

```bash
gh repo view branddydev/skills-library --json name -q .name   # debe responder: skills-library
node -v                                                       # v20 o superior
```
