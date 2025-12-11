# **04.3 – Supervision et Indicateurs Critiques (KPI)**

## **1. Introduction**

La base MySQL du WMS est un composant critique pour l’activité logistique : la moindre latence ou indisponibilité impacte directement les opérations d’entreposage.  
La supervision vise à détecter les anomalies, alerter les équipes techniques et garantir une remédiation rapide.  
Cette section définit les cinq indicateurs critiques (KPI), leurs seuils d’alerte et les actions correctives associées.

---

## **2. Objectifs de la supervision**

- Surveiller la disponibilité et la performance de la base.
    
- Détecter les ralentissements applicatifs (slow queries).
    
- Identifier les risques de saturation disque ou CPU.
    
- Vérifier en continu l’état de la réplication MySQL.
    
- Contrôler la fraîcheur des sauvegardes binlogs et full backups.
    
- Alerter automatiquement le support en cas d’anomalie.

---

# **3. Indicateurs critiques (KPI)**

---

## **3.1 KPI 1 – Replication Lag MySQL**

**Description :** 

Mesure le décalage entre le primary et le replica via `Seconds_Behind_Master`.

**Seuils :**

- Warning : **> 30 sec**
    
- Critique : **> 60 sec**

**Actions correctives :**

- Vérifier la charge I/O du primary.
    
- Vérifier la bande passante réseau entre les sites.
    
- Inspecter les binlogs (`SHOW BINARY LOGS`).
    
- Vérifier l’état de la réplication :
    
    `SHOW SLAVE STATUS\G;`
    
- Relancer la synchronisation si nécessaire.

---

## **3.2 KPI 2 – Slow Queries**

**Description :**  
Pourcentage de requêtes dépassant **200 ms**, basé sur performance_schema.

**Seuils :**

- Warning : **> 10 %** de requêtes > 200 ms
    
- Critique : **> 25 %**

**Actions correctives :**

- Analyse via :
    
    `EXPLAIN ANALYZE <requête>;`
    
- Ajout ou optimisation d’index.
    
- Analyse d’éventuels locks (`SHOW ENGINE INNODB STATUS`).
    
- Optimiser les jointures et requêtes volumineuses.

---

## **3.3 KPI 3 – Latence disque (I/O)**

**Description :**  
Mesure latence lecture/écriture sur le stockage MySQL (NAS/SSD).

**Seuils :**

- Warning : **> 15 ms**
    
- Critique : **> 20 ms**

**Actions correctives :**

- Vérifier la saturation du stockage.
    
- Déplacer les binlogs ou datafiles vers un volume plus performant.
    
- Examiner les snapshots en cours (impact I/O).
    
- Vérifier InnoDB Buffer Pool.

---

## **3.4 KPI 4 – Espace disque disponible**

**Description :**  
Surveillance de l’espace restant sur le volume contenant la base et les binlogs.

**Seuils :**

- Warning : < **20 %** libre
    
- Critique : < **10 %** libre

**Actions correctives :**

- Purge des anciens binlogs :
    
    `PURGE BINARY LOGS BEFORE NOW() - INTERVAL 48 HOUR;`
    
- Augmenter l’espace disque (NAS / Cloud).
    
- Vérifier la taille des tmpfiles et logs applicatifs.

---

## **3.5 KPI 5 – État des sauvegardes (Full + Binlogs)**

**Description :**  
Vérifie la fraîcheur et la réussite des sauvegardes.

**Seuils :**

- Warning : dernière sauvegarde > **12 h**
    
- Critique : dernière sauvegarde > **24 h**

**Actions correctives :**

- Vérifier les logs du backup.
    
- Relancer la sauvegarde manuellement.
    
- Vérifier le transfert vers NAS + Azure.
    
- Faire un test de restauration rapide (PITR).

---

# **4. Tableau récapitulatif**

|KPI|Warning|Critique|Action principale|
|---|---|---|---|
|Replication lag|> 30 sec|> 60 sec|Vérifier réplication & I/O|
|Slow queries|> 10% > 200 ms|> 25% > 200 ms|Optimisation SQL / index|
|Latence disque|> 15 ms|> 20 ms|Analyse stockage / I/O|
|Espace disque|< 20% libre|< 10% libre|Purge + extension stockage|
|État sauvegardes|> 12 h|> 24 h|Vérifier backup + PITR|

---

# **5. Schéma de supervision**

```mermaid
flowchart LR
    A["Base MySQL WMS"] --> B["performance_schema / metrics exporter"]
    B --> C["Serveur de supervision (Zabbix / Grafana)"]
    C --> D["Alerting (Mail / Teams)"]
    D --> E["Administrateur SGBD / Support N2"]
```


---

# **6. Conclusion**

La supervision MySQL définie repose sur cinq indicateurs essentiels permettant une détection rapide des problèmes et une intervention efficace.  
Elle garantit un MCO robuste du WMS, minimise les interruptions de service et assure une visibilité complète du comportement de la base de données.
