# Project 1: Active Directory Monitoring with Grafana

## 1. Introduction
Grafana is an open-source platform for monitoring and observability. It provides a powerful and flexible dashboard to visualize Active Directory metrics, making it easier to monitor and manage AD health and performance.

## 2. Architecture & Lab Setup
**Infrastructure Requirements :**
- **Windows Server 2022 :** Active Directory Domain Services (AD DS) - *Serveur surveillé (Source des données)*.
- **Ubuntu Server :** *Serveur de supervision*.

**Tools :**
- **Prometheus Windows Exporter** (installé sur Windows) : Collecte les métriques AD et les expose sur le port 9182.
- **Prometheus** (installé sur Ubuntu) : Aspire et stocke les données depuis l'exportateur.
- **Grafana** (installé sur Ubuntu) : Se connecte à Prometheus pour visualiser les données sur des tableaux de bord (port 3000).

---

## 3. Tool Installation (Guide Détaillé)

### Étape 1 : Installer Prometheus Windows Exporter (Sur le Windows Server 2022)
1. Téléchargez le *Windows Exporter* (fichier `.msi`) depuis le GitHub officiel de Prometheus.
2. Ouvrez un terminal **PowerShell** en tant qu'administrateur et exécutez :
   ```powershell
   Start-Process -FilePath .\windows_exporter-<version>-amd64.msi -ArgumentList /quiet
   ```
3. Configurez l'exportateur pour qu'il s'exécute en tant que service automatique :
   ```powershell
   New-Service -Name "wmi_exporter" -BinaryPathName "C:\Program Files\wmi_exporter\wmi_exporter.exe" -DisplayName "WMI Exporter" -StartupType Automatic
   Start-Service -Name "wmi_exporter"
   ```
*Important : Assurez-vous que le port TCP `9182` est ouvert dans le Pare-feu Windows pour autoriser le serveur Ubuntu à lire les données.*

### Étape 2 : Installer Prometheus (Sur le Serveur Ubuntu)
1. Téléchargez et extrayez Prometheus :
   ```bash
   wget [https://github.com/prometheus/prometheus/releases/download/v2.51.1/prometheus-2.51.1.linux-amd64.tar.gz](https://github.com/prometheus/prometheus/releases/download/v2.51.1/prometheus-2.51.1.linux-amd64.tar.gz)
   tar xvf prometheus-2.51.1.linux-amd64.tar.gz
   cd prometheus-2.51.1.linux-amd64
   ```
2. Modifiez le fichier `prometheus.yml` pour cibler votre serveur Windows. Ajoutez ceci à la fin du fichier (remplacez `IP_WINDOWS` par l'adresse IP de votre serveur Windows 2022) :
   ```yaml
   scrape_configs:
     - job_name: 'windows_exporter'
       static_configs:
         - targets: ['IP_WINDOWS:9182']
   ```
3. Lancez Prometheus :
   ```bash
   ./prometheus --config.file=prometheus.yml
   ```

### Étape 3 : Installer Grafana (Sur le Serveur Ubuntu)
1. Installez les prérequis et ajoutez le dépôt officiel de Grafana :
   ```bash
   sudo apt-get install -y apt-transport-https software-properties-common wget
   sudo mkdir -p /etc/apt/keyrings/
   wget -q -O - [https://apt.grafana.com/gpg.key](https://apt.grafana.com/gpg.key) | gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
   echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] [https://apt.grafana.com](https://apt.grafana.com) stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
   ```
2. Installez, démarrez et activez Grafana :
   ```bash
   sudo apt-get update
   sudo apt-get install grafana -y
   sudo systemctl daemon-reload
   sudo systemctl start grafana-server
   sudo systemctl enable grafana-server
   ```
3. Accédez à l'interface web via `http://<IP_UBUNTU>:3000` (Identifiants par défaut : `admin` / `admin`).

---

## 4. Exercises

### Exercise 1: Setting up Prometheus Data Source in Grafana
**Steps:**
1. Log in to Grafana.
2. Navigate to **Configuration > Data Sources**.
3. Click **Add data source** and select **Prometheus**.
4. Enter the Prometheus server URL (e.g., `http://localhost:9090` or your Ubuntu IP) and click **Save & Test**.
**Expected Output:** Grafana successfully connects to Prometheus and is ready to visualize data.

### Exercise 2: Creating a Dashboard for AD Metrics
**Steps:**
1. Go to **Create > Dashboard**.
2. Click **Add new panel**.
3. Select a metric from Prometheus related to AD (e.g., `windows_logical_disk_free_bytes`).
4. Configure the visualization type (e.g., graph, gauge).
5. Click **Apply**.
**Expected Output:** A dashboard panel displaying real-time AD metrics is created.

### Exercise 3: Monitoring AD User Logon Events
**Steps:**
1. Ensure the Windows Exporter is configured to collect Event Log metrics.
2. Create a new panel in Grafana.
3. Select a metric for user logon events (e.g., `windows_eventlog_security_logon`).
4. Apply filters to focus on specific event IDs.
**Expected Output:** A panel showing user logon events and their frequency is displayed.

### Exercise 4: Setting Up Alerts for AD Performance Issues
**Steps:**
1. Edit a panel showing critical AD metrics (e.g., CPU usage).
2. Go to the **Alert** tab.
3. Define alert conditions (e.g., if CPU usage > 80% for 5 minutes).
4. Configure notification channels (e.g., email, Slack).
5. Click **Save**.
**Expected Output:** Alerts are set up to notify administrators of potential AD performance issues.

### Exercise 5: Visualizing AD Security Events
**Steps:**
1. Create a new dashboard specifically for security events.
2. Add panels for various security metrics (e.g., failed logon attempts, account lockouts).
3. Use appropriate filters to refine the data displayed.
4. Arrange panels for a comprehensive security overview.
**Expected Output:** A security-focused dashboard displaying key AD security metrics.

***
