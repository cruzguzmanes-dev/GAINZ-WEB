# Gainz — Project Context for Claude Code

## ¿Qué es Gainz?
App de vocabulario en inglés para iPhone y Apple Watch.
El usuario descarga paquetes de palabras por categoría.
Cada palabra tiene contenido bilingüe (EN + ES).

---

## Stack

| Capa | Tecnología |
|------|-----------|
| Base de datos | Supabase (PostgreSQL) |
| API | Supabase REST (PostgREST) — sin backend custom |
| Web Admin | HTML + Vanilla JS, desplegado en Vercel |
| iOS App | Swift, Xcode 26 |
| Watch App | watchOS, WatchConnectivity desde iPhone |

---

## Repositorio
- GitHub: `GAINZ-WEB` (usuario: cruzguzmanes)
- Estructura: `/Web/gainz-admin.html`
- Deploy: Vercel → Root Directory = `Web`

---

## Base de datos — Supabase

### Tabla: `categories`
```sql
id          uuid PRIMARY KEY DEFAULT gen_random_uuid()
name        text NOT NULL
emoji       text
created_at  timestamptz DEFAULT now()
```

### Tabla: `words`
```sql
id            uuid PRIMARY KEY DEFAULT gen_random_uuid()
category_id   uuid REFERENCES categories(id) ON DELETE CASCADE
word_en       text NOT NULL
definition_en text
examples_en   text[]
meaning_es    text
uses_es       text[]
created_at    timestamptz DEFAULT now()
updated_at    timestamptz DEFAULT now()
```

### Row Level Security
- Lectura pública habilitada (para que el iPhone consuma sin auth)
- Escritura pública habilitada (por ahora, solo admin usa la web)

---

## API — Endpoints que usa el iPhone

### Obtener todas las categorías
```
GET /rest/v1/categories?select=*&order=created_at.asc
Headers:
  apikey: <anon_key>
  Authorization: Bearer <anon_key>
```

### Obtener palabras de una categoría
```
GET /rest/v1/words?category_id=eq.{id}&select=*
Headers:
  apikey: <anon_key>
  Authorization: Bearer <anon_key>
```

### Estructura del JSON que recibe el iPhone
```json
[
  {
    "id": "uuid",
    "category_id": "uuid",
    "word_en": "leverage",
    "definition_en": "The use of borrowed capital to increase potential return...",
    "examples_en": ["She used her network to leverage the deal.", "..."],
    "meaning_es": "Apalancamiento / aprovechar",
    "uses_es": ["Hay que aprovechar cada oportunidad.", "..."],
    "created_at": "2026-03-10T..."
  }
]
```

---

## Web Admin (`gainz-admin.html`)
- Panel CMS para crear/editar/eliminar categorías y palabras
- Conecta a Supabase via fetch con anon key (guardada en localStorage)
- Campos por palabra: `word_en`, `definition_en`, `examples_en[]`, `meaning_es`, `uses_es[]`
- Desplegado en Vercel, root directory = `Web`

---

## iOS App — Estado actual
- Xcode 26 (release)
- Se usa Claude Code desde terminal para desarrollo
- Arquitectura: descarga por categoría bajo demanda (no bulk)
- Guardado local: SwiftData (offline support)
- Apple Watch recibe la palabra activa via WatchConnectivity

---

## Flujo de datos
```
Supabase DB
    ↓  REST GET /words?category_id=eq.{id}
iPhone (Swift) — SwiftData local cache
    ↓  WatchConnectivity
Apple Watch — muestra palabra activa
```

---

## Convenciones de código (iOS)
- Swift moderno, async/await para networking
- SwiftData para persistencia local
- Sin backend custom — iPhone llama directo a Supabase REST

---

## Próximos pasos
- [ ] Conectar iOS app a Supabase REST
- [ ] Modelo SwiftData para Category y Word
- [ ] Vista de categorías en iPhone
- [ ] Vista de palabra activa en Apple Watch
- [ ] (Futuro) Supabase Auth para proteger el web admin
