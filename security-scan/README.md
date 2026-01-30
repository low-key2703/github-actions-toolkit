# Security Scanner Action

Scan Docker images for vulnerabilities using Trivy security scanner.

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `image` | ✅ Yes | - | Docker image to scan (e.g., `nginx:latest`) |
| `severity` | ❌ No | `CRITICAL,HIGH` | Severity levels to check |
| `format` | ❌ No | `table` | Output format: `table`, `json`, `sarif` |
| `exit-code` | ❌ No | `1` | Exit code on vulnerabilities (0=continue, 1=fail) |
| `output-file` | ❌ No | `''` | Save report to file (optional) |

## Severity Levels

- `CRITICAL` - Critical vulnerabilities
- `HIGH` - High severity
- `MEDIUM` - Medium severity
- `LOW` - Low severity
- `UNKNOWN` - Unknown severity

## Usage Examples

### Basic Scan
```yaml
- name: Scan Docker image
  uses: ./security-scan
  with:
    image: 'nginx:latest'
```

### Scan with Custom Severity
```yaml
- name: Scan critical and high only
  uses: ./security-scan
  with:
    image: 'myapp:1.0.0'
    severity: 'CRITICAL,HIGH'
```

### Continue on Vulnerabilities (Non-blocking)
```yaml
- name: Scan but don't fail
  uses: ./security-scan
  with:
    image: 'myapp:dev'
    severity: 'CRITICAL,HIGH,MEDIUM'
    exit-code: '0'
```

### Save Report to File
```yaml
- name: Scan and save JSON report
  uses: ./security-scan
  with:
    image: 'myapp:1.0.0'
    format: 'json'
    output-file: 'trivy-report.json'
```

### GitHub Security Tab Integration (SARIF)
```yaml
- name: Scan and upload to Security tab
  uses: ./security-scan
  with:
    image: 'myapp:1.0.0'
    format: 'sarif'
    output-file: 'trivy-results.sarif'

- name: Upload results to GitHub Security
  uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: 'trivy-results.sarif'
```

### Complete CI/CD Example
```yaml
name: Build and Scan

on: [push]

jobs:
  build-and-scan:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Build Docker image
        run: docker build -t myapp:${{ github.sha }} .

      - name: Scan for vulnerabilities
        uses: ./security-scan
        with:
          image: 'myapp:${{ github.sha }}'
          severity: 'CRITICAL,HIGH'
          format: 'table'
          exit-code: '1'
```

## Best Practices

- **Development:** Use `exit-code: '0'` to warn but not block
- **Production:** Use `exit-code: '1'` to enforce security
- **SARIF format:** Integrate with GitHub Security tab for tracking
- **Regular scans:** Run on every build to catch new vulnerabilities

## What is Trivy?

Trivy is a comprehensive security scanner that detects:
- OS package vulnerabilities (Alpine, RHEL, CentOS, etc.)
- Application dependencies (npm, pip, gem, etc.)
- IaC misconfigurations
- Secrets and licenses
