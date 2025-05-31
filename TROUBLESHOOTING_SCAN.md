# Troubleshooting Guide: Skims Not Finding Vulnerabilities

## 🚨 Problem Description

WackoPicko is an **intentionally vulnerable** web application, but Skims reported:
> "Summary: No vulnerabilities were found in your targets."

This is **definitely incorrect** and indicates a configuration or scanning issue.

## 🔍 Root Cause Analysis

### What We Found:
1. **Configuration warnings**: `Some keys were not recognized: include, exclude, languages, severity, settings`
2. **Git context issues**: `Unable to find commit HEAD on analyzed directory`
3. **Wrong configuration format**: Our YAML format wasn't compatible with the Skims version

### Expected Vulnerabilities in WackoPicko:

Looking at `website/test.php`, we should find at least:
```php
<?php
$head = $_GET['head'];     // ❌ XSS vulnerability
$title = $_GET['title'];   // ❌ XSS vulnerability  
$href = $_GET['href'];     // ❌ Open redirect
$script = $_GET['script']; // ❌ Code injection
?>
<script>
   <?php echo $script; ?>  // ❌ Direct script injection
</script>
```

## 🛠️ Solutions Implemented

### 1. Simplified Configuration Format
**Changed from complex format to specific vulnerability checks:**

```yaml
# Old format (didn't work)
checks:
  appsec: true
  sast: true
  
# New format (should work)
checks:
  F001: true  # SQL Injection
  F004: true  # Cross-Site Scripting (XSS)
  F091: true  # Command Injection
```

### 2. Enhanced Pipeline Debugging
**Added debugging steps to see what's happening:**
- List files being scanned
- Show scan output in real-time
- Try alternative scan methods
- Check all generated files

### 3. Fallback Scanning Method
**Added a backup scan without config file:**
```bash
/skims scan --output results.csv .
```

## 🧪 Testing the Fix

### Expected Results After Fix:
1. **Files should be detected**: PHP files in `website/` directory
2. **Vulnerabilities should be found**: At least XSS, injection, etc.
3. **CSV file should contain findings**: Not just a summary line

### Verification Commands:
```bash
# Check if files are being scanned
find . -name "*.php" -type f

# Verify test.php content
cat website/test.php

# Check scan results
cat *.csv
```

## 🔧 Alternative Scanning Methods

If the main pipeline still fails, try these manual approaches:

### Method 1: Direct Docker Command
```bash
docker run --rm -v $(pwd):/workspace \
  ghcr.io/fluidattacks/makes/amd64:latest \
  m gitlab:fluidattacks/universe@trunk /skims scan --output manual-results.csv /workspace
```

### Method 2: Scan Specific File
```bash
docker run --rm -v $(pwd):/workspace \
  ghcr.io/fluidattacks/makes/amd64:latest \
  m gitlab:fluidattacks/universe@trunk /skims scan --output test-results.csv /workspace/website/test.php
```

### Method 3: Use Different Configuration
Create a minimal config:
```yaml
namespace: "test-scan"
working_dir: "."
output:
  file_path: "minimal-results.csv"
  format: "CSV"
```

## 📊 Understanding Scan Results

### Valid Results Should Look Like:
```csv
title,cwe,description,cvss,cvss_v4,finding,stream,kind,where,snippet,method
XSS Injection,CWE-79,"Cross-site scripting vulnerability",7.5,7.1,F004,website/test.php,lines,5,"echo $_GET['head']",SAST
```

### Invalid Results (What We Got):
```csv
title,cwe,description,cvss,cvss_v4,finding,stream,kind,where,snippet,method
Summary: No vulnerabilities were found in your targets.
```

## 🚀 Next Steps

1. **Commit the updated configuration** (`.skims.yaml`)
2. **Push changes** to trigger the new pipeline
3. **Monitor the "Check scan results" step** for debugging info
4. **Download artifacts** to verify actual content
5. **If still no results**, try the manual Docker commands locally

## 📝 Key Learnings

### What We Learned:
- Skims configuration format is **very specific**
- The `makes` wrapper adds complexity but is necessary
- GitHub Actions environment may behave differently than local
- Always verify your test application actually has vulnerabilities

### Red Flags to Watch For:
- ❌ Configuration warnings about unrecognized keys
- ❌ Empty or minimal CSV files (< 100 characters)
- ❌ "No vulnerabilities found" on intentionally vulnerable apps
- ❌ Git context warnings

### Success Indicators:
- ✅ Multiple F### vulnerability codes detected
- ✅ CSV files with detailed findings
- ✅ File paths pointing to actual vulnerable code
- ✅ CWE numbers and CVSS scores present

## 🆘 If All Else Fails

### Contact Points:
1. **Fluid Attacks Documentation**: https://help.fluidattacks.com/
2. **GitHub Community Discussions**: Search for "Fluid Attacks Skims"
3. **Local Testing**: Try running Docker commands manually

### Escalation Steps:
1. Test with a simple, obviously vulnerable file
2. Try different Docker images (if available)
3. Contact your security team for alternative tools
4. Consider using other SAST tools as backup (SonarQube, Semgrep, etc.)

---

**Remember**: The goal is learning! Even troubleshooting scanning tools teaches valuable lessons about security tooling and CI/CD pipelines. 