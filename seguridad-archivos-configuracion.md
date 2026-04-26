# 🔐 Manejo Seguro de `.env` y `appsettings` con IA (Copilot, Claude, Codex)

## 🎯 Objetivo

Evitar que herramientas de IA accedan a información sensible (`.env`, `appsettings.json`, certificados, etc.) y mantener una arquitectura segura tanto en desarrollo como en producción.

---

# ⚠️ Problema

Archivos como:

* `.env`
* `appsettings.json`
* `.p12`, `.pem`, `.key`

👉 **pueden ser leídos por herramientas de IA si están dentro del workspace abierto**

> `.gitignore` NO protege contra IA, solo contra Git.

---

# ✅ Estrategia General

✔ Sacar secretos fuera del proyecto
✔ Usar archivos `.example` dentro del repo
✔ Configurar ignores para IA
✔ Usar variables de entorno del sistema
✔ Usar Secret Managers en producción

---

# 📁 Estructura Recomendada

```bash
/proyecto
  /src
  .env.example
  appsettings.json
  .copilotignore
  .cursorignore
  .codeiumignore
  /.vscode/settings.json

/Users/tuusuario/secrets
  frontend.env
  backend.appsettings.json
```

---

# 🔒 1. Crear archivos ignore para IA

## `.copilotignore`, `.cursorignore`, `.codeiumignore`, `.codexignore`

```gitignore
.env
.env.*
*.env
appsettings.*.json
secrets/
*.p12
*.pem
*.key
*.crt
*.cer
```

---

# 🔒 2. Configuración para Claude Code

Crear:

```bash
.claude/settings.local.json
```

```json
{
  "permissions": {
    "deny": [
      "Read(.env)",
      "Read(.env.*)",
      "Read(**/*.env)",
      "Read(appsettings.*.json)",
      "Read(secrets/**)",
      "Read(**/*.p12)",
      "Read(**/*.pem)",
      "Read(**/*.key)"
    ]
  }
}
```

---

# 🔒 3. Ocultar archivos en VS Code

Archivo:

```bash
.vscode/settings.json
```

```json

  "files.exclude": {
    "**/.env": true,
    "**/.env.local": true,
    "**/.env.development": true,
    "**/.env.production": true,
    "**/*.env": true,

    "**/appsettings.json": true,
    "**/appsettings.Development.json": true,
    "**/appsettings.Production.json": true,

    "**/secrets": true
  },

  "search.exclude": {
    "**/.env": true,
    "**/.env.local": true,
    "**/.env.development": true,
    "**/.env.production": true,
    "**/*.env": true,

    "**/appsettings.json": true,
    "**/appsettings.Development.json": true,
    "**/appsettings.Production.json": true,

    "**/secrets": true
  }
```

---

# 🟢 4. Variables de entorno del sistema

Posiblemente va a tener que hacer esto primero:

### Abre tu shell config (Sonoma usa zsh por defecto)

```bash
nano ~/.zshrc
```

### Agrega:

```bash
export MI_VARIABLE="valor"
```

## Mac / Linux

```bash
export FRONTEND_ENV_PATH=/Users/tuusuario/secrets/frontend.env
export APPSETTINGS_PATH=/Users/tuusuario/secrets/appsettings.secrets.json
```

## Windows (PowerShell)

```powershell
setx FRONTEND_ENV_PATH "C:\secrets\frontend.env"
setx APPSETTINGS_PATH "C:\secrets\appsettings.secrets.json"
```

---

# 🟢 5. Next.js leyendo `.env` externo

Instalar dotenv:

```bash
npm install dotenv
```

## `next.config.mjs`

```js
import dotenv from "dotenv";

dotenv.config({
  path: process.env.FRONTEND_ENV_PATH,
});

export default {
  reactStrictMode: true,
};
```

## Uso en servidor

```ts
const apiUrl = process.env.API_URL;
```

> ⚠️ NO usar variables en Client Components

---

# 🟢 6. .NET leyendo config externo

## `Program.cs`

```csharp
var builder = WebApplication.CreateBuilder(args);

// appsettings externo
var externalAppSettings = Environment.GetEnvironmentVariable("APPSETTINGS_PATH");

if (!string.IsNullOrEmpty(externalAppSettings) && File.Exists(externalAppSettings))
{
    builder.Configuration.AddJsonFile(externalAppSettings, optional: true, reloadOnChange: true);
}

// .env externo
var envPath = Environment.GetEnvironmentVariable("ENV_FILE_PATH");

if (!string.IsNullOrEmpty(envPath) && File.Exists(envPath))
{
    foreach (var line in File.ReadAllLines(envPath))
    {
        if (string.IsNullOrWhiteSpace(line) || line.StartsWith("#"))
            continue;

        var parts = line.Split('=', 2);
        if (parts.Length == 2)
        {
            Environment.SetEnvironmentVariable(parts[0].Trim(), parts[1].Trim());
        }
    }
}
```

---

# 📄 7. Ejemplo `.env`

```env
API_URL=https://api.midominio.com
AZURE_KEYVAULT_URL=https://mi-vault.vault.azure.net/
JWT_SECRET=super_secreto
```

---

# 📄 8. Ejemplo `.env.example`

```env
API_URL=https://api.example.com
AZURE_KEYVAULT_URL=https://example-vault.vault.azure.net/
JWT_SECRET=your_secret_here
```

---

# 🔥 9. Buenas prácticas clave

✔ Nunca subir `.env` reales
✔ Mantener secretos fuera del workspace
✔ Rotar claves si se exponen

---

# 🧠 Regla de oro

> ⚠️ Si el archivo está dentro del proyecto abierto → la IA podría leerlo.

---

# 🚀 Nivel PRO (Producción)

Backend (.NET):

* Azure Key Vault
* Managed Identity

Frontend (Next.js):

* Variables en hosting (Vercel / Azure)

---

# ✅ Resumen

| Técnica            | Seguridad |
| ------------------ | --------- |
| `.gitignore`       | ❌         |
| `.env` en proyecto | ⚠️        |
| `.env` fuera       | ✅         |
| ignores IA         | ✅         |
| Key Vault          | 🔥        |

---

# 🔐 Conclusión

Este setup:

* Protege contra IA
* Escala a producción
* Sigue buenas prácticas reales

---
