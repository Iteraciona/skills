# Template Structure & Technology Stack

## Project Tree

```
.
├── .env                          # Environment variables (auto-created)
├── .env.example                  # Template for environment vars
├── .gitignore                    # Git ignore rules
├── CHANGELOG.md                  # Version history
├── LICENSE                       # MIT License
├── README.md                     # Project documentation
├── package.json                  # Dependencies & scripts
├── server.js                     # Entry: banner, env validation, DB init, HTTP server
├── app.js                        # Express config: middlewares, CORS, routes
├── db.js                         # Mongoose connection builder
├── initLogger.js                 # Winston + Luxon logger setup
└── src/
    ├── helpers/                  # Global utilities
    │   ├── logger.js             # Winston logger instance
    │   └── token.js              # JWT token helpers
    ├── middlewares/               # Express middlewares
    │   ├── authMiddleware.js     # JWT authentication guard
    │   └── rateLimiter.js        # Rate limiting config
    ├── modules/                   # Business logic (modular)
    │   └── home/                 # Default "home" module
    │       ├── controllers/
    │       │   └── homeController.js
    │       ├── models/
    │       ├── routes/
    │       │   └── homeRoutes.js
    │       └── services/
    │           └── homeService.js
    └── routes/
        └── v1/
            └── index.js          # Versioned route aggregator
```

## Core Files

| File | Purpose |
|---|---|
| `server.js` | Entry point. Prints the professional startup banner with system stats. Validates `ENVIRONMENT` and `DEFAULT_PORT`. Handles MongoDB connection logic (non-blocking in DEV, strict in PROD). Creates HTTP server. |
| `app.js` | Express application setup. Mounts CORS, Morgan logging, rate limiter, JSON parsing, and versioned routes (`/api/v1/`). |
| `db.js` | Builds the MongoDB connection URI from `.env` vars and connects via Mongoose. |
| `initLogger.js` | Initializes Winston with daily rotate file transport and Luxon-based timestamps (`America/Lima` timezone). |

## Modular Architecture

Each business module lives inside `src/modules/<module-name>/` with 4 layers:

```
src/modules/<module-name>/
├── controllers/    # Request handling, input validation, response formatting
├── models/         # Mongoose schemas and model definitions
├── routes/         # Express Router with endpoint definitions
├── services/       # Business logic, database queries, external API calls
└── validators/     # express-validator rules and error-handling middleware
```

When adding a new module:
1. Create the module folder and its sub-folders
2. Define the Mongoose model in `models/`
3. Write business logic in `services/`
4. Create the controller in `controllers/`
5. Define routes in `routes/`
6. Register the module routes in `src/routes/v1/index.js`

## Technology Stack

### Core Dependencies
| Package | Version | Purpose |
|---|---|---|
| `express` | ^5.2.1 | Web framework |
| `mongoose` | ^9.2.4 | MongoDB ODM |
| `jsonwebtoken` | ^9.0.3 | JWT authentication |
| `bcrypt` | ^6.0.0 | Password hashing |
| `dotenv` | ^17.3.1 | Environment variable loading |
| `cors` | ^2.8.6 | Cross-origin resource sharing |
| `express-rate-limit` | ^8.2.1 | API rate limiting |
| `express-validator` | ^7.3.1 | Request validation |
| `winston` | ^3.19.0 | Logging framework |
| `winston-daily-rotate-file` | ^5.0.0 | Daily log rotation |
| `luxon` | ^3.7.2 | Date/time handling |

### Utilities
| Package | Version | Purpose |
|---|---|---|
| `lodash` | ^4.17.23 | JavaScript utilities |
| `socket.io` | ^4.8.3 | Real-time communication |
| `resend` | ^6.9.3 | Email sending |
| `uuid` | ^13.0.0 | UUID generation |
| `multer` | ^2.1.1 | File upload handling |
| `sanitize-html` | ^2.17.1 | HTML sanitization |
| `morgan` | ^1.10.1 | HTTP request logger |
| `geolib` | ^3.3.4 | Geolocation utilities |
| `ejs` | ^4.0.1 | Template engine |
| `request-ip` | ^3.3.0 | Client IP detection |
| `bson` | ^7.2.0 | BSON serialization |

### Development
| Package | Version | Purpose |
|---|---|---|
| `nodemon` | ^3.1.14 | Auto-restart on file changes (devDependency) |

## NPM Scripts

```json
{
  "start": "node server.js",       // Production
  "dev": "nodemon server.js"       // Development (hot reload)
}
```
