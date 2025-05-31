# Security Pipeline Documentation

## 🛡️ Overview

Este proyecto incluye un pipeline de seguridad que utiliza **Fluid Attacks Skims** a través de la imagen **makes** que es la que funciona correctamente con los parámetros estándar.

## ⚠️ Important Note

Aunque la documentación oficial menciona `docker.io/fluidattacks/skims:latest`, hemos encontrado que esta imagen no soporta la flag `--output`. Por eso usamos `ghcr.io/fluidattacks/makes/amd64:latest` que incluye Skims con todas las funcionalidades.

## 🔧 How It Works

### When Does It Run?
- **Push a main/master**: Cada vez que se hace push a la rama principal
- **Pull Requests**: En cada PR hacia main/master
- **Manual Execution**: Puede ejecutarse manualmente desde GitHub Actions

### Working Pipeline Structure
```yaml
# Usando la imagen makes que funciona correctamente
- name: Checkout repository
  uses: actions/checkout@v4

# Generar reporte CSV
- name: Skims scan (CSV output)
  uses: docker://ghcr.io/fluidattacks/makes/amd64:latest
  with:
    args: m gitlab:fluidattacks/universe@trunk /skims scan --output results.csv .

# Generar reporte JSON
- name: Skims scan (JSON output)
  uses: docker://ghcr.io/fluidattacks/makes/amd64:latest
  with:
    args: m gitlab:fluidattacks/universe@trunk /skims scan --output results.json --format json .

# Subir ambos formatos
- name: Upload CSV results
  uses: actions/upload-artifact@v4
  with:
    name: security-report-csv
    path: results.csv

- name: Upload JSON results
  uses: actions/upload-artifact@v4
  with:
    name: security-report-json
    path: results.json
```

## 📊 What Gets Scanned
Skims automatically detects and scans:
- ✅ **PHP files** - Busca inyecciones SQL, XSS, inclusión de archivos
- ✅ **JavaScript files** - Detecta vulnerabilidades client-side
- ✅ **HTML files** - Revisa contenido estático
- ✅ **Configuration files** - Analiza configuraciones inseguras
- ✅ **Otros archivos** - Según su base de conocimiento

## 📋 How to View Results

### In GitHub Actions
1. Ve a la pestaña **Actions** en tu repositorio
2. Selecciona el workflow **Security Analysis**
3. Haz clic en la ejecución más reciente
4. Descarga los artifacts:
   - **security-report-csv** - Contiene `results.csv`
   - **security-report-json** - Contiene `results.json`

### Local Analysis (Optional)
```bash
# Ejecutar localmente con CSV
docker run --rm -v $(pwd):/workspace \
  ghcr.io/fluidattacks/makes/amd64:latest \
  m gitlab:fluidattacks/universe@trunk /skims scan --output results.csv /workspace

# Ejecutar localmente con JSON
docker run --rm -v $(pwd):/workspace \
  ghcr.io/fluidattacks/makes/amd64:latest \
  m gitlab:fluidattacks/universe@trunk /skims scan --output results.json --format json /workspace
```

## 📈 Expected Results for WackoPicko

Dado que WackoPicko es una aplicación intencionalmente vulnerable, deberías ver vulnerabilidades como:

1. **Cross-Site Scripting (XSS)** en `website/test.php`
2. **SQL Injection** en varios archivos PHP
3. **Path Traversal** y **File Inclusion**
4. **Command Injection**
5. **Weak Authentication**

## 🔍 Why We Use Makes Image

### ❌ Problem with Official Image
```bash
# Esta imagen NO funciona con --output
docker.io/fluidattacks/skims:latest
# Error: No such option: --output
```

### ✅ Solution with Makes Image
```bash
# Esta imagen SÍ funciona con todos los parámetros
ghcr.io/fluidattacks/makes/amd64:latest
# Funciona correctamente con --output y --format
```

## 🎯 Benefits of This Approach

### ✅ Advantages:
- **Funciona correctamente** - Soporta todas las opciones necesarias
- **Dos formatos** - CSV para análisis rápido, JSON para procesamiento automático
- **Imagen estable** - Parte del ecosistema oficial de Fluid Attacks
- **Artifacts separados** - Fácil descarga de cada formato

### 📝 What You Get:

**CSV Format (`results.csv`):**
```csv
title,cwe,description,cvss,cvss_v4,finding,stream,kind,where,snippet,method
Cross-site scripting,CWE-79,"XSS vulnerability",7.5,7.1,F004,website/test.php,lines,2,"echo $_GET['head']",SAST
```

**JSON Format (`results.json`):**
```json
{
  "vulnerabilities": [
    {
      "title": "Cross-site scripting",
      "cwe": "CWE-79", 
      "description": "XSS vulnerability",
      "cvss": 7.5,
      "file": "website/test.php",
      "line": 2
    }
  ]
}
```

## 🔧 Customization (Optional)

### Escanear solo un directorio específico:
```yaml
# Solo escanear la carpeta website/
args: m gitlab:fluidattacks/universe@trunk /skims scan --output results.csv ./website
```

### Agregar más opciones:
```yaml
# Diferentes opciones de salida
args: m gitlab:fluidattacks/universe@trunk /skims scan --output results.csv --verbose .
```

## 🆘 Troubleshooting

### Si no encuentras vulnerabilidades:
1. ✅ Verifica que ambos archivos se generaron (`results.csv` y `results.json`)
2. ✅ Descarga ambos artifacts y revisa el contenido
3. ✅ WackoPicko **debe** tener vulnerabilidades - si no aparecen, hay un problema
4. ✅ Compara resultados entre CSV y JSON para verificar consistencia

### Comandos útiles para verificar:
```bash
# Verificar archivos PHP en el proyecto
find . -name "*.php" -type f

# Ver contenido del archivo vulnerable
cat website/test.php

# Verificar resultados localmente (CSV)
docker run --rm -v $(pwd):/workspace \
  ghcr.io/fluidattacks/makes/amd64:latest \
  m gitlab:fluidattacks/universe@trunk /skims scan --output test.csv /workspace/website/test.php

# Verificar resultados localmente (JSON)
docker run --rm -v $(pwd):/workspace \
  ghcr.io/fluidattacks/makes/amd64:latest \
  m gitlab:fluidattacks/universe@trunk /skims scan --output test.json --format json /workspace/website/test.php
```

## 📚 Learning Resources

- [Fluid Attacks Official Guide](https://help.fluidattacks.com/portal/en/kb/articles/use-the-scanners-in-ci-cd#Run_on_GitHub_Actions)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [PHP Security Best Practices](https://cheatsheetseries.owasp.org/cheatsheets/PHP_Configuration_Cheat_Sheet.html)

## 🎓 Next Steps

1. **Ejecuta el pipeline** y verifica que se generen ambos formatos
2. **Compara CSV vs JSON** - aprende las ventajas de cada formato
3. **Analiza los resultados** - entiende cada vulnerabilidad encontrada
4. **Experimenta con fixes** - intenta corregir algunas vulnerabilidades
5. **Compara resultados** - ve cómo cambian los reportes después de fixes

## 📊 Format Comparison

### CSV - Mejor para:
- ✅ Análisis rápido en Excel/Google Sheets
- ✅ Lectura humana directa
- ✅ Reportes simples
- ✅ Comparación manual

### JSON - Mejor para:
- ✅ Procesamiento automático
- ✅ Integración con otras herramientas
- ✅ APIs y scripts
- ✅ Análisis programático

---

**Nota importante**: Aunque la documentación oficial menciona `docker.io/fluidattacks/skims:latest`, esta imagen tiene limitaciones. La imagen `makes` es más completa y funcional para CI/CD. 