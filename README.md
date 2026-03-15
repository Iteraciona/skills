# Iteraciona Skills

Colección de skills reutilizables desarrollados por **Iteraciona** para estandarizar y acelerar la creación de proyectos. Cada skill es un conjunto de instrucciones, convenciones y plantillas que un agente de IA puede seguir para generar código consistente y de alta calidad.

La lista crece cada día conforme identificamos patrones que vale la pena automatizar.

## ✨ Skills Disponibles

| Skill | Descripción |
|---|---|
| **[api-creator](./skills/api-creator/)** | Scaffold de APIs con Node.js + Express, MongoDB, JWT, Winston y arquitectura modular |
| **[svelte-creator](./skills/svelte-creator/)** | Scaffold de frontends con SvelteKit 5 + TailwindCSS 4, TypeScript, api-proxy y adapter-node |
| **[readme-creator](./skills/readme-creator/)** | Generación de README.md profesionales a partir de descripción del usuario o análisis del proyecto |

## 🚀 ¿Cómo se usan?

Los skills están diseñados para ser utilizados por un **agente de IA** (como Gemini o Claude) dentro de un entorno de desarrollo. Para activar un skill en un workspace:

1. Copia o vincula la carpeta del skill en `.agents/skills/` dentro de tu proyecto:

   ```bash
   # Desde la raíz de tu proyecto
   mkdir -p .agents/skills
   ln -s /ruta/a/iteraciona/skills/<nombre-skill> .agents/skills/<nombre-skill>
   ```

2. El agente detectará automáticamente el skill y lo usará cuando el contexto lo requiera.

## 📁 Estructura

```
.
├── README.md                 # Este archivo
└── skills/
    ├── api-creator/          # Skill para APIs Node.js + Express
    │   ├── SKILL.md          # Instrucciones del skill
    │   └── references/       # Documentación complementaria
    ├── svelte-creator/       # Skill para frontends SvelteKit
    │   ├── SKILL.md
    │   └── references/
    └── readme-creator/       # Skill para generar READMEs
        └── SKILL.md
```

## 🤝 Contribuir

¿Tienes un patrón que usas en cada proyecto? Conviértelo en un skill:

1. Crea una carpeta con el nombre del skill (ej. `docker-creator/`)
2. Agrega un archivo `SKILL.md` con frontmatter YAML (`name`, `description`) y las instrucciones detalladas
3. Opcionalmente, incluye una carpeta `references/` con documentación de apoyo

---

*Hecho con 🧠 por Iteraciona*
