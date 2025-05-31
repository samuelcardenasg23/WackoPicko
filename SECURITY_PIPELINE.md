# Security Pipeline Documentation

## 🛡️ Overview

Este proyecto incluye un pipeline de seguridad automatizado que utiliza **Fluid Attacks Skims** para analizar el código en busca de vulnerabilidades de seguridad.

## 🔧 How It Works

### When Does It Run?
El pipeline de seguridad se ejecuta automáticamente en los siguientes casos:
- **Push a main/master**: Cada vez que se hace push a la rama principal
- **Pull Requests**: En cada PR hacia main/master
- **Manual Execution**: Puede ejecutarse manualmente desde GitHub Actions

### What Does It Scan?
El scanner analiza:
- ✅ **PHP files** - Busca inyecciones SQL, XSS, inclusión de archivos, etc.
- ✅ **JavaScript files** - Detecta XSS, prototype pollution, unsafe eval
- ✅ **HTML files** - Revisa contenido estático por vulnerabilidades
- ✅ **SQL files** - Analiza consultas por problemas de seguridad
- ✅ **Configuration files** - Revisa configuraciones inseguras
- ✅ **Secrets** - Detecta credenciales hardcodeadas

### Security Checks Performed
- **SAST (Static Application Security Testing)**: Análisis estático del código
- **Secret Detection**: Búsqueda de credenciales expuestas
- **SCA (Software Composition Analysis)**: Análisis de dependencias vulnerables

## 📊 Understanding Results

### Severity Levels
- 🔴 **Critical**: Vulnerabilidades que requieren atención inmediata
- 🟠 **High**: Problemas serios que deben corregirse pronto
- 🟡 **Medium**: Vulnerabilidades moderadas a considerar
- 🔵 **Low**: Problemas menores o mejores prácticas

### Common Vulnerabilities Expected in This Project
Dado que este es un proyecto de práctica (WackoPicko), es probable que encuentre:

1. **SQL Injection**: Consultas no parametrizadas
2. **Cross-Site Scripting (XSS)**: Output no sanitizado
3. **Path Traversal**: Acceso no autorizado a archivos
4. **File Inclusion**: Inclusión insegura de archivos
5. **Command Injection**: Ejecución de comandos del sistema
6. **Weak Authentication**: Autenticación débil

## 📋 How to View Results

### In GitHub Actions
1. Ve a la pestaña **Actions** en tu repositorio
2. Selecciona el workflow **Security Analysis**
3. Haz clic en la ejecución más reciente
4. Descarga los artifacts **skims-security-report** (CSV) y **skims-json-report** (JSON)

### Local Analysis
También puedes ejecutar el scanner localmente usando Docker:

```bash
# Using Fluid Attacks Makes with Skims
docker run --rm -v $(pwd):/workspace \
  ghcr.io/fluidattacks/makes/amd64:latest \
  m gitlab:fluidattacks/universe@trunk /skims scan /workspace/.skims.yaml

# Alternative: Using direct Skims image (if available)
docker run --rm -v $(pwd):/workspace \
  --workdir /workspace \
  fluidattacks/skims:latest \
  skims scan .skims.yaml
```

## 🔧 Customizing the Pipeline

### Modifying Scanned Files
Edit `.skims.yaml` to change what gets scanned:

```yaml
include:
  - "**/*.php"      # Add or remove file patterns
  - "**/*.js"
  
exclude:
  - "tests/**"      # Exclude test files
  - "vendor/**"     # Exclude dependencies
```

### Changing Output Format
Modify the output configuration in `.skims.yaml`:

```yaml
output:
  file_path: "custom-results.csv"  # Change output filename
  format: "CSV"                    # Format options: CSV, JSON
```

### Adjusting Security Checks
Enable or disable specific checks:

```yaml
checks:
  sast: true        # Static analysis
  secrets: true     # Secret detection
  sca: false        # Disable dependency scanning
```

### Setting Failure Conditions
Modify the workflow to fail on certain conditions by adding this step:

```yaml
# Add this step after the scan to fail on critical vulnerabilities
- name: Check for Critical Issues
  run: |
    if [ -f "security-scan-results.csv" ]; then
      if grep -i "critical" security-scan-results.csv; then
        echo "Critical vulnerabilities found!"
        echo "Review the CSV file for details."
        exit 1
      fi
    fi
```

## 🎯 Best Practices

1. **Review Results Regularly**: Don't ignore security findings
2. **Fix Critical Issues First**: Prioritize by severity
3. **Test Fixes**: Ensure fixes don't break functionality
4. **Document Exceptions**: If you can't fix something, document why
5. **Keep Tools Updated**: Regularly update the Makes/Skims image

## 🆘 Troubleshooting

### Common Issues

#### Pipeline Fails with Docker Image Error
- **Problem**: Cannot pull or run the Fluid Attacks image
- **Solution**: Check internet connectivity and Docker Hub access

#### No Results Generated
- **Problem**: Scan completes but no CSV/JSON files are created
- **Solution**: 
  - Check if the `.skims.yaml` file is properly formatted
  - Verify that files to scan exist in the repository
  - Review the scan logs for errors

#### Large Scan Times
- **Problem**: Scans take too long to complete
- **Solution**: 
  - Exclude unnecessary directories in `.skims.yaml`
  - Reduce the scope of files being scanned
  - Increase timeout in configuration if needed

#### False Positives
- **Problem**: Scanner reports issues that aren't real vulnerabilities
- **Solution**: 
  - Review findings carefully
  - Adjust scanner configuration to reduce false positives
  - Document legitimate exceptions

### Getting Help
- Check [Fluid Attacks Documentation](https://help.fluidattacks.com/)
- Review scanner output for specific error messages
- Check GitHub Actions logs for detailed error information
- Consult security team for complex vulnerabilities

## 📚 Learning Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [PHP Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/PHP_Configuration_Cheat_Sheet.html)
- [Secure Coding Practices](https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/)
- [Fluid Attacks Documentation](https://help.fluidattacks.com/)

## 🔄 Pipeline Updates

### Recent Changes (v2.0)
- **Fixed Docker Image**: Changed from `fluidattacks/skims:latest` to `ghcr.io/fluidattacks/makes/amd64:latest`
- **Updated Command**: Now uses `m gitlab:fluidattacks/universe@trunk /skims scan` command
- **Improved Output**: Better handling of CSV and JSON results
- **Enhanced Error Handling**: More robust artifact collection

### Configuration Format
The `.skims.yaml` format has been updated to work with the Makes pipeline:
- Single output file instead of multiple formats
- Simplified configuration structure
- Better compatibility with CI/CD environments 