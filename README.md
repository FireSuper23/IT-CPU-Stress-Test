A guide to help you run a **1-hour hardware stress test** using the terminal. These methods help verify system stability, thermal performance, and hardware integrity during IT projects without requiring standard software installations.

---

## 🐧 Linux Terminal Methods

### Option A: The Dedicated Tool (`stress-ng`)
This is the industry standard for IT benchmarking. It runs a sequence of mathematical tests, monitors system metrics, and outputs a complete summary at the end.
You need to install the program, use your distribution's native package manager command in the terminal.
```bash
stress-ng --matrix 0 --timeout 1h --metrics
```
* **`--matrix 0`**: Runs complex mathematical memory/CPU matrices across all available cores.
* **`--timeout 1h`**: Automatically terminates the test after 1 hour.
* **`--metrics`**: Outputs a clear data table showing total operations and system error states upon completion.

### Option B: The Built-In Native Tool (No Downloads)
If you cannot download any external tools, this native Bash script divides 1 hour into 60 individual 1-minute test cycles, tracking passes and failures.

```bash
TOTAL_TESTS=60; TEST_DURATION=60; PASSED=0; FAILED=0; REPORT=()
CORES=\$(nproc)

for i in (seq 1 TOTAL_TESTS); do
    echo "Running Test \$i/\(TOTAL_TESTS for\){TEST_DURATION}s..."
    timeout \${TEST_DURATION}s sh -c "for c in \$(seq \$CORES); do yes > /dev/null & done; wait" 2>/dev/null
    if [ \(? -eq 124 ] \vert{}\vert{} [\)? -eq 0 ]; then
        ((PASSED++))
        REPORT+=("Test \$i: PASS")
        echo -e "\e[32m-> Test \$i: PASS\e[0m"
    else
        ((FAILED++))
        REPORT+=("Test \$i: FAILED")
        echo -e "\e[31m-> Test \$i: FAILED\e[0m"
    fi
done

clear
echo -e "\e[33m=== 1-HOUR STRESS TEST SUMMARY ===\e[0m"
for line in "\({REPORT[@]}"; do echo "\)line"; done
echo -e "\e[33m==================================\e[0m"
echo -e "\e[32mTOTAL PASSED: \$PASSED\e[0m"
echo -e "\e[31mTOTAL FAILED: \$FAILED\e[0m"
```

---

## 🪟 Windows Terminal Methods

### Option A: The Portable Industry Tool (`Prime95`)
The most common program used by IT engineers. It is a portable executable (`.exe`) that doesn't require an installer. It performs deep mathematical validation (Fast Fourier Transforms) and flags hardware calculation errors.

1. Download the portable zip from [Mersenne Prime95](https://mersenne.org) and extract it.
2. Open PowerShell or Command Prompt as **Administrator** inside that folder.
3. Run the following command:

```powershell
.\(\prime95.\)exe -t -m3600
```
* **`-t`**: Forces the application directly into Torture Test mode.
* **`-m3600`**: Sets a hard countdown timer for exactly 3600 seconds (1 hour). 
* If a CPU core miscalculates or drops a thread due to hardware instability, the program logs the failure instantly to `results.txt`.

### Option B: The Built-In Native Script (No Downloads)
If you must test a Windows machine using absolutely zero third-party software, paste this native script into **PowerShell (Admin)**. It sequences 60 consecutive 1-minute stress cycles.

```powershell
totalTests = 60; testDuration = 60; passed = 0; failed = 0; \(report = @()\)cores = (Get-CimInstance Win32_ComputerSystem).NumberOfLogicalProcessors

for (i = 1; i -le totalTests; i++) {
    Write-Host "Running Test \$i/totalTests for testDuration seconds..." -ForegroundColor Cyan
    jobs = 1..cores | ForEach-Object { 
        Start-Job -ScriptBlock { 
            \$sw = [System.Diagnostics.Stopwatch]::StartNew()
            while (sw.Elapsed.TotalSeconds -lt using:testDuration) { [math]::Sqrt([int]::MaxValue) } 
        } 
    }
    \$null = jobs | Wait-Job -Timeout (testDuration + 5)
    testFailed = false
    foreach (j in jobs) {
        if (\$j.State -ne "Completed") { testFailed = true }
        Remove-Job \$j -Force
    }
    if (\(testFailed) {\)failed++; report += "Test i: FAILED"
        Write-Host "-> Test \$i: FAILED" -ForegroundColor Red
    } else {
        \$passed++; report += "Test i: PASS"
        Write-Host "-> Test \$i: PASS" -ForegroundColor Green
    }
}

Clear-Host
Write-Host "=== 1-HOUR STRESS TEST SUMMARY ===" -ForegroundColor Yellow
report | ForEach-Object Write-Host _ }
Write-Host "==================================" -ForegroundColor Yellow
Write-Host "TOTAL PASSED: \$passed" -ForegroundColor Green
Write-Host "TOTAL FAILED: failed" -ForegroundColor (failed -gt 0 ? "Red" : "Gray")
```

---

## 📊 Interpreting Results

* **FINISHED / PASS**: The system maintained stability for the full 60 minutes. The cooling solution is sufficient and the hardware calculations are accurate.
* **FAILED**: The background test threads crashed, or the tool flagged an error. This indicates unstable hardware, corrupted voltage settings, or thermal throttling.
* **System Crash (BSOD / Kernel Panic)**: If the system completely freezes or blue-screens during the test, the hardware failed the load requirements.
