# Security Pipeline Documentation

## 🛡️ Overview

Este proyecto utiliza un pipeline simplificado con la **imagen oficial de Fluid Attacks** (`docker.io/fluidattacks/skims:latest`) y captura el output para generar reportes en CSV.

## 🎯 Simple and Clean Approach

### ✅ Advantages:
- **Imagen oficial** - Usa la imagen recomendada en la documentación
- **Sin logs innecesarios** - No más 6000 líneas de logs de makes
- **Rápido** - Sin overhead de la imagen makes
- **Output capturado** - Guardamos todo el output para análisis
- **CSV generado** - Creamos formato CSV útil

## 🔧 How It Works

### Pipeline Structure
```yaml
# Simple workflow with official image
- name: Checkout repository
  uses: actions/checkout@v4

# Run scan and capture ALL output
- name: Run Skims scan and capture output
  run: |
    docker run --rm -v $(pwd):/workspace \
      docker.io/fluidattacks/skims:latest \
      skims scan /workspace > results/skims-output.txt 2>&1

# Check what files were generated
- name: Check for generated files
  run: |
    ls -la
    find . -name "*.csv" -type f

# Create CSV if needed
- name: Create CSV from output if needed
  run: |
    if [ ! -f "vulnerabilities.csv" ]; then
      echo "title,description,file,line,severity" > results/manual-results.csv
      grep -i "vulnerability\|finding" results/skims-output.txt >> results/manual-results.csv
    fi

# Upload everything
- name: Upload scan results
  uses: actions/upload-artifact@v4
  with:
    name: security-scan-results
    path: |
      results/
      *.csv
```

## 📊 What You Get

### Files Generated:
1. **`results/skims-output.txt`** - Todo el output completo del scan
2. **`vulnerabilities.csv`** - CSV generado por Skims (si funciona)
3. **`results/manual-results.csv`** - CSV creado por nosotros como fallback

### Configuration File:
Tenemos un archivo `.skims.yaml` simple:
```yaml
# Simple Skims Configuration for WackoPicko
namespace: "wackopicko-scan"
working_dir: "."

# Output settings - simple CSV format
output:
  format: "CSV"
  file_path: "vulnerabilities.csv"

# Focus on critical vulnerabilities
checks:
  F001: true  # SQL Injection
  F004: true  # Cross-Site Scripting (XSS)  
  F008: true  # Path Traversal
  F021: true  # Path Injection
  F034: true  # Unvalidated Redirect
  F091: true  # Command Injection
  F127: true  # Open Redirect

# Simple paths
include:
  - "website/"
  - "."

exclude:
  - ".git/"
  - "node_modules/"
```

## 📋 How to View Results

### In GitHub Actions
1. Ve a **Actions** → **Security Analysis**
2. Haz clic en la ejecución más reciente
3. Ve a la sección **Summary** para ver:
   - Primeras 20 líneas del output
   - Últimas 20 líneas del output
   - Lista de archivos generados
4. Descarga el artifact **security-scan-results**

### What to Look For in Logs:
```
=== First 20 lines of scan output ===
[INFO] Skims starting...
[INFO] Scanning directory: /workspace
[INFO] Found vulnerabilities in: website/test.php

=== Last 20 lines of scan output ===
[INFO] Scan completed
[INFO] Vulnerabilities found: 5
[INFO] Report written to: vulnerabilities.csv
```

## 📈 Expected Results for WackoPicko

### CSV Format Example:
```csv
title,description,file,line,severity
Cross-site scripting,XSS vulnerability found,website/test.php,2,High
SQL injection,Possible SQL injection,website/login.php,15,Critical
Path traversal,Directory traversal risk,website/upload.php,8,Medium
```

## 🔧 Local Testing

```bash
# Test locally with official image
docker run --rm -v $(pwd):/workspace \
  docker.io/fluidattacks/skims:latest \
  skims scan /workspace

# Capture output locally
docker run --rm -v $(pwd):/workspace \
  docker.io/fluidattacks/skims:latest \
  skims scan /workspace > local-scan-output.txt 2>&1

# Check what was generated
cat local-scan-output.txt
ls -la *.csv
```

## 🆘 Troubleshooting

### If No Vulnerabilities Found:
1. ✅ Check `results/skims-output.txt` para ver qué pasó
2. ✅ Verifica que `website/test.php` existe y tiene contenido
3. ✅ Mira si hay errores en el output
4. ✅ WackoPicko **debe** tener vulnerabilidades

### Understanding the Output:
```bash
# Download the artifact and check:
cat results/skims-output.txt | grep -i "vulnerability"
cat results/skims-output.txt | grep -i "error"
cat results/skims-output.txt | grep -i "found"
```

### If Skims Generates No CSV:
El pipeline automáticamente crea `results/manual-results.csv` extrayendo información útil del output de texto.

## 🎓 Next Steps

1. **Ejecuta el pipeline** y verifica que funciona sin errores
2. **Revisa los logs** en el summary del workflow
3. **Descarga el artifact** y explora los archivos generados
4. **Analiza las vulnerabilidades** encontradas en WackoPicko

---

**Ventaja clave**: Máxima simplicidad con la imagen oficial, sin perder funcionalidad ni generar logs innecesarios. 