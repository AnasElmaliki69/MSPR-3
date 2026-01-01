
# **Politique de Sécurité SQL**

## **1. Introduction**

La sécurité de la base WMS est essentielle pour garantir la continuité des opérations logistiques de l’entreprise.  
Cette politique définit l’ensemble des règles d’accès permettant d’assurer la confidentialité, l’intégrité et la disponibilité des données.  
Elle s’appuie sur le principe du moindre privilège, une séparation stricte des rôles et un contrôle rigoureux des actions effectuées sur la base.

---

## **2. Objectifs de la politique de sécurité**

- Protéger les données critiques du WMS.
    
- Limiter les accès aux seules opérations nécessaires.
    
- Empêcher toute élévation de privilège non autorisée.
    
- Garantir une traçabilité complète des actions SQL.
    
- Réduire le risque d’erreur humaine ou malveillance.
    
- Séparer les rôles applicatifs, techniques et administratifs.

---

## **3. Définition des rôles SQL**

### **3.1 Administrateur SGBD – `dba_admin`**

- Droits complets sur la base WMS.
    
- Création et gestion des rôles.
    
- Gestion des index, tables, schémas.
    
- Supervision des sauvegardes et restaurations.
    
- Pas d'utilisation applicative.

### **3.2 Compte applicatif – `wms_app`**

- Utilisé uniquement par l'application WMS.
    
- Droits autorisés :
    
    - `SELECT`, `INSERT`, `UPDATE`, `DELETE`
        
- Restrictions :
    
    - Aucun droit `ALTER`, `DROP`, `CREATE`, `TRUNCATE`.
        
    - Accès uniquement aux tables nécessaires.

### **3.3 Compte lecture seule – `read_only`**

- Utilisé par le support ou l’audit.
    
- Droits autorisés :
    
    - `SELECT` uniquement.
        
- Aucun droit de modification.

### **3.4 Comptes techniques**

- **`backup_user`** : exécute les sauvegardes, accès au système de fichiers dédié.
    
- **`monitor_user`** : lecture des métriques internes, accès aux vues performance_schema.
    
- Les mots de passe sont gérés via un Secret Manager.
    
- Aucun accès aux données fonctionnelles.

---

## **4. Règles du moindre privilège**

- Aucun compte ne possède de droits supérieurs à son besoin fonctionnel.
    
- Aucun compte partagé n'est autorisé.
    
- Les rôles admin, applicatifs et techniques doivent rester séparés.
    
- Les mots de passe sont régénérés régulièrement.
    
- Les droits globaux (comme SUPERUSER) sont interdits.
    
- Activation des logs d’audit pour toutes les actions sensibles.
    
- Journalisation obligatoire des connexions et erreurs.

---

## **5. Exemples de GRANT (MySQL)**

```-- Compte applicatif WMS
CREATE USER 'wms_app'@'%' IDENTIFIED BY 'xxxx';
GRANT SELECT, INSERT, UPDATE, DELETE ON wms.* TO 'wms_app'@'%';

-- Compte lecture seule
CREATE USER 'read_only'@'%' IDENTIFIED BY 'xxxx';
GRANT SELECT ON wms.* TO 'read_only'@'%';

-- Compte de sauvegarde (dumps + binlogs)
CREATE USER 'backup_user'@'%' IDENTIFIED BY 'xxxx';
GRANT RELOAD, LOCK TABLES, REPLICATION CLIENT, REPLICATION SLAVE ON *.* TO 'backup_user'@'%';

-- Compte de supervision
CREATE USER 'monitor_user'@'%' IDENTIFIED BY 'xxxx';
GRANT PROCESS, REPLICATION CLIENT, SELECT ON performance_schema.* TO 'monitor_user'@'%';
```

---

## **6. Tableau récapitulatif des rôles**

| Rôle             | Permissions                                   | Usage           | Restrictions                  |
| ---------------- | --------------------------------------------- | --------------- | ----------------------------- |
| **dba_admin**    | Tous droits                                   | Administration  | Non utilisé par l'application |
| **wms_app**      | SELECT, INSERT, UPDATE, DELETE                | Application WMS | Aucun droit structurel        |
| **read_only**    | SELECT                                        | Support / Audit | Lecture seule                 |
| **backup_user**  | RELOAD, LOCK TABLES, REPLICATION CLIENT/SLAVE | Backups         | Aucun accès applicatif        |
| **monitor_user** | SELECT ON performance_schema.                 | Supervision     | Lecture uniquement            |

---

## **7. Conclusion**

La politique de sécurité SQL définie pour la base WMS garantit une séparation stricte des responsabilités, une limitation fine des privilèges et une protection renforcée des données critiques.  
Elle constitue un socle essentiel pour la continuité de service et l’exploitabilité de la base au quotidien.
