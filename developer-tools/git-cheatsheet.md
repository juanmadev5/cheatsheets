# Git — Cheatsheet

---

## 📑 Índice

- [Configuración inicial](#️-configuración-inicial)
- [Repositorio — init, clone](#-repositorio)
- [Stage & Commit — add, commit, amend](#-stage--commit)
- [Firma de commits (SSH) — gpg.format ssh](#-firma-de-commits-ssh)
- [Historial y diferencias — log, diff, blame, show](#-historial-y-diferencias)
- [Ramas — branch, switch, delete, rename](#-ramas-branches)
- [Worktrees — múltiples ramas en paralelo](#-worktrees)
- [Merge & Rebase — merge, rebase interactivo, cherry-pick](#-merge--rebase)
- [Tags — crear, subir, eliminar](#️-tags)
- [Remotos — fetch, pull, push, force-with-lease](#-remotos)
- [Stash — guardar, aplicar, listar, limpiar](#-stash)
- [Deshacer cambios — restore, reset, revert, clean](#️-deshacer-cambios)
- [Búsqueda — grep, log -S, bisect](#-búsqueda)
- [Utilidades — shortlog, reflog, archive, gc](#️-utilidades)
- [.gitignore — patrones](#--gitignore--patrones)
- [.gitattributes — normalizar line endings, binarios, linguist](#-gitattributes--normalizar-el-repo)
- [Flujo de trabajo típico — feature branches](#-flujo-de-trabajo-típico)
- [Buenas prácticas — Conventional Commits](#-buenas-prácticas)

---

## ⚙️ Configuración inicial

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"
git config --global core.editor "code --wait"   # VS Code como editor
git config --global init.defaultBranch main
git config --list                                # Ver toda la configuración
git config --global alias.lg "log --oneline --graph --decorate --all"
```

---

## 📁 Repositorio

| Comando | Descripción |
|---|---|
| `git init` | Inicializar repositorio local |
| `git init <dir>` | Crear repo en directorio nuevo |
| `git clone <url>` | Clonar repositorio remoto |
| `git clone <url> <dir>` | Clonar en carpeta específica |
| `git clone --depth 1 <url>` | Clonar solo el último commit (shallow) |
| `git clone -b <branch> <url>` | Clonar rama específica |

---

## 📸 Stage & Commit

| Comando | Descripción |
|---|---|
| `git status` | Ver estado del working tree |
| `git status -s` | Estado en formato corto |
| `git add <file>` | Agregar archivo al stage |
| `git add .` | Agregar todo al stage |
| `git add -p` | Agregar cambios interactivamente (por hunks) |
| `git restore --staged <file>` | Sacar archivo del stage |
| `git restore <file>` | Descartar cambios en working tree |
| `git commit -m "mensaje"` | Crear commit |
| `git commit -am "mensaje"` | Stage + commit de archivos tracked |
| `git commit --amend` | Modificar el último commit |
| `git commit --amend --no-edit` | Enmendar sin cambiar el mensaje |

---

## 🔏 Firma de commits (SSH)

Desde Git 2.34+ se puede firmar commits/tags con una clave SSH en vez de GPG — reutiliza la misma clave que ya usás para autenticarte.

```bash
git config --global gpg.format ssh
git config --global user.signingkey ~/.ssh/id_ed25519.pub
git config --global commit.gpgsign true       # Firmar todos los commits automáticamente
git config --global tag.gpgsign true          # Firmar todos los tags automáticamente

git commit -S -m "mensaje"                    # Firmar un commit puntual (si gpgsign no es global)
git log --show-signature -1                    # Verificar la firma de un commit
```

---

## 🔍 Historial y diferencias

| Comando | Descripción |
|---|---|
| `git log` | Ver historial |
| `git log --oneline` | Historial compacto |
| `git log --oneline --graph --all` | Historial con árbol de ramas |
| `git log -n 10` | Últimos 10 commits |
| `git log --author="nombre"` | Commits de un autor |
| `git log --since="2 weeks ago"` | Commits recientes |
| `git log -- <file>` | Historial de un archivo |
| `git show <commit>` | Ver contenido de un commit |
| `git diff` | Cambios no staged |
| `git diff --staged` | Cambios en stage |
| `git diff <branch1>..<branch2>` | Diferencia entre ramas |
| `git blame <file>` | Ver quién cambió cada línea |

---

## 🌿 Ramas (Branches)

| Comando | Descripción |
|---|---|
| `git branch` | Listar ramas locales |
| `git branch -a` | Listar ramas locales y remotas |
| `git branch <name>` | Crear rama |
| `git switch <name>` | Cambiar de rama |
| `git switch -c <name>` | Crear y cambiar de rama |
| `git branch -d <name>` | Eliminar rama (safe) |
| `git branch -D <name>` | Forzar eliminación de rama |
| `git branch -m <old> <new>` | Renombrar rama |
| `git branch --merged` | Ramas ya mergeadas |
| `git branch --no-merged` | Ramas sin mergear |

> `git checkout` sigue funcionando, pero `git switch` y `git restore` son los comandos modernos (Git 2.23+).

---

## 🌳 Worktrees

Permite tener varias ramas checkouteadas al mismo tiempo, cada una en su propio directorio, compartiendo el mismo `.git`. Ideal para un hotfix urgente sin perder el estado de lo que estás trabajando (alternativa a `stash`) o para correr agentes/tests en paralelo sobre distintas ramas.

| Comando | Descripción |
|---|---|
| `git worktree add ../hotfix main` | Crear worktree nuevo desde `main` en `../hotfix` |
| `git worktree add -b fix/bug-123 ../fix-123` | Crear worktree + rama nueva |
| `git worktree list` | Listar worktrees activos |
| `git worktree remove ../hotfix` | Eliminar un worktree |
| `git worktree prune` | Limpiar referencias a worktrees eliminados manualmente |

---

## 🔀 Merge & Rebase

| Comando | Descripción |
|---|---|
| `git merge <branch>` | Mergear rama en la actual |
| `git merge --no-ff <branch>` | Merge forzando commit de merge |
| `git merge --squash <branch>` | Squash de todos los commits |
| `git merge --abort` | Cancelar merge en curso |
| `git rebase <branch>` | Rebase sobre otra rama |
| `git rebase -i HEAD~3` | Rebase interactivo (últimos 3 commits) |
| `git rebase --continue` | Continuar rebase tras resolver conflicto |
| `git rebase --abort` | Cancelar rebase |
| `git cherry-pick <commit>` | Aplicar commit específico a la rama actual |
| `git cherry-pick <c1>..<c2>` | Aplicar rango de commits |

### Rebase interactivo — opciones

| Opción | Descripción |
|---|---|
| `pick` | Mantener commit tal cual |
| `reword` | Cambiar mensaje del commit |
| `edit` | Pausar para modificar el commit |
| `squash` | Fusionar con el commit anterior |
| `fixup` | Como squash, descarta el mensaje |
| `drop` | Eliminar el commit |

---

## 🏷️ Tags

| Comando | Descripción |
|---|---|
| `git tag` | Listar tags |
| `git tag <name>` | Crear tag ligero |
| `git tag -a <name> -m "msg"` | Crear tag anotado |
| `git tag -a <name> <commit>` | Taggear commit específico |
| `git show <tag>` | Ver detalles del tag |
| `git tag -d <name>` | Eliminar tag local |
| `git push origin <tag>` | Subir tag al remoto |
| `git push origin --tags` | Subir todos los tags |
| `git push origin --delete <tag>` | Eliminar tag remoto |

---

## 🌐 Remotos

| Comando | Descripción |
|---|---|
| `git remote -v` | Ver remotos configurados |
| `git remote add origin <url>` | Agregar remoto |
| `git remote rename <old> <new>` | Renombrar remoto |
| `git remote remove <name>` | Eliminar remoto |
| `git remote set-url origin <url>` | Cambiar URL del remoto |
| `git fetch` | Traer cambios sin integrar |
| `git fetch --prune` | Traer y limpiar ramas remotas eliminadas |
| `git pull` | Fetch + merge |
| `git pull --rebase` | Fetch + rebase (historial más limpio) |
| `git push` | Subir commits al remoto |
| `git push -u origin <branch>` | Subir rama y setear upstream |
| `git push --force-with-lease` | Force push seguro (verifica upstream) |
| `git push origin --delete <branch>` | Eliminar rama remota |

---

## 📦 Stash

| Comando | Descripción |
|---|---|
| `git stash` | Guardar cambios sin commitear |
| `git stash push -m "descripción"` | Stash con mensaje |
| `git stash -u` | Incluir archivos untracked |
| `git stash list` | Ver lista de stashes |
| `git stash pop` | Aplicar y eliminar el último stash |
| `git stash apply stash@{2}` | Aplicar stash específico (sin eliminarlo) |
| `git stash drop stash@{0}` | Eliminar stash específico |
| `git stash clear` | Eliminar todos los stashes |
| `git stash branch <branch>` | Crear rama desde un stash |

---

## ↩️ Deshacer cambios

| Comando | Descripción |
|---|---|
| `git restore <file>` | Descartar cambios en working tree |
| `git restore --staged <file>` | Sacar del stage (sin perder cambios) |
| `git reset HEAD~1` | Deshacer último commit, mantener cambios en stage |
| `git reset --soft HEAD~1` | Deshacer commit, mantener cambios staged |
| `git reset --mixed HEAD~1` | Deshacer commit, cambios quedan unstaged (default) |
| `git reset --hard HEAD~1` | Deshacer commit y **descartar cambios** |
| `git revert <commit>` | Crear commit inverso (no reescribe historia) |
| `git revert HEAD` | Revertir último commit |
| `git clean -fd` | Eliminar archivos y dirs no tracked |
| `git clean -n` | Ver qué eliminaría `git clean` (dry run) |

> ⚠️ `reset --hard` y `clean` son destructivos. Usar con cuidado.

---

## 🔎 Búsqueda

| Comando | Descripción |
|---|---|
| `git grep "texto"` | Buscar en los archivos del repo |
| `git log -S "texto"` | Commits que agregaron/quitaron ese texto |
| `git log --grep="texto"` | Buscar en mensajes de commit |
| `git bisect start` | Iniciar búsqueda binaria de bug |
| `git bisect bad` | Marcar commit actual como malo |
| `git bisect good <commit>` | Marcar commit conocido como bueno |
| `git bisect reset` | Finalizar bisect |

---

## 🛠️ Utilidades

| Comando | Descripción |
|---|---|
| `git shortlog -sn` | Commits por autor |
| `git archive --format=zip HEAD > out.zip` | Exportar repo como ZIP |
| `git reflog` | Historial de movimientos de HEAD |
| `git gc` | Garbage collection (optimizar repo) |
| `git count-objects -vH` | Ver tamaño del repositorio |
| `git ls-files` | Listar archivos tracked |
| `git ls-files --others --exclude-standard` | Listar archivos untracked |

---

## 📄 `.gitignore` — patrones

```gitignore
# Archivos específicos
.env
*.log

# Directorios
node_modules/
dist/
.idea/
.vscode/

# Extensiones
*.class
*.jar

# Excepción (no ignorar aunque matchee regla anterior)
!important.log

# Todo en un directorio
logs/**

# Solo en la raíz
/config.local.json
```

> Generar `.gitignore` para tu stack: [gitignore.io](https://www.toptal.com/developers/gitignore)

---

## 📐 `.gitattributes` — normalizar el repo

```gitattributes
# Normalizar line endings (LF) para todos los archivos de texto
* text=auto eol=lf

# Forzar CRLF solo en scripts de Windows
*.bat text eol=crlf

# Tratar como binario (sin diff ni normalización)
*.png binary
*.jpg binary

# Excluir del linguist de GitHub (no cuenta como código del repo)
vendor/* linguist-vendored
docs/* linguist-documentation
```

---

## 🔁 Flujo de trabajo típico

```bash
# 1. Crear rama para feature
git switch -c feature/login

# 2. Trabajar y commitear
git add -p
git commit -m "feat: add login form"

# 3. Traer cambios de main antes de mergear
git fetch origin
git rebase origin/main

# 4. Subir rama
git push -u origin feature/login

# 5. Después del PR mergeado, limpiar
git switch main
git pull
git branch -d feature/login
```

---

## 💡 Buenas prácticas

- **Commits atómicos**: un commit = un cambio lógico. Más fácil de revertir y revisar.
- **Mensajes descriptivos**: seguir [Conventional Commits](https://www.conventionalcommits.org/): `feat:`, `fix:`, `chore:`, `refactor:`, `docs:`, `test:`.
- **Nunca reescribir historia pública**: no usar `reset --hard` ni `rebase` sobre ramas compartidas.
- **Usar `--force-with-lease`** en lugar de `--force` al hacer force push.
- **Branches de corta vida**: mergear y eliminar ramas frecuentemente para evitar divergencias grandes.
- **Revisar antes de commitear**: `git diff --staged` para ver exactamente qué va en el commit.
- **`.gitignore` desde el inicio**: agregarlo antes del primer commit.
- **`git pull --rebase`** para mantener historial lineal en vez de generar commits de merge innecesarios.
