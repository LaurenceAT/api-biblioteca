# API Biblioteca Campus del Duero

API REST del Laboratorio 09 (Desarrollo de Aplicativos Móviles). Centraliza el catálogo de la
biblioteca, aplica las reglas de préstamo y responde siempre en JSON. La consume la app Android
**BiblioCampus** (Kotlin + Jetpack Compose + Retrofit, patrón MVVM).

**URL pública:** https://api-biblioteca-gd7i.onrender.com
(catálogo: https://api-biblioteca-gd7i.onrender.com/api/v1/libros)

## Ejecutar en local

```bash
npm install
npm start        # http://localhost:3000  (emulador Android: http://10.0.2.2:3000)
npm run dev      # reinicia el servidor al guardar
npm test         # 13 pruebas automáticas
```

Requiere Node.js 20 o superior.

## Endpoints

| Requisito | Método | Ruta | Respuestas |
|---|---|---|---|
| — | GET | `/` | 200 (estado de la API) |
| RF01-RF03 | GET | `/api/v1/libros?q=texto&categoria=Informática` | 200 |
| RF04 | GET | `/api/v1/libros/categorias` | 200 |
| RF05 | GET | `/api/v1/libros/{id}` | 200, 400, 404 |
| RF06 | POST | `/api/v1/libros` | 201, 400, 409 |
| RF07 | PUT | `/api/v1/libros/{id}` | 200, 400, 404, 409 |
| RF08 | PATCH | `/api/v1/libros/{id}/prestamo` | 200, 404, 409 |
| RF09 | PATCH | `/api/v1/libros/{id}/devolucion` | 200, 404, 409 |
| RF10 | DELETE | `/api/v1/libros/{id}` | 200, 404 |

## Formato de respuesta

Todas las respuestas (también los errores y las rutas inexistentes) usan el mismo "sobre":

```json
{
  "success": true,
  "message": "Libro encontrado",
  "data": { "id": 2, "titulo": "Clean Code", "autor": "Robert C. Martin", "isbn": "978-0-13-235088-4",
            "anio": 2008, "categoria": "Informática", "ejemplares": 4, "disponibles": 4,
            "ubicacion": "Planta 2 - Estantería B4" },
  "timestamp": "2026-10-09T16:12:18.816Z"
}
```

Los errores de validación (400) incluyen `errors: [{ "campo": "anio", "detalle": "..." }]`.

## Reglas de negocio

- **RN01** Un libro nuevo empieza con todos sus ejemplares disponibles.
- **RN02** No se presta un libro sin ejemplares disponibles → 409.
- **RN03** No se devuelve un ejemplar si todos están en la biblioteca → 409.
- **RN04** El ISBN no se repite → 409.
- **RN05** Título (≥ 2), autor (≥ 3), ISBN (10-17 dígitos o guiones), año (1000-actual),
  categoría, ejemplares (1-100) y ubicación son obligatorios → 400 con `errors`.
- **RN06** Al cambiar el número de ejemplares se conservan los préstamos activos.

## Estructura

```
src/
├── server.js                      arranca el servidor (lee PORT)
├── app.js                         configura Express, 404 y errores
├── routes/libros.routes.js        URL + método HTTP
├── middlewares/validarLibro.js    valida el JSON recibido (RN05)
├── controllers/libros.controller.js  reglas de negocio y códigos HTTP
├── data/libros.data.js            catálogo en memoria
└── utils/respuesta.js             formato JSON estándar
tests/api.test.js                  pruebas automáticas (node --test)
```

## Despliegue en Render

New → Web Service → este repositorio. Language: Node · Build: `npm install` ·
Start: `npm start` · Instance: Free. El plan gratuito se suspende tras 15 minutos sin tráfico
y los datos (en memoria) se reinician al reiniciarse el servicio.
