# 04 - Sécurité, Sauvegardes et Supervision

---

## 1. Politique de sécurité SQL

### 1.1 Objectifs

La politique de sécurité vise à :
- Limiter les accès à la base WMS.
- Appliquer le principe du moindre privilège.
- Protéger les données critiques.
- Garantir la traçabilité des actions.

---

## 1.2 Rôles SQL

### Administrateur SGBD (DBA)
- Gestion complète de la base.
- Peut créer des rôles, appliquer des GRANT et gérer les index.
- Responsable des sauvegardes/restaurations.

### Compte applicatif WMS (wms_app)
- Aucun droit d'administration.
- Accès uniquement aux tables nécessaires.
- Opérations autorisées : SELECT, INSERT, UPDATE, DELETE.

### Compte lecture seule (read_only)
- Utilisé par le support.
- Droit : SELECT uniquement.

### Comptes techniques
- backup_user : sauvegardes.
- monitor_user : supervision.
- Stockés dans un secret manager.

---

## 1.3 Règles du moindre privilège
- Aucun compte partagé.
- Aucun droit global (SUPER, SUPERUSER).
- Permissions strictement limitées au besoin.
- Rotation de mots de passe.
- Audit log activé.

---

## 1.4 Exemple GRANT (PostgreSQL)

```sql
CREATE ROLE wms_app LOGIN PASSWORD 'xxxx';
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO wms_app;

CREATE ROLE read_only LOGIN PASSWORD 'xxxx';
GRANT SELECT ON ALL TABLES IN SCHEMA public TO read_only;

CREATE ROLE monitor_user LOGIN PASSWORD 'xxxx';
GRANT pg_monitor TO monitor_user;
```

---

## 2. Stratégies de sauvegarde (RPO 15 min / RTO 1h)

### 2.1 Objectifs
- RPO = 15 minutes.
- RTO = 1 heure.

---

## 2.2 Politique de sauvegardes

### Sauvegarde complète
- Une fois par jour (02h00).

### Sauvegardes incrémentales
- Toutes les 15 minutes via WAL/binlog.

### Point-In-Time Recovery (PITR)
- Permet restauration à n'importe quelle minute.

---

## 2.3 Stockage

### Local
- NAS avec RAID + snapshots.

### Externalisé
- Vers site secondaire ou cloud.
- Chiffrement AES-256.

### Immutabilité
- Snapshots WORM.

---

## 2.4 Rétention
- Full backup : 14 jours.
- Incrémentales : 7 jours.
- Journaux WAL/Binlog : 48h.

---

## 2.5 Tests
- Test de restauration mensuel.
- Vérification d'intégrité.
- Simulation perte complète : restauration < 1 heure.

---

## 3. Supervision

### 3.1 Indicateurs critiques

### Replication lag
- Alerte > 60 sec.

### Slow queries
- Alerte > 10% au-dessus de 200 ms.

### Latence disque
- Alerte > 20 ms.

### Espace disque
- Alerte > 90%.

### Sauvegardes
- Alerte si > 24h.

---

## 3.2 Remédiations

### Replication lag
- Vérifier charge / réseau.
- Re-synchroniser réplica.

### Slow queries
- Analyse EXPLAIN.
- Ajout index.
- Optimisation requêtes.

### Espace disque
- Purge logs.
- Augmenter stockage.

### Sauvegardes en échec
- Lire logs.
- Relancer backup.
- Test restauration.
