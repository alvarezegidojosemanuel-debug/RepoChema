# PROMPT PARA CODEX — REHACER SPRITES Y SUBIRLOS A GIT

Quiero que trabajes directamente sobre este repositorio:

- **Repositorio:** `alvarezegidojosemanuel-debug/RepoChema`
- **Rama objetivo:** `claude/file-upload-download-gmtihu`
- **Ruta base de assets:** `Packs/Puñetos/`

## Objetivo

Quiero rehacer y dejar correctamente organizados en Git los sprites del juego **Puñetos**, empezando por el **Mundo 1 — Síntoma Operativo**.

No quiero hojas de sprites gigantes ni PNG con fondos falsos.

Quiero assets listos para runtime, organizados por enemigo, con:

- PNG individuales;
- transparencia alfa real;
- sin texto incrustado;
- sin checkerboard pintado;
- sin nombres rasterizados dentro del sprite;
- sin trozos de sprites vecinos;
- estados y animaciones claramente separados;
- documentación Markdown;
- validación automática antes de hacer commit.

---

# REGLA GENERAL DE TRABAJO

Antes de modificar nada:

1. Inspecciona el repositorio.
2. Localiza los assets actuales de Puñetos.
3. Localiza qué sprites ya están integrados en el juego.
4. NO borres archivos usados actualmente hasta comprobar referencias en código.
5. NO cambies nombres de rutas usadas por el motor sin actualizar todas las referencias.
6. Trabaja de forma incremental.
7. Haz commit solo cuando el pack pase validación.

Si existe código que cargue assets por rutas concretas, respétalo.

---

# ESTRUCTURA PROPUESTA

Dentro de:

`Packs/Puñetos/`

quiero una estructura similar a:

```text
Packs/
└── Puñetos/
    ├── world_01_sintoma_operativo/
    │   ├── enemies/
    │   │   ├── telefono_rojo/
    │   │   ├── cs_reclamador/
    │   │   ├── ticket_urgente/
    │   │   ├── alarma/
    │   │   ├── sensor_caido/
    │   │   ├── timeout/
    │   │   ├── log_ruidoso/
    │   │   ├── falso_positivo/
    │   │   ├── proceso_duplicado/
    │   │   ├── dependencia_rota/
    │   │   ├── proceso_zombi/
    │   │   ├── memory_leak/
    │   │   ├── paquete_corrupto/
    │   │   └── core_dump/
    │   ├── hazards/
    │   ├── fx/
    │   ├── pickups/
    │   ├── boss/
    │   │   └── incident_manager/
    │   ├── README.md
    │   └── manifest.json
    └── README.md
```

Si el repositorio ya tiene una estructura mejor, puedes adaptarla, pero debes documentar qué decides y por qué.

---

# ENEMIGOS DEL MUNDO 1

El Mundo 1 es:

**Síntoma Operativo**

Fases:

1. Entrada de incidencia.
2. Monitorización / PRTG.
3. Triage.
4. Contención.
5. Boss.

Enemigos:

## Fase 1
- Teléfono Rojo
- CS Reclamador
- Ticket Urgente

## Fase 2
- Alarma
- Sensor Caído
- Timeout

## Fase 3
- Log Ruidoso
- Falso Positivo
- Proceso Duplicado
- Dependencia Rota

## Fase 4
- Proceso Zombi
- Memory Leak
- Paquete Corrupto
- Core Dump

## Boss
- Incident Manager

---

# FORMATO DE CADA ENEMIGO

Cada enemigo debe tener una carpeta propia.

Ejemplo:

```text
memory_leak/
  base.png
  damaged.png
  hit.png
  muerte/
    muerte_00.png
    muerte_01.png
    muerte_02.png
    muerte_03.png
    muerte_04.png
  MONTAJE.md
  manifest.json
```

No todos los enemigos necesitan exactamente los mismos estados.

Si un enemigo tiene una animación específica, añadir su carpeta.

Ejemplos:

```text
telefono_rojo/
  base.png
  ring/
    ring_00.png
    ring_01.png
    ring_02.png
  damaged.png
  hit.png
  muerte/
  onda_sonora.png
  MONTAJE.md
  manifest.json
```

```text
alarma/
  verde.png
  amarillo.png
  naranja.png
  rojo.png
  hit.png
  muerte/
  MONTAJE.md
  manifest.json
```

```text
timeout/
  idle/
  countdown/
  explosion/
  hit.png
  MONTAJE.md
  manifest.json
```

---

# TRANSPARENCIA

Este punto es CRÍTICO.

Cada PNG debe:

- usar RGBA;
- tener canal alfa real;
- tener fondo completamente transparente;
- no incluir blanco;
- no incluir negro;
- no incluir damero;
- no incluir la hoja de referencia.

Comprobar automáticamente:

```python
from PIL import Image

im = Image.open(path).convert("RGBA")
alpha = im.getchannel("A")

assert alpha.getextrema()[0] == 0
assert alpha.getbbox() is not None
```

Excepción:

un frame de muerte final puede ser completamente transparente si está documentado.

---

# RECORTE

Para assets de runtime:

- recortar al contenido;
- dejar un margen pequeño;
- no cortar ninguna parte del sprite;
- no incluir elementos vecinos.

Para una animación:

- usar el mismo canvas si el motor necesita estabilidad de pivote;
- o guardar pivot/anchor en manifest.

No permitir que el personaje "baile" porque cada frame haya quedado recortado diferente.

---

# PIVOTES

Cada animación debe tener pivote coherente.

Ejemplos:

- enemigos flotantes: centro;
- terrestres: centro inferior;
- proyectiles: centro;
- explosiones: centro;
- Memory Leak: centro inferior o centro de masa.

Guardar los pivotes en `manifest.json`.

Ejemplo:

```json
{
  "pivot": [0.5, 0.8]
}
```

---

# MEMORY LEAK — REFERENCIA DE ESTRUCTURA

Este enemigo debe quedar así:

```text
memory_leak/
  base.png
  damaged.png
  hit.png
  muerte/
    muerte_00.png
    muerte_01.png
    muerte_02.png
    muerte_03.png
    muerte_04.png
```

Mecánica:

- crece mientras está vivo;
- reduce el espacio de vuelo;
- NO persigue;
- el crecimiento se hace por código;
- `damaged.png` se usa con poca vida;
- `hit.png` es flash blanco;
- muerte = desinflado progresivo;
- `muerte_04.png` puede quedar totalmente vacío.

---

# REGLAS DE GAMEPLAY A DOCUMENTAR

Cada `MONTAJE.md` debe incluir:

- rol del enemigo;
- fase;
- movimiento;
- telegraph;
- ataque;
- daño;
- muerte;
- collider;
- pivote;
- FPS recomendados;
- lógica de spawn;
- comportamiento en móvil;
- qué se anima por sprites;
- qué se anima por código.

---

# PROYECTILES

Si un enemigo dispara hacia el jugador:

- calcular dirección en el momento del disparo;
- después el proyectil viaja recto;
- NO homing;
- NO perseguir al jugador.

Solo usar homing si un enemigo concreto lo requiere explícitamente.

---

# VALIDACIÓN AUTOMÁTICA

Quiero un script dentro del repo, por ejemplo:

```text
tools/validate_punetos_assets.py
```

Debe recorrer los assets y comprobar:

## PNG

- se pueden abrir;
- RGBA;
- canal alfa;
- dimensiones válidas;
- no están vacíos salvo excepciones;
- no tienen fondo opaco completo;
- no superan tamaño razonable.

## Animaciones

- frames consecutivos;
- no faltan números intermedios;
- mismos tamaños de canvas cuando sea necesario;
- pivotes definidos.

## Documentación

Cada enemigo debe tener:

- `MONTAJE.md`;
- `manifest.json`.

## Estructura

Comprobar que están presentes todos los enemigos previstos.

Al final debe imprimir algo parecido a:

```text
WORLD 1 ASSET VALIDATION

Enemies expected: 14
Enemies found: 14

PNG checked: 187
Broken PNG: 0
Missing alpha: 0
Empty unexpected: 0
Missing manifests: 0
Missing docs: 0

STATUS: PASS
```

Si hay un problema:

`STATUS: FAIL`

y el script debe salir con código distinto de 0.

---

# VALIDACIÓN COMO SI FUERAS EL RECEPTOR

Esta regla es obligatoria.

Cuando termines un pack:

NO te limites a asumir que está bien porque lo acabas de crear.

Haz como si fueras otro desarrollador que acaba de clonar el repositorio.

Pregúntate:

> "¿Con lo que hay aquí podría montar este enemigo sin preguntarle nada al artista?"

Revisa:

- ¿están todos los estados?
- ¿falta un proyectil?
- ¿falta el efecto de impacto?
- ¿falta el estado de muerte?
- ¿falta el collider/pivot?
- ¿se entiende la mecánica?
- ¿hay sprites vacíos?
- ¿hay fondos falsos?
- ¿hay texto dentro de imágenes?
- ¿hay assets sin usar?
- ¿el juego puede localizar los archivos?

Si falta algo imprescindible:

**complétalo antes del commit.**

---

# MANIFEST GLOBAL

Crear:

```text
world_01_sintoma_operativo/manifest.json
```

Debe listar los enemigos y rutas.

Ejemplo:

```json
{
  "world": "sintoma_operativo",
  "enemies": {
    "telefono_rojo": "enemies/telefono_rojo",
    "cs_reclamador": "enemies/cs_reclamador",
    "ticket_urgente": "enemies/ticket_urgente",
    "alarma": "enemies/alarma",
    "sensor_caido": "enemies/sensor_caido",
    "timeout": "enemies/timeout",
    "log_ruidoso": "enemies/log_ruidoso",
    "falso_positivo": "enemies/falso_positivo",
    "proceso_duplicado": "enemies/proceso_duplicado",
    "dependencia_rota": "enemies/dependencia_rota",
    "proceso_zombi": "enemies/proceso_zombi",
    "memory_leak": "enemies/memory_leak",
    "paquete_corrupto": "enemies/paquete_corrupto",
    "core_dump": "enemies/core_dump"
  },
  "boss": "boss/incident_manager"
}
```

---

# GIT

Cuando todo pase validación:

1. `git status`
2. revisar que no haya basura;
3. no incluir archivos temporales;
4. no incluir ZIP si ya están todos los PNG descomprimidos salvo que sea necesario;
5. `git add` solo de assets/documentación/scripts previstos;
6. commit.

Commit sugerido:

```text
assets: rebuild world 1 operational symptom sprites
```

Después:

```bash
git push origin claude/file-upload-download-gmtihu
```

---

# IMPORTANTE

No quiero una respuesta diciendo solamente qué harías.

Quiero que:

1. inspecciones el repo;
2. rehagas/organices los assets;
3. crees validación;
4. ejecutes la validación;
5. arregles lo que falle;
6. hagas commit;
7. hagas push;
8. me devuelvas:
   - commit SHA;
   - lista de enemigos;
   - resultado de validación;
   - rutas principales;
   - cualquier incidencia que quede pendiente.

Si no tienes permisos de push:

- prepara igualmente todos los cambios;
- deja el commit creado localmente;
- dime exactamente qué permiso falta;
- NO descartes el trabajo.

---

# REGLA FINAL

**No dar por terminado un sprite porque exista un PNG.**

Un sprite está terminado cuando:

- visualmente es correcto;
- tiene transparencia;
- tiene los estados necesarios;
- está documentado;
- el motor puede montarlo;
- pasa validación;
- está versionado en Git.
