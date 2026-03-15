---
name: api-creator
description: Create a new Node.js + Express API project or add new modules to an existing one. Use this skill whenever the user wants to create an API, start a new backend, scaffold a Node.js project, or says things like "crear un api", "crea un api en node", "nuevo backend", "quiero un api", "new api project", "start a node project". Also use this skill when the user wants to add a new module or component to an existing API, such as "crea un módulo", "agrega un módulo de users", "nuevo componente", "create a module", "add a new module", "necesito un módulo de productos", or any variation involving adding business logic modules to an Express API. Always use this skill even if the user doesn't specifically mention "it-node-api-creator" or "create-node-component" — if they want a new API or a new module inside one, this is the way.
---

# api-creator

Scaffold a production-ready Node.js + Express API in seconds using the `it-node-api-creator` CLI. The generated project comes preconfigured with a professional startup banner, modular architecture, MongoDB/Mongoose, JWT auth, Winston logging, rate limiting, CORS, and more.

## Code Conventions

**All code must be written in English** — regardless of the language the user prompts in. This includes:
- Variable and function names (`getUserById`, not `obtenerUsuarioPorId`)
- File and folder names (`userController.js`, not `controladorUsuario.js`)
- Comments and JSDoc annotations
- Route paths (`/users`, not `/usuarios`)
- Database field names and schema definitions

**User-facing content can be in any language.** This means response messages, error texts, labels, buttons, and any text that end users will see can be in Spanish or another language as needed. For example:

```javascript
// ✅ Correct: English code, Spanish user-facing message
export const index = async (req, res) => {
    res.status(200).json({
        message: "Bienvenido al sistema",
        success: true
    });
};

// ❌ Wrong: Spanish variable/function names
export const obtenerTodos = async (req, res) => { ... };
```

## Part 1: Create a New API Project

### Step 1: Pre-flight Checks

Before creating anything, verify the current workspace state:

**1a. Check if an API project already exists**

Look for a `package.json` in the current workspace root. If it exists, check if it has Express-related dependencies (`express`, `mongoose`) or API-related scripts (`start`, `dev`). If a project is already there, inform the user and ask if they want to proceed in a subdirectory instead.

```bash
cat package.json 2>/dev/null
```

If there's already an API project, say something like: "It looks like there's already a Node.js project here. Want me to create the new API in a subfolder?"

**1b. Check if the CLI tools are available**

```bash
which create-it-api
which create-component
```

If `create-it-api` is not found, install it:
```bash
npm install -g it-node-api-creator
```

If `create-component` is not found, install it:
```bash
npm install -g create-node-component
```

### Step 2: Create the Project

Ask the user for a project name if they haven't provided one, then run:

```bash
create-it-api <project-name>
```

This command does the following automatically:
1. Creates the project directory
2. Copies the complete template (server, app, db, modules, middlewares, helpers, routes)
3. Creates `.env` from `.env.example`
4. Runs `npm install` to install all dependencies

Wait for the command to finish — it will install dependencies which takes a moment.

### Step 3: Post-Setup Guidance

After the project is created, guide the user through the next steps:

**Show the generated structure:**

```
<project-name>/
├── .env                  # Auto-created from .env.example
├── .env.example          # Environment variable template
├── .gitignore
├── CHANGELOG.md
├── LICENSE
├── README.md
├── package.json
├── server.js             # Entry point with startup banner & DB logic
├── app.js                # Express app configuration (middlewares, routes)
├── db.js                 # MongoDB/Mongoose connection
├── initLogger.js         # Winston logger initialization
└── src/
    ├── helpers/          # Global utilities (logger, token, etc.)
    ├── middlewares/      # Express middlewares (auth, rate-limit)
    ├── modules/          # Business logic by module
    │   └── home/         # Default module
    │       ├── controllers/
    │       ├── models/
    │       ├── routes/
    │       └── services/
    └── routes/
        └── v1/           # Versioned API route entry points
```

**Key files to configure:**

1. **`.env`** — Configure at minimum:
   - `ENVIRONMENT=DEVELOPMENT` (or `PRODUCTION`)
   - `DEFAULT_PORT=3001`
   - MongoDB credentials if needed (`MONGO_USER`, `MONGO_PASSWORD`, `MONGO_SERVER`, `MONGO_DB`)

2. **Start the API:**
   ```bash
   cd <project-name>
   npm run dev    # Development mode (nodemon, hot reload)
   npm start      # Production mode (node)
   ```

**Important behaviors:**
- In `DEVELOPMENT` mode, missing MongoDB config shows a warning but the server starts anyway — this is intentional so you can start building routes without a database.
- In `PRODUCTION` mode, missing MongoDB config prompts the user to confirm before continuing.

---

## Part 2: Add New Modules

Use `create-component` to scaffold new modules inside the API and then wire them into the route system.

### Step 1: Create the Module Structure

From inside the project root, run:

```bash
create-component <module-name> src/modules
```

This generates:

```
src/modules/<module-name>/
├── controllers/
│   └── <module-name>Controller.js    # (empty file)
├── models/
│   └── <module-name>Model.js         # (empty file)
├── routes/
│   └── <module-name>Routes.js        # (empty file)
└── services/
    └── <module-name>Service.js        # (empty file)
```

> **Important:** Always pass `src/modules` as the second argument — the default directory is `./components/` which doesn't match this project's structure.

### Step 2: Write Boilerplate Code in the Generated Files

The CLI creates empty files, so you need to populate them with working boilerplate. Follow these patterns from the existing `home` module:

**Routes file** (`src/modules/<name>/routes/<name>Routes.js`):
```javascript
import express from 'express';
const router = express.Router();

import * as Controller from '../controllers/<name>Controller.js';

router.get('/', Controller.index);

export default router;
```

**Controller file** (`src/modules/<name>/controllers/<name>Controller.js`):
```javascript
export const index = async (req, res) => {
    res.status(200).json({
        message: "Module <name> works!",
        success: true
    });
};
```

**Service file** (`src/modules/<name>/services/<name>Service.js`):
```javascript
// Business logic for <name> module
```

**Model file** (`src/modules/<name>/models/<name>Model.js`):
```javascript
import mongoose from 'mongoose';

const <name>Schema = new mongoose.Schema({
    // define fields here
}, {
    timestamps: true,
    versionKey: false
});

export default mongoose.model('<Name>', <name>Schema);
```

### Step 3: Register the Module Routes (CRITICAL)

This is the most important step. The module won't be accessible until its routes are registered. There are **two places** where routes get mounted, depending on the intent:

#### Option A: Versioned API route (recommended for most modules)

Register in `src/routes/v1/index.js`. This mounts the module under `/v1/<module-name>`:

```javascript
import { Router } from 'express';
import <name>Routes from '../../modules/<name>/routes/<name>Routes.js';

const router = Router();

// Add this line for each new module:
router.use('/<name>', <name>Routes);

export default router;
```

The result: endpoints become accessible at `/v1/<module-name>/...`

#### Option B: Root-level route (like the home module)

Register directly in `app.js` by adding an import and a `app.use()` call. This is how the `home` module works — it's mounted at `/`:

```javascript
// In app.js, add the import at the top:
import <name>Routes from './src/modules/<name>/routes/<name>Routes.js';

// Then mount it (BEFORE the notFoundHandler and errorHandler):
app.use('/<name>', <name>Routes);
```

> ⚠️ **Route registration order matters in `app.js`!** Always mount new routes BEFORE `notFoundHandler` and `errorHandler` — these are the last middleware handlers and they catch anything unmatched. If you add routes after them, the routes will never be reached.

### Route Registration Checklist

When adding a new module, always verify:
1. ✅ Module files created in `src/modules/<name>/`
2. ✅ Routes file has at least one endpoint with `export default router`
3. ✅ Controller has the corresponding handler functions
4. ✅ Routes registered in `src/routes/v1/index.js` (or `app.js` for root-level)
5. ✅ Import path is correct (relative paths from the registration file)

---

## References

For more details, read:
- `references/template-structure.md` — Complete project structure, core files, and full technology stack
- `references/env-config.md` — All `.env` variables explained
