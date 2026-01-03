# Configuración de IDE y Herramientas

## Overview
Guía para configurar tu IDE y herramientas de desarrollo para maximizar productividad trabajando con Bolsillo Flow API.

---

## Visual Studio Code (Recomendado)

### Instalación

**Descargar desde:** [code.visualstudio.com](https://code.visualstudio.com/)

**O vía Homebrew (macOS):**
```bash
brew install --cask visual-studio-code
```

---

### Extensiones Esenciales

#### 1. ESLint

**ID:** `dbaeumer.vscode-eslint`

**Funcionalidad:**
- Linting en tiempo real
- Auto-fix al guardar
- Highlight de errores

**Configuración:**

`.vscode/settings.json`:
```json
{
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact"
  ],
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true
  }
}
```

---

#### 2. Prettier - Code Formatter

**ID:** `esbenp.prettier-vscode`

**Funcionalidad:**
- Formateo automático
- Consistencia de estilo

**Configuración:**

`.vscode/settings.json`:
```json
{
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.formatOnSave": true,
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

**`.prettierrc`:**
```json
{
  "semi": true,
  "trailingComma": "all",
  "singleQuote": true,
  "printWidth": 100,
  "tabWidth": 2,
  "arrowParens": "avoid"
}
```

---

#### 3. Prisma

**ID:** `Prisma.prisma`

**Funcionalidad:**
- Syntax highlighting para `.prisma`
- Autocompletado de modelos
- Formateo de schema
- Integración con Prisma CLI

**Shortcuts:**
- `Cmd/Ctrl + Shift + P` → "Prisma: Format"

---

#### 4. TypeScript Vue Plugin (Volar)

**ID:** `Vue.vscode-typescript-vue-plugin`

**Funcionalidad:**
- TypeScript intellisense mejorado
- Validación de tipos

---

#### 5. Jest Runner

**ID:** `firsttris.vscode-jest-runner`

**Funcionalidad:**
- Ejecutar tests individuales
- Debug de tests
- Ver coverage en editor

**Shortcuts:**
- Hover sobre test → "Run" o "Debug"

---

#### 6. GitLens

**ID:** `eamodio.gitlens`

**Funcionalidad:**
- Blame inline
- Historial de archivos
- Comparación de commits

---

#### 7. Path Intellisense

**ID:** `christian-kohler.path-intellisense`

**Funcionalidad:**
- Autocompletado de rutas de archivos

---

#### 8. REST Client

**ID:** `humao.rest-client`

**Funcionalidad:**
- Probar endpoints sin salir de VS Code
- Sintaxis simple para requests HTTP

**Ejemplo de uso:**

`requests.http`:
```http
### Health Check
GET http://localhost:3000/api/health

### Get All Categories
GET http://localhost:3000/api/categories

### Create Category
POST http://localhost:3000/api/categories
Content-Type: application/json

{
  "name": "Transporte",
  "description": "Gastos de movilización"
}
```

---

#### 9. Error Lens

**ID:** `usernamehw.errorlens`

**Funcionalidad:**
- Muestra errores inline
- Highlight más visible

---

#### 10. Import Cost

**ID:** `wix.vscode-import-cost`

**Funcionalidad:**
- Muestra tamaño de imports
- Útil para optimización

---

### Archivo de Extensiones Recomendadas

**`.vscode/extensions.json`:**

```json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "Prisma.prisma",
    "firsttris.vscode-jest-runner",
    "eamodio.gitlens",
    "christian-kohler.path-intellisense",
    "humao.rest-client",
    "usernamehw.errorlens",
    "wix.vscode-import-cost",
    "ms-vscode.vscode-typescript-next"
  ]
}
```

**Al abrir el proyecto, VS Code preguntará si instalar estas extensiones.**

---

### Configuración del Workspace

**`.vscode/settings.json`:**

```json
{
  // Editor
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": true,
    "source.organizeImports": true
  },
  "editor.tabSize": 2,
  "editor.rulers": [100],
  "editor.bracketPairColorization.enabled": true,
  "editor.guides.bracketPairs": true,

  // TypeScript
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true,
  "typescript.preferences.importModuleSpecifier": "non-relative",

  // Files
  "files.exclude": {
    "**/node_modules": true,
    "**/dist": true,
    "**/.git": true,
    "**/coverage": true
  },
  "files.watcherExclude": {
    "**/node_modules/**": true,
    "**/dist/**": true
  },

  // Search
  "search.exclude": {
    "**/node_modules": true,
    "**/dist": true,
    "**/coverage": true,
    "**/.git": true
  },

  // ESLint
  "eslint.validate": [
    "javascript",
    "javascriptreact",
    "typescript",
    "typescriptreact"
  ],

  // Prisma
  "[prisma]": {
    "editor.defaultFormatter": "Prisma.prisma"
  },

  // JSON
  "[json]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },

  // Markdown
  "[markdown]": {
    "editor.wordWrap": "on"
  }
}
```

---

### Snippets Personalizados

**`.vscode/typescript.json`:**

```json
{
  "NestJS Controller": {
    "prefix": "nest-controller",
    "body": [
      "import { Controller, Get, Post, Put, Delete, Body, Param } from '@nestjs/common';",
      "import { ${1:Resource}Service } from './${1/(.)/${1:/downcase}/}.service';",
      "import { Create${1}Dto, Update${1}Dto } from './dto';",
      "",
      "@Controller('${1/(.)/${1:/downcase}/}s')",
      "export class ${1}Controller {",
      "  constructor(private readonly ${1/(.)/${1:/downcase}/}Service: ${1}Service) {}",
      "",
      "  @Get()",
      "  findAll() {",
      "    return this.${1/(.)/${1:/downcase}/}Service.findAll();",
      "  }",
      "",
      "  @Get(':id')",
      "  findOne(@Param('id') id: string) {",
      "    return this.${1/(.)/${1:/downcase}/}Service.findOne(id);",
      "  }",
      "",
      "  @Post()",
      "  create(@Body() dto: Create${1}Dto) {",
      "    return this.${1/(.)/${1:/downcase}/}Service.create(dto);",
      "  }",
      "",
      "  @Put(':id')",
      "  update(@Param('id') id: string, @Body() dto: Update${1}Dto) {",
      "    return this.${1/(.)/${1:/downcase}/}Service.update(id, dto);",
      "  }",
      "",
      "  @Delete(':id')",
      "  remove(@Param('id') id: string) {",
      "    return this.${1/(.)/${1:/downcase}/}Service.remove(id);",
      "  }",
      "}"
    ]
  },
  "NestJS Service": {
    "prefix": "nest-service",
    "body": [
      "import { Injectable, NotFoundException } from '@nestjs/common';",
      "import { PrismaService } from '../prisma/prisma.service';",
      "import { Create${1:Resource}Dto, Update${1}Dto } from './dto';",
      "",
      "@Injectable()",
      "export class ${1}Service {",
      "  constructor(private prisma: PrismaService) {}",
      "",
      "  async findAll() {",
      "    return this.prisma.${1/(.)/${1:/downcase}/}.findMany();",
      "  }",
      "",
      "  async findOne(id: string) {",
      "    const ${1/(.)/${1:/downcase}/} = await this.prisma.${1/(.)/${1:/downcase}/}.findUnique({",
      "      where: { id }",
      "    });",
      "",
      "    if (!${1/(.)/${1:/downcase}/}) {",
      "      throw new NotFoundException('${1} not found');",
      "    }",
      "",
      "    return ${1/(.)/${1:/downcase}/};",
      "  }",
      "",
      "  async create(dto: Create${1}Dto) {",
      "    return this.prisma.${1/(.)/${1:/downcase}/}.create({",
      "      data: dto",
      "    });",
      "  }",
      "",
      "  async update(id: string, dto: Update${1}Dto) {",
      "    await this.findOne(id);",
      "    return this.prisma.${1/(.)/${1:/downcase}/}.update({",
      "      where: { id },",
      "      data: dto",
      "    });",
      "  }",
      "",
      "  async remove(id: string) {",
      "    await this.findOne(id);",
      "    return this.prisma.${1/(.)/${1:/downcase)/}.delete({",
      "      where: { id }",
      "    });",
      "  }",
      "}"
    ]
  }
}
```

---

### Tasks (Build, Test)

**`.vscode/tasks.json`:**

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "npm: start:dev",
      "type": "npm",
      "script": "start:dev",
      "problemMatcher": [],
      "isBackground": true,
      "presentation": {
        "reveal": "always",
        "panel": "new"
      }
    },
    {
      "label": "npm: test",
      "type": "npm",
      "script": "test",
      "problemMatcher": []
    },
    {
      "label": "npm: test:watch",
      "type": "npm",
      "script": "test:watch",
      "problemMatcher": [],
      "isBackground": true
    },
    {
      "label": "Prisma: Generate",
      "type": "shell",
      "command": "npx prisma generate",
      "problemMatcher": []
    },
    {
      "label": "Prisma: Studio",
      "type": "shell",
      "command": "npx prisma studio",
      "isBackground": true
    }
  ]
}
```

**Uso:** `Cmd/Ctrl + Shift + P` → "Tasks: Run Task"

---

### Launch Configurations (Debug)

**`.vscode/launch.json`:**

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug NestJS",
      "runtimeExecutable": "npm",
      "runtimeArgs": ["run", "start:debug"],
      "console": "integratedTerminal",
      "restart": true,
      "protocol": "inspector",
      "skipFiles": ["<node_internals>/**"]
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Jest: Current File",
      "program": "${workspaceFolder}/node_modules/.bin/jest",
      "args": ["${fileBasenameNoExtension}", "--config", "jest.config.js"],
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen",
      "windows": {
        "program": "${workspaceFolder}/node_modules/jest/bin/jest"
      }
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Jest: All Tests",
      "program": "${workspaceFolder}/node_modules/.bin/jest",
      "args": ["--runInBand", "--config", "jest.config.js"],
      "console": "integratedTerminal",
      "internalConsoleOptions": "neverOpen"
    }
  ]
}
```

**Uso:** `F5` para iniciar debug

---

## WebStorm / IntelliJ IDEA

### Configuración

1. **Open Project:** Abrir carpeta del proyecto
2. **Node Interpreter:** Settings → Languages & Frameworks → Node.js → Configure
3. **TypeScript:** Settings → Languages & Frameworks → TypeScript → Enable TypeScript language service
4. **Prisma:** Plugin ya incluido en versiones recientes

---

### Plugins Recomendados

1. **Prisma**: Soporte para `.prisma` files
2. **.env files support**: Syntax para `.env`
3. **Rainbow Brackets**: Colorear brackets
4. **Key Promoter X**: Aprender shortcuts

---

### Run Configurations

**Crear nueva config:**
1. Run → Edit Configurations
2. Add new → npm
3. Scripts: `start:dev`
4. Name: "NestJS Dev Server"

---

## Postman / Insomnia

### Postman Setup

#### 1. Importar Collection

**Crear `postman_collection.json`:**

```json
{
  "info": {
    "name": "Bolsillo Flow API",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "Health",
      "request": {
        "method": "GET",
        "header": [],
        "url": {
          "raw": "{{baseUrl}}/health",
          "host": ["{{baseUrl}}"],
          "path": ["health"]
        }
      }
    },
    {
      "name": "Categories",
      "item": [
        {
          "name": "List All",
          "request": {
            "method": "GET",
            "url": "{{baseUrl}}/categories"
          }
        },
        {
          "name": "Create",
          "request": {
            "method": "POST",
            "header": [{"key": "Content-Type", "value": "application/json"}],
            "body": {
              "mode": "raw",
              "raw": "{\n  \"name\": \"Nueva Categoría\",\n  \"description\": \"Descripción\"\n}"
            },
            "url": "{{baseUrl}}/categories"
          }
        }
      ]
    }
  ],
  "variable": [
    {
      "key": "baseUrl",
      "value": "http://localhost:3000/api"
    }
  ]
}
```

#### 2. Importar en Postman

1. File → Import
2. Seleccionar `postman_collection.json`
3. Collection aparecerá en sidebar

---

### Environment Variables

**Crear environment "Local":**
```json
{
  "baseUrl": "http://localhost:3000/api",
  "token": ""
}
```

---

## Git Configuration

### .gitignore

**Ya debe estar en el proyecto, verificar incluye:**

```gitignore
# Dependencies
node_modules/
.pnp/
.pnp.js

# Testing
coverage/
.nyc_output/

# Build
dist/
build/

# Environment
.env
.env.local
.env.*.local

# Logs
logs/
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# IDE
.vscode/
!.vscode/settings.json
!.vscode/tasks.json
!.vscode/launch.json
!.vscode/extensions.json
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Prisma
*.db
*.db-journal

# Misc
.cache/
```

---

### Git Hooks (Husky)

**Instalar:**
```bash
npm install --save-dev husky lint-staged
npx husky install
```

**Pre-commit hook:**
```bash
npx husky add .husky/pre-commit "npm run lint-staged"
```

**`package.json`:**
```json
{
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write",
      "git add"
    ],
    "*.{json,md}": [
      "prettier --write",
      "git add"
    ]
  }
}
```

---

## Terminal Mejorado

### Oh My Zsh (macOS/Linux)

**Instalar:**
```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

**Plugins recomendados en `~/.zshrc`:**
```bash
plugins=(
  git
  node
  npm
  docker
  vscode
)
```

---

### Aliases Útiles

**Agregar a `~/.zshrc` o `~/.bashrc`:**

```bash
# Proyecto
alias bf="cd ~/projects/bolsillo-flow-api"

# NPM
alias ni="npm install"
alias nrs="npm run start:dev"
alias nrt="npm run test"
alias nrtw="npm run test:watch"

# Prisma
alias pg="npx prisma generate"
alias pm="npx prisma migrate dev"
alias ps="npx prisma studio"

# Git
alias gs="git status"
alias ga="git add"
alias gc="git commit"
alias gp="git push"
alias gpl="git pull"

# Docker (si lo usas)
alias dcu="docker-compose up"
alias dcd="docker-compose down"
```

**Recargar config:**
```bash
source ~/.zshrc
```

---

## Herramientas CLI Adicionales

### HTTPie (Alternativa a curl)

**Instalar:**
```bash
# macOS
brew install httpie

# Linux
apt install httpie
```

**Uso:**
```bash
# GET
http localhost:3000/api/categories

# POST
http POST localhost:3000/api/categories name="Transporte" description="Movilización"
```

---

### jq (Parse JSON)

**Instalar:**
```bash
brew install jq
```

**Uso:**
```bash
curl localhost:3000/api/categories | jq '.[] | {id, name}'
```

---

## Shortcuts Esenciales

### VS Code

| Acción | macOS | Windows/Linux |
|--------|-------|---------------|
| Command Palette | `Cmd+Shift+P` | `Ctrl+Shift+P` |
| Quick Open | `Cmd+P` | `Ctrl+P` |
| Toggle Terminal | `Ctrl+` ` | `Ctrl+` ` |
| Find in Files | `Cmd+Shift+F` | `Ctrl+Shift+F` |
| Go to Definition | `F12` | `F12` |
| Rename Symbol | `F2` | `F2` |
| Format Document | `Shift+Alt+F` | `Shift+Alt+F` |
| Multi-cursor | `Cmd+D` | `Ctrl+D` |
| Comment Line | `Cmd+/` | `Ctrl+/` |

---

## Referencias

- [VS Code Documentation](https://code.visualstudio.com/docs)
- [NestJS DevTools](https://docs.nestjs.com/devtools/overview)
- [Prisma Studio](https://www.prisma.io/docs/concepts/components/prisma-studio)
- [Setup: Development](./development-setup.md)