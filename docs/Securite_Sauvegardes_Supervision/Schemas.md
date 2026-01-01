# **Schémas**

Cette section regroupe les schémas utilisés pour illustrer les mécanismes de sauvegarde et de supervision de la base WMS.  

---

# **1. Schéma de la stratégie de sauvegardes (MySQL)**

```mermaid
flowchart LR
    A["MySQL PRIMARY (Lille)"] --> B["Binlogs (toutes les 15 min)"]
    A --> C["Full Backup (mysqldump quotidien)"]
    A --> D["MySQL REPLICA (Lens)"]

    C --> E["NAS local (RAID)"]
    B --> E

    C --> F["Azure Cloud Storage"]
    B --> F
```

---

# **2. Schéma du pipeline de supervision**

```mermaid
flowchart LR
    A["Base MySQL WMS"] --> B["performance_schema / metrics exporter"]
    B --> C["Serveur de supervision (Zabbix / Grafana)"]
    C --> D["Alerting (Mail / Teams)"]
    D --> E["Administrateur SGBD / Support N2"]
```

---

# **3. Conclusion**

Ces schémas illustrent visuellement les mécanismes essentiels qui garantissent la continuité et la disponibilité du WMS :

- une stratégie de sauvegarde robuste (binlogs + full backups + redondance NAS/Azure)
    
- une supervision proactive permettant de détecter les anomalies avant qu’elles n’affectent l’activité
