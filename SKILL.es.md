---
name: Code Soothsayer
description: Sistema de predicción de fallos de código impulsado por IA que analiza bases de código para identificar posibles errores, fallos en tiempo de ejecución y vulnerabilidades antes de que se manifiesten en producción
version: 2.4.1
author: Kilo Engineering Team
tags:
  - prediction
  - failures
  - code
  - bugs
  - static-analysis
  - ml
dependencies:
  - python>=3.9
  - tree-sitter>=0.20.0
  - torch>=2.0.0
  - transformers>=4.30.0
  - astroid>=2.15.0
  - bandit>=1.7.0
  - semgrep>=1.50.0
  - pre-commit>=3.0.0
required_services: []
conflicts:
  - old-soothsayer
  - legacy-analyzer
---

# Propósito

Code Soothsayer predice fallos de código antes de que se manifiesten combinando análisis estático, modelos de aprendizaje automático entrenados en millones de commits de corrección de errores, y reconocimiento de patrones en tiempo de ejecución. Identifica problemas sutiles que los linters tradicionales pasan por alto: condiciones de carrera, fugas de recursos, uso incorrecto de APIs, vulnerabilidades de seguridad y defectos arquitectónicos.

**Casos de uso reales:**
- Prevenir interrupciones en producción detectando condiciones de carrera en código Go asíncrono (`soothsayer analyze --lang=go ./services/payment --pattern=race`)
- Identificar fugas de memoria en C++ antes de que consuman toda la RAM (`soothsayer predict --lang=cpp --memory-profile=high ./src/core`)
- Descubrir vulnerabilidades de inyección SQL en código Python 2 legado (`soothsayer check --lang=py --version=2.7 ./legacy/app.py`)
- Predecir qué actualizaciones de dependencias romperán tu build (`soothsayer simulate --upgrade=true ./package.json`)
- Encontrar rutas de código inalcanzables en componentes React complejos (`soothsayer analyze --framework=react --dead-code ./components`)

---

# Alcance

## Comandos

### `analyze`
Análisis estático profundo con coincidencia de patrones e inferencia ML.

**Flags:**
- `--lang=<language>` (required): `python`, `javascript`, `go`, `rust`, `java`, `cpp`, `ruby`, `swift`, `kotlin`, `typescript`
- `--pattern=<type>`: `race`, `memory`, `null`, `sql`, `xss`, `dead-code`, `all` (default)
- `--output=<format>`: `json`, `sarif`, `terminal`, `html` (default: `terminal`)
- `--severity-threshold=<int>`: 1-5 (default: 3)
- `--include-paths=<glob>`: e.g., `src/**,services/**`
- `--exclude-paths=<glob>`: e.g., `node_modules/**,test/**`
- `--context-lines=<int>`: Líneas de contexto alrededor del problema (default: 3)
- `--confidence-threshold=<float>`: 0.0-1.0 (default: 0.7)
- `--no-cache`: Omitir resultados en caché
- `--fix-suggestions`: Incluir parches de corrección automática (experimental)

**Ejemplo:**
```bash
soothsayer analyze --lang=python --pattern=memory --output=sarif --severity-threshold=4 ./app/ > report.sarif
```

### `predict`
Predecir fallos futuros basándose en patrones de evolución de código y comportamiento de desarrolladores.

**Flags:**
- `--horizon=<time>`: `1w`, `1m`, `3m`, `6m` (default: `3m`)
- `--risk-threshold=<float>`: 0.0-1.0 (default: 0.5)
- `--hotspot-analysis`: Enfocarse en archivos de alta actividad
- `--developer-risks`: Considerar niveles de experiencia de desarrolladores (requiere `--git-repo`)
- `--git-repo=<path>`: Ruta al repositorio git (default: actual)
- `--since=<commit>`: Analizar commits desde (default: últimos 100)
- `--output=<format>`: `json`, `table`, `graph` (default: `table`)

**Ejemplo:**
```bash
soothsayer predict --horizon=1m --hotspot-analysis --git-repo=./myproject --output=graph > risk-graph.html
```

### `check`
Validación rápida de archivos o directorios específicos.

**Flags:**
- `--strict`: Fallar en cualquier advertencia
- `--fail-under=<float>`: Puntuación mínima (0-100) para aprobar (default: 80)
- `--ci`: Salida optimizada para CI/CD (parseable por máquina)
- `--baseline=<file>`: Comparar contra reporte de línea base
- `--diff`: Solo verificar archivos modificados (requiere git)
- `--format=<format>`: `compact`, `detailed`, `summary` (default: `detailed`)

**Ejemplo:**
```bash
soothsayer check --strict --ci --diff ./src/ | tee soothsayer.log
exit ${PIPESTATUS[0]}
```

### `simulate`
Simular el impacto de cambios (actualizaciones de dependencias, refactors, cambios de configuración).

**Flags:**
- `--upgrade=<spec>`: e.g., `django:4.2`, `'requests:*'`
- `--refactor=<pattern>`: `extract-method`, `rename-variable`, `split-class`
- `--config-change=<file>`: Ruta a configuración modificada
- `--dry-run`: Mostrar predicciones sin aplicar cambios
- `--confidence`: Mostrar intervalos de confianza
- `--compare-to=<commit>`: Comparar predicciones antes/después

**Ejemplo:**
```bash
soothsayer simulate --upgrade='django:4.2,react:18.0' --dry-run --confidence ./ > upgrade-predictions.txt
```

### `learn`
Reentrenar o hacer fine-tuning de modelos con datos de tu propia base de código.

**Flags:**
- `--dataset=<path>`: Ruta a dataset etiquetado (CSV/JSON)
- `--model-type=<type>`: `bug-classifier`, `race-detector`, `memory-analyzer` (default: `bug-classifier`)
- `--epochs=<int>`: Épocas de entrenamiento (default: 10)
- `--validate-split=<float>`: Ratio de división de validación (default: 0.2)
- `--export=<path>`: Exportar modelo entrenado
- `--import-model=<path>`: Importar modelo existente para fine-tuning
- `--eval-only`: Solo evaluar, no entrenar

**Ejemplo:**
```bash
soothsayer learn --dataset=./historical-bugs.csv --model-type=bug-classifier --epochs=20 --export=./custom-model.pt
```

### `report`
Generar reportes comprehensivos de análisis previos.

**Flags:**
- `--input=<file>`: Reporte JSON/SARIF de entrada (default: `soothsayer-report.json`)
- `--format=<format>`: `pdf`, `html`, `markdown`, `json` (default: `html`)
- `--template=<name>`: `executive`, `developer`, `audit` (default: `developer`)
- `--since=<date>`: Filtrar por fecha (ISO 8601)
- `--group-by=<field>`: `file`, `severity`, `pattern`, `developer` (default: `file`)
- `--include-fixes`: Incluir soluciones sugeridas en el reporte

**Ejemplo:**
```bash
soothsayer report --input=last-scan.json --format=pdf --template=executive --since=2024-01-01 > q1-report.pdf
```

### `validate-model`
Verificar precisión del modelo contra casos de prueba conocidos.

**Flags:**
- `--test-suite=<path>`: Ruta a directorio de suite de pruebas
- `--benchmark`: Ejecutar suite de benchmarks (toma ~30min)
- `--precision-threshold=<float>`: Precisión mínima (default: 0.85)
- `--recall-threshold=<float>`: Recall mínimo (default: 0.75)
- `--output-dir=<path>`: Dónde guardar resultados (default: `./validation-results/`)

**Ejemplo:**
```bash
soothsayer validate-model --test-suite=./tests/validation/ --benchmark --precision-threshold=0.90
```

---

# Proceso de Trabajo

## Flujo de Trabajo Estándar

1. **Inicialización**
   ```bash
   soothsayer init --project-type=webapp --langs=python,js
   ```
   Crea archivo de configuración `.soothsayer.yml` y cachea modelos de lenguaje.

2. **Análisis de Línea Base**
   ```bash
   soothsayer analyze --lang=python --output=json > baseline.json
   ```
   Establece línea base inicial de predicción de fallos.

3. **Integración CI/CD**
   ```bash
   soothsayer check --strict --ci --diff
   ```
   Se ejecuta en cada PR/commit, falla si se detectan nuevas predicciones de alta severidad.

4. **Predicción Periódica**
   ```bash
   soothsayer predict --horizon=1m --hotspot-analysis > risk-forecast.json
   ```
   Programado semanalmente para identificar áreas de riesgo emergentes.

5. **Simulación Pre-Actualización**
   ```bash
   soothsayer simulate --upgrade='package.json' --dry-run > upgrade-risks.json
   ```
   Antes de cualquier actualización de dependencias.

## Flujo de Trabajo Avanzado: Puerta de Múltiples Etapas

```bash
# Etapa 1: Verificación rápida en archivos modificados
soothsayer check --diff --format=compact > quick-check.txt

# Etapa 2: Análisis profundo en hotspots (solo si Etapa 1 encuentra problemas)
if grep -q "SEVERITY.*[45]" quick-check.txt; then
  soothsayer analyze --lang=python --severity-threshold=3 ./src/ > deep-analysis.json
fi

# Etapa 3: Predecir fallos futuros en módulos modificados
soothsayer predict --hotspot-analysis --git-repo=. --since=HEAD~5 > future-risks.json

# Etapa 4: Bloquear merge si puntuación de riesgo > umbral
RISK_SCORE=$(jq '.risk_score' future-risks.json)
if (( $(echo "$RISK_SCORE > 0.7" | bc -l) )); then
  echo "Merge bloqueado: riesgo de fallo predicho demasiado alto (puntuación: $RISK_SCORE)"
  exit 1
fi
```

---

# Reglas de Oro

1. **Nunca ignores predicciones de alta confianza (≥0.9) con severidad 4-5** - estas correlacionan 94% con incidentes en producción según nuestros datos internos.

2. **Siempre usa `--diff` en CI** - analizar toda la base de código en cada commit es derrochador y produce ruido.

3. **Configura `--confidence-threshold` apropiadamente para tu madurez de código:**
   - Proyectos nuevos: 0.5 (más permisivo)
   - Proyectos maduros: 0.8 (estricto)
   - Sistemas críticos: 0.95 (muy estricto)

4. **Revisa y reentrena modelos trimestralmente** - los patrones de código evolucionan, los modelos degradan sin datos frescos.

5. **Nunca desactives el caché en producción** - análisis completo sin caché es 10-50x más lento.

6. **Siempre combina `predict` con `--hotspot-analysis`** - predecir en archivos de baja actividad produce falsos positivos.

7. **Trata los códigos de salida de `soothsayer check` como canónicos:**
   - 0: No se encontraron problemas
   - 1: Problemas encontrados pero dentro del umbral
   - 2: Umbral excedido (fallar)
   - 3: Error de configuración
   - 4: Análisis fallido (crash)

8. **Nunca commits fixes auto-generados sin revisión manual** - las sugerencias de fix están correctas 78% del tiempo para problemas simples, pero solo 42% para complejos.

9. **Siempre ejecuta `validate-model` después de entrenamiento personalizado** - no despliegues modelos que no cumplan umbrales de precisión/recall.

10. **Configura `--exclude-paths` para código generado y dependencias de terceros** - analizar código de terceros desperdicia tiempo y crea falsos positivos.

---

# Ejemplos

## Ejemplo 1: Detectando Condición de Carrera en Servicio Go

**Comando del usuario:**
```bash
soothsayer analyze --lang=go --pattern=race --confidence-threshold=0.8 --output=json ./services/order/
```

**Salida (formateada):**
```json
{
  "scan_id": "scan_20240308_001",
  "timestamp": "2024-03-08T14:23:45Z",
  "predictions": [
    {
      "file": "services/order/processor.go",
      "line": 247,
      "column": 12,
      "pattern": "race",
      "severity": 5,
      "confidence": 0.92,
      "title": "Posible condición de carrera en actualización de estado de orden",
      "description": "Acceso no sincronizado a order.status entre goroutines. El objeto order se comparte entre múltiples workers sin protección mutex.",
      "code_snippet": "func (w *OrderWorker) process(o *Order) {\n    if o.Status == PENDING {  // RACE HERE\n      o.Status = PROCESSING\n    }\n  }",
      "fix_suggestion": "Añadir sync.Mutex a struct Order y lockaround comprobaciones de estado:\n\nvar mu sync.Mutex\nmu.Lock()\nif o.Status == PENDING {\n  o.Status = PROCESSING\n}\nmu.Unlock()",
      "cwe_id": "CWE-362",
      "references": [
        "https://go.dev/doc/race",
        "https://cwe.mitre.org/data/definitions/362.html"
      ]
    }
  ],
  "statistics": {
    "total_files": 42,
    "files_analyzed": 41,
    "predictions_count": 1,
    "analysis_time_seconds": 12.3,
    "cache_hit_rate": 0.73
  }
}
```

**Acción del usuario:** El desarrollador revisa la predicción, confirma la condición de carrera, añade protección mutex, y vuelve a ejecutar análisis para verificar el fix.

---

## Ejemplo 2: Prediciendo Fallo Después de Actualización de Dependencia

**Comando del usuario:**
```bash
soothsayer simulate --upgrade='django:4.2,psycopg2:3.1' --dry-run --confidence --output=table ./backend/
```

**Salida (formateada):**
```
Simulation Results (Dry Run)
Generated: 2024-03-08 14:30:15 UTC
Project: ./backend/
Upgrades: django:4.2, psycopg2:3.1

┌──────────────────────────────────────┬─────────┬─────────────┬─────────────────────────────────────────────┐
│ Issue                                │ Severity│ Confidence │ Impact                                      │
├──────────────────────────────────────┼─────────┼─────────────┼─────────────────────────────────────────────┤
│ Django 4.2 elimina soporte para      │   4     │    0.87     │ 3 módulos fallarán en import tras upgrade   │
│   django.utils.six                   │         │             │                                             │
│ Psycopg2 3.0+ requiere PostgreSQL   │   3     │    0.91     │ Fallos de conexión si PG < 12               │
│   12+                                │         │             │                                             │
│ Django 4.2 cambia tag 'url' por     │   3     │    0.78     │ Todas las 'url' tags deben actualizarse     │
│   'uri' por defecto                  │         │             │                                             │
└──────────────────────────────────────┴─────────┴─────────────┴─────────────────────────────────────────────┘

Summary:
- 3 problemas detectados (1 crítico, 2 medio)
- Esfuerzo estimado de upgrade: 8-12 horas
- Puntuación de riesgo: 0.76 (ALTO)
Recomendación: Abordar problemas antes del upgrade. Ver detalles completos en django-upgrade-plan.md (generado con flag --report).
```

**Acción del usuario:** El desarrollador pasa un día actualizando código para compatibilidad Django 4.2, luego vuelve a ejecutar simulación para verificar que todos los problemas se resolvieron antes del upgrade real.

---

## Ejemplo 3: Bloqueo CI/CD en Cambio de Alto Riesgo

**Comando (en pipeline CI):**
```bash
soothsayer check --strict --ci --diff --output=compact
```

**Salida cuando se detecta fallo:**
```
::error::Code Soothsayer detectó problemas críticos
Scan ID: scan_ci_83274
Archivos modificados: 3
Predicción: ALTO_RIESGO

ISSUE 1: services/auth/middleware.py:89:5
  Pattern: sql
  Severity: 5
  Confidence: 0.94
  Title: Posible inyección SQL en búsqueda de usuario
  Fix: Usar consultas parametrizadas

ISSUE 2: tests/test_api.py:234:12
  Pattern: dead-code
  Severity: 3
  Confidence: 0.81
  Title: Función de prueba nunca llamada (código muerto)

Puntuación de Riesgo General: 0.83 (umbral: 0.5)
```

**Salida del sistema CI:**
```
##[error]Proceso completado con código de salida 2.
```

**Acción del usuario:** El desarrollador corrige vulnerabilidad de inyección SQL, elimina código muerto, hace push del nuevo commit, CI pasa.

---

## Ejemplo 4: Predicción de Riesgo a Largo Plazo

**Comando del usuario:**
```bash
soothsayer predict --horizon=6m --hotspot-analysis --developer-risks --output=graph ./src/
```

**Salida (gráfico guardado en `risk-forecast.html` con visualización D3.js interactiva):**

HTML interactivo con:
- Mapa de calor de archivos por probabilidad de fallo predicha (rojo = alto riesgo)
- Línea de tiempo mostrando tendencia de riesgo sobre 6 meses
- Overlay de impacto de desarrollador (archivos tocados por ingenieros junior marcados)
- Click para drilled-down a detalles de predicción específicos

**Resumen de texto también impreso:**
```
Resumen de Pronóstico de Riesgo (horizonte 6 meses)
Proyecto: ./src/
Hotspots identificados: 7 archivos

Top 3 Archivos de Alto Riesgo:
1. src/utils/cache.py (riesgo: 0.87) - Patrón de fuga de memoria, lógica de caché compleja
2. src/api/auth.py (riesgo: 0.82) - posible omisión de autenticación, tocado recientemente por 3 devs
3. src/workers/order.py (riesgo: 0.79) - Condiciones de carrera en worker asíncrono

Recomendación: Priorizar refactor de cache.py antes del release Q3.
```

**Acción del usuario:** El tech lead programa sprint para refactorizar archivos de alto riesgo, asigna ingeniero senior a cache.py debido a la complejidad.

---

# Comandos de Rollback

## Rollback de Estado de Análisis

**Limpiar caché de análisis:**
```bash
soothsayer cache --clear --all
```

**Restaurar desde backup (si existe `.soothsayer-backup/`):**
```bash
soothsayer restore --backup-dir=.soothsayer-backup --date=2024-03-07
```

## Rollback de Actualizaciones de Modelo

**Revertir a versión de modelo anterior:**
```bash
soothsayer model --version=2.3.0 --set-default
```

**Deshabilitar modelo personalizado y usar defaults:**
```bash
soothsayer model --reset-to-default
```

## Deshacer Auto-Fixes

**Revertir fixes aplicados (basado en git):**
```bash
git revert --no-commit HEAD~N  # N = número de commits de fix
```

**Listar fixes aplicados:**
```bash
soothsayer history --applied-fixes
```
Salida:
```
Applied Fixes:
2024-03-07 10:23: fix-race-condition-abc123/src/worker.py
2024-03-06 14:45: fix-sql-injection-def456/auth/middleware.py
```

**Revertir selectivamente fix específico:**
```bash
soothsayer revert --fix-id=fix-race-condition-abc123
```

## Revertir Umbrales de Predicción

**Restaurar umbrales anteriores desde backup de configuración:**
```bash
cp .soothsayer.backup.yml .soothsayer.yml
soothsayer config --reload
```

## Rollback Completo del Sistema

**Si soothsayer causa problemas en CI:**
```bash
# Deshabilitar en pre-commit temporalmente
soothsayer disable --global --pr-mode

# O remover de pre-commit config completamente
sed -i '/soothsayer/d' .pre-commit-config.yaml

# Restaurar configuración de trabajo anterior
git checkout HEAD~1 -- .soothsayer.yml
```

---

# Pasos de Verificación

## Después de Instalación
```bash
soothsayer --version  # Debería imprimir 2.4.1
soothsayer validate-model --test-suite=./tests/smoke/  # Debería pasar
soothsayer analyze --lang=python ./tests/sample/  # Debería producir predicciones
```

## Después de Cambio de Configuración
```bash
soothsayer config --validate  # Verificar sintaxis YAML
soothsayer analyze --lang=python ./tests/sample/ --no-cache  # Asegurar caché no usada
```

## Después de Entrenamiento de Modelo
```bash
soothsayer validate-model --test-suite=./tests/validation/ --precision-threshold=0.85 --recall-threshold=0.75
# Código de salida 0 = modelo aceptable, non-zero = rechazar
```

## Prueba de Integración CI/CD
```bash
# Simular entorno CI
soothsayer check --ci --diff --format=compact > ci-output.txt
if [ $? -eq 2 ]; then
  echo "CI fallaría - problemas detectados"
  cat ci-output.txt
  exit 1
fi
```

---

# Solución de Problemas

## Problema: Análisis extremadamente lento (5+ minutos por archivo)
**Causa:** No hay caché configurado o directorio de caché lleno.
**Fix:**
```bash
# Verificar configuración de caché
soothsayer config --get cache_dir

# Limpiar y reconstruir caché
soothsayer cache --clear
soothsayer analyze --no-cache=false ./  # Repoblar caché

# Aumentar TTL de caché en .soothsayer.yml:
# cache_ttl_hours: 168  # (default 24)
```

## Problema: Errores "Model not found"
**Causa:** Modelos no descargados o corruptos.
**Fix:**
```bash
# Re-descargar todos los modelos (5-10GB)
soothsayer models --download --force

# Verificar instalación
soothsayer models --list
```

## Problema: Alta tasa de falsos positivos
**Causa:** Umbral de confianza demasiado bajo para tu código.
**Fix:**
```bash
# Aumentar umbral de confianza
soothsayer analyze --confidence-threshold=0.85 ./src/

# Persistir en config
soothsayer config --set confidence_threshold=0.85

# Reportar falsos positivos para mejorar modelos
soothsayer feedback --false-positive --file=utils.py --line=123 --pattern=race
```

## Problema: Sin memoria durante análisis
**Causa:** Archivos grandes o RAM insuficiente para modelos ML.
**Fix:**
```bash
# Usar modo batch (procesar archivos secuencialmente)
soothsayer analyze --batch-size=10 ./src/

# Excluir archivos generados grandes
# Añadir a .soothsayer.yml:
# exclude_paths:
#   - "**/migrations/*.py"
#   - "**/*.pb.go"

# O usar modelo más pequeño
soothsayer analyze --model=lightweight ./src/
```

## Problema: Jobs CI/CD timeout
**Causa:** Análisis completo en cada commit.
**Fix:**
```bash
# Siempre usar --diff en CI
soothsayer check --ci --diff --format=compact

# Cache a través de ejecuciones CI (GitLab example):
# before_script:
#   - soothsayer cache --restore --remote=ci-cache
# after_script:
#   - soothsayer cache --save --remote=ci-cache
```

## Problema: Pre-commit hooks fallando con "permission denied"
**Causa:** soothsayer no en PATH o shebang incorrecto.
**Fix:**
```bash
# Asegurar soothsayer en PATH para entorno de hook
which soothsayer  # Debería devolver ruta

# Reinstalar pre-commit hooks
pre-commit uninstall-all
pre-commit install

# O usar ruta absoluta en .pre-commit-config.yaml:
# - repo: local
#   hooks:
#     - id: soothsayer
#       entry: /usr/local/bin/soothsayer
```

## Problema: Predicciones no coinciden con fallos de runtime reales
**Causa:** Modelos pueden estar desactualizados o no entrenados en tu dominio.
**Fix:**
```bash
# Recolectar datos de fallos propios (últimos 6 meses de tickets de incidentes)
soothsayer collect-incidents --since=2024-01-01 --output=incidents.json

# Reentrenar con datos específicos de dominio
soothsayer learn --dataset=incidents.json --model-type=bug-classifier --epochs=30

# A/B test: comparar predicciones antiguas vs nuevas
soothsayer predict --model=default ./src/ > old-predictions.json
soothsayer predict --model=custom ./src/ > new-predictions.json
soothsayer compare old-predictions.json new-predictions.json
```

## Problema: Salida SARIF no mostrándose en GitHub Code Scanning
**Causa:** Formato de archivo SARIF incorrecto o demasiado grande.
**Fix:**
```bash
# Generar SARIF proper
soothsayer analyze --output=sarif --max-sarif-size=100MB ./src/ | gzip -c > results.sarif.gz

# Subir usando GitHub CLI
gh code-scanning upload --sarif results.sarif.gz

# O asegurar formato proper
soothsayer analyze --output=sarif --sarif-version=2.1.0 ./src/
```

---

# Variables de Entorno

- `SOOTHSAYER_HOME`: Sobrescribir directorio de configuración por defecto (`~/.soothsayer/`)
- `SOOTHSAYER_CACHE_DIR`: Sobrescribir directorio de caché
- `SOOTHSAYER_MODEL_DIR`: Sobrescribir ubicación de almacenamiento de modelos
- `SOOTHSAYER_CONFIG`: Ruta a archivo de configuración personalizado (sobrescribe `.soothsayer.yml`)
- `SOOTHSAYER_LOG_LEVEL`: `DEBUG`, `INFO`, `WARNING`, `ERROR` (default: `INFO`)
- `SOOTHSAYER_DISABLE_ANALYTICS`: Establecer a `true` para deshabilitar reportes de uso
- `SOOTHSAYER_MAX_WORKERS`: Número de workers de análisis en paralelo (default: número de núcleos CPU)
- `SOOTHSAYER_MEMORY_LIMIT_MB`: Uso máximo de memoria antes de throttling (default: 4096)

Ejemplo:
```bash
export SOOTHSAYER_CACHE_DIR=/mnt/ssd/soothsayer-cache
export SOOTHSAYER_MAX_WORKERS=8
soothsayer analyze --lang=python ./large-project/
```

---

# Códigos de Salida

- `0`: Éxito (sin problemas o problemas dentro del umbral)
- `1`: Problemas encontrados pero dentro de umbral aceptable (no fatal)
- `2`: Problemas exceden umbral de severidad/riesgo (fallar para CI)
- `3`: Error de configuración (YAML inválido, flags requeridos faltantes)
- `4`: Análisis fallido (crash, sin memoria, error carga modelo)
- `5`: Argumentos inválidos o comando desconocido
- `6`: Dependencia faltante (ejecutar `soothsayer doctor` para diagnosticar)

---

# Soporte

- Reportar bugs: `soothsayer report-bug`
- Ver documentación: `soothsayer docs --topic=troubleshooting`
- Verificar salud del sistema: `soothsayer doctor`
- Obtener info de versión: `soothsayer version --full`
```