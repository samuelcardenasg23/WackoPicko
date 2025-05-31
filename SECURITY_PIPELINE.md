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
4. Descarga el artifact **skims-security-report**

### Local Analysis
También puedes ejecutar el scanner localmente:

```bash
# Using Docker directly
docker run --rm -v $(pwd):/workspace fluidattacks/skims:latest \
  --working-dir /workspace \
  --output /workspace/results \
  scan /workspace

# Using the configuration file
docker run --rm -v $(pwd):/workspace fluidattacks/skims:latest \
  --config /workspace/.skims.yaml \
  scan /workspace
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

### Adjusting Security Checks
Enable or disable specific checks:

```yaml
checks:
  sast: true        # Static analysis
  secrets: true     # Secret detection
  sca: false        # Disable dependency scanning
```

### Setting Failure Conditions
Modify the workflow to fail on certain conditions:

```yaml
# Add this step to fail on critical vulnerabilities
- name: Check for Critical Issues
  run: |
    if grep -q '"severity": "critical"' skims-results/*.json; then
      echo "Critical vulnerabilities found!"
      exit 1
    fi
```

## 🎯 Best Practices

1. **Review Results Regularly**: Don't ignore security findings
2. **Fix Critical Issues First**: Prioritize by severity
3. **Test Fixes**: Ensure fixes don't break functionality
4. **Document Exceptions**: If you can't fix something, document why
5. **Keep Tools Updated**: Regularly update the Skims image

## 🆘 Troubleshooting

### Common Issues
- **Long scan times**: Large codebases may take time
- **False positives**: Review findings carefully
- **Access permissions**: Ensure GitHub Actions has proper permissions

### Getting Help
- Check [Fluid Attacks Documentation](https://help.fluidattacks.com/)
- Review scanner output for specific error messages
- Consult security team for complex vulnerabilities

## 📚 Learning Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [PHP Security Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/PHP_Configuration_Cheat_Sheet.html)
- [Secure Coding Practices](https://owasp.org/www-project-secure-coding-practices-quick-reference-guide/) 