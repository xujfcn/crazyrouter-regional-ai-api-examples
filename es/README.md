# Cómo usar GPT, Claude y Gemini con una sola API key

Si estás construyendo productos de IA desde México —un chatbot para soporte, un copiloto interno, una herramienta de análisis o automatizaciones para tu equipo— probablemente ya te encontraste con este problema: cada proveedor tiene su propia API key, SDK, formato de modelos y lógica de errores.

Una forma más simple es usar un gateway compatible con OpenAI: configuras una sola API key y cambias de modelo con el campo `model`.

Esta guía muestra una configuración práctica con Crazyrouter para llamar GPT, Claude y Gemini desde un solo endpoint, sin reescribir tu aplicación cada vez que pruebas un proveedor nuevo.

## Cuándo conviene este enfoque

Usar una sola API key tiene sentido si quieres:

- Probar varios modelos en el mismo flujo de producto.
- Mantener un solo cliente HTTP o SDK compatible con OpenAI.
- Cambiar entre modelos por costo, latencia, disponibilidad o calidad.
- Evitar manejar credenciales de varios proveedores dentro de tu app.

No necesitas migrar todo de golpe. Puedes empezar con una ruta de prueba y mover más tráfico cuando el setup esté estable.

## 1. Crea tu API key

Primero revisa la introducción de la documentación y genera tu key desde el panel de Crazyrouter:

- Documentación: [Crazyrouter docs](https://docs.crazyrouter.com/en/introduction?utm_source=mexico&utm_medium=regional_auto&utm_campaign=docs_intro)
- Inicio rápido: [Quickstart](https://docs.crazyrouter.com/en/quickstart?utm_source=mexico&utm_medium=regional_auto&utm_campaign=quickstart)

Guarda la key como variable de entorno:

```bash
export CRAZYROUTER_API_KEY="cr_..."
```

En producción, usa el secret manager de tu plataforma. No subas esta key a GitHub ni la pongas en código frontend.

## 2. Usa el endpoint compatible con OpenAI

Crazyrouter puede usarse con clientes que aceptan `baseURL`. En JavaScript:

```js
import OpenAI from "openai";

const client = new OpenAI({
  apiKey: process.env.CRAZYROUTER_API_KEY,
  baseURL: "https://crazyrouter.com/v1",
});

const response = await client.chat.completions.create({
  model: "gpt-4o-mini",
  messages: [
    { role: "system", content: "Responde de forma clara y breve." },
    { role: "user", content: "Resume las ventajas de usar un gateway de IA." },
  ],
});

console.log(response.choices[0].message.content);
```

El punto importante: tu aplicación sigue usando una interfaz familiar, pero el modelo puede cambiar por proveedor.

## 3. Cambia entre GPT, Claude y Gemini

Puedes mantener el mismo código y cambiar solo el valor de `model`:

```js
const models = [
  "gpt-4o-mini",
  "claude-haiku-4-5",
  "gemini-2.5-flash",
];

for (const model of models) {
  const result = await client.chat.completions.create({
    model,
    messages: [{ role: "user", content: "Dame una idea para mejorar una landing page SaaS." }],
  });

  console.log("\nMODEL:", model);
  console.log(result.choices[0].message.content);
}
```

Este patrón es útil para pruebas A/B internas: mismo prompt, misma estructura de respuesta esperada, distintos modelos.

## 4. Ejemplo en Python

Si tu stack usa Python, la idea es igual:

```python
from openai import OpenAI
import os

client = OpenAI(
    api_key=os.environ["CRAZYROUTER_API_KEY"],
    base_url="https://crazyrouter.com/v1",
)

completion = client.chat.completions.create(
    model="claude-haiku-4-5",
    messages=[
        {"role": "system", "content": "Eres un asistente técnico para developers."},
        {"role": "user", "content": "Explica qué es un endpoint compatible con OpenAI."},
    ],
)

print(completion.choices[0].message.content)
```

## 5. Buenas prácticas para producción

Antes de mandar tráfico real, agrega tres cosas:

1. **Timeouts**: no dejes requests esperando indefinidamente.
2. **Retries con backoff**: reintenta errores temporales, pero evita loops agresivos.
3. **Logging por modelo**: guarda modelo, latencia y resultado general para comparar calidad y costo.

Ejemplo simple de timeout en Node.js:

```js
const controller = new AbortController();
const timeout = setTimeout(() => controller.abort(), 30_000);

try {
  const response = await client.chat.completions.create({
    model: "gemini-2.5-flash",
    messages: [{ role: "user", content: "Genera 5 títulos para un blog fintech." }],
  }, { signal: controller.signal });

  console.log(response.choices[0].message.content);
} finally {
  clearTimeout(timeout);
}
```

## Siguiente paso

Empieza con un endpoint pequeño —por ejemplo, generación de títulos, clasificación de tickets o resumen de texto— y prueba dos o tres modelos con el mismo prompt. Cuando tengas métricas de latencia, costo y calidad, decide cuál modelo usar por defecto y cuál dejar como alternativa.

Para revisar el flujo completo de configuración, vuelve al [quickstart de Crazyrouter](https://docs.crazyrouter.com/en/quickstart?utm_source=mexico&utm_medium=regional_auto&utm_campaign=next_step).
