# DEVSECOPS

---

# ISS-DevSecOPS-Tool

This repository leverages GitHub Actions to automate security and compliance scanning for your codebase. The workflow, **ISS-DevSecOPS-Tool**, integrates multiple industry-standard security tools to ensure your project is continuously monitored for vulnerabilities, misconfigurations, and compliance issues.

## Workflow Overview

This workflow is triggered automatically on every push to the `main` branch. It performs the following security and compliance checks:

1. **Static Application Security Testing (SAST) with Semgrep**
2. **Vulnerability, Secret, and Misconfiguration Scanning with Trivy**
3. **Infrastructure as Code (IaC) Analysis with Checkov**
4. **Software Bill of Materials (SBOM) Generation with Anchore**

All scan results are uploaded to the GitHub Security tab for easy review and tracking.

---

## Workflow Details

### Trigger

- **Event:** `push` to the `main` branch

### Permissions

- **actions:** read
- **contents:** read
- **security-events:** write (required for uploading SARIF results)

### Jobs and Steps

#### 1. Checkout Repository

```yaml
- uses: actions/checkout@v4
```
Checks out the repository code so it can be scanned by the subsequent tools.

#### 2. Run Semgrep Security Scan

```yaml
- name: Run Semgrep security scan
  uses: docker://returntocorp/semgrep:latest
  with:
    entrypoint: /bin/sh
    args: -c "semgrep scan -q --sarif --config auto . > semgrep-results.sarif"
```
Performs static code analysis using Semgrep and outputs results in SARIF format.

#### 3. Upload Semgrep Results

```yaml
- name: Upload Semgrep SARIF results
  uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: semgrep-results.sarif
```
Uploads Semgrep scan results to the GitHub Security tab.

#### 4. Run Trivy Filesystem Scan

```yaml
- name: Run Trivy filesystem scan
  uses: aquasecurity/trivy-action@master
  with:
    scan-type: 'fs'
    scan-ref: ${{ github.workspace }}
    scanners: 'vuln,secret,misconfig'
    severity: 'CRITICAL,HIGH,MEDIUM,LOW,UNKNOWN'
    format: 'sarif'
    exit-code: 0
    output: 'trivy-results.sarif'
```
Scans the repository for vulnerabilities, secrets, and misconfigurations using Trivy.

#### 5. Upload Trivy Results

```yaml
- name: Upload Trivy scan results to GitHub Security tab
  uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: 'trivy-results.sarif'
```
Uploads Trivy scan results to the GitHub Security tab.

#### 6. Run Checkov IaC Scan

```yaml
- name: Checkov IaC Scan
  uses: bridgecrewio/checkov-action@master
  with:
    directory: .
    output_format: cli,sarif
    output_file_path: console,checkov-results.sarif
```
Analyzes Infrastructure as Code (IaC) files for security and compliance issues using Checkov.

#### 7. Upload Checkov Results

```yaml
- name: Upload Checkov SARIF results
  uses: github/codeql-action/upload-sarif@v3
  with:
    sarif_file: checkov-results.sarif
```
Uploads Checkov scan results to the GitHub Security tab.

#### 8. Generate SBOM

```yaml
- name: Generate SBOM
  uses: anchore/sbom-action@v0
  with:
    format: spdx-json  
    upload-artifact: true 
    upload-release-assets: false
```
Generates a Software Bill of Materials (SBOM) in SPDX JSON format and uploads it as a workflow artifact.

---

## Benefits

- **Automated Security:** Continuous scanning for vulnerabilities, secrets, and misconfigurations.
- **Compliance:** Infrastructure as Code (IaC) checks ensure best practices and compliance.
- **Transparency:** SBOM generation helps track dependencies and supply chain risks.
- **Centralized Reporting:** All results are available in the GitHub Security tab for easy access and review.

---

## Getting Started

1. **Fork or clone this repository.**
2. **Push code to the `main` branch** to trigger the workflow.
3. **Review scan results** in the GitHub Security tab and download the SBOM artifact as needed.

---

## Customization

- **Schedule Scans:** Add a `schedule` trigger to run scans periodically.
- **Branch Selection:** Modify the `branches` array to scan other branches.
- **Tool Configuration:** Adjust tool-specific arguments to fit your project’s needs.

## Contact me
@ https://github.com/TennyV 

