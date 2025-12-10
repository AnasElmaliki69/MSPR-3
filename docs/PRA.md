
---

# PLAN DE REPRISE D’ACTIVITÉ (PRA)

Base de Données MySQL – Projet WMS NTL / NFL IT

---

# 1. Objectifs du PRA

Le Plan de Reprise d’Activité vise à garantir la continuité du service du système de gestion d’entrepôt (WMS) en cas d’incident majeur affectant la base MySQL.

Objectifs définis par la direction de NTL :

* RTO (Recovery Time Objective) : 1 heure
  Délai maximal pour restaurer le service.

* RPO (Recovery Point Objective) : 15 minutes
  Perte maximale de données admissible.

Le PRA couvre toutes les situations pouvant entraîner une interruption totale ou partielle des services MySQL : panne matérielle, corruption de données, erreur humaine, cyberattaque, incendie, perte du datacenter, etc.

---

# 2. Architecture de Résilience

La stratégie de continuité repose sur trois niveaux.

## 2.1. Haute Disponibilité (HA) – Plan de Continuité d’Activité (PCA)

* Serveur MySQL PRIMARY hébergé au datacenter de Lille
* Serveur MySQL REPLICA hébergé au datacenter de Lens
* Réplication MySQL asynchrone
* Surveillance du retard de réplication (replication lag)
* Commutation manuelle ou semi-automatisée en cas de défaillance du serveur primaire

## 2.2. Sauvegardes locales (NAS)

* Sauvegarde complète quotidienne réalisée avec mysqldump ou Percona XtraBackup
* Sauvegarde des binlogs toutes les 15 minutes
* Stockage sur un NAS isolé du serveur MySQL
* Rétention : 7 jours, 4 hebdomadaires, 6 mensuelles

## 2.3. Sauvegardes externalisées (Azure Blob Storage)

* Copie chiffrée quotidienne du backup complet
* Copie chiffrée des binlogs
* Stockage conforme au modèle 3-2-1
* Conservation dans un environnement géographiquement distinct

---

# 3. Scénarios de Sinistre et Procédures de Reprise

## 3.1. Scénario 1 – Panne du serveur PRIMARY

Symptômes :

* Perte d’accès MySQL
* Réplication interrompue
* Dégradation du service WMS

Procédure :

1. Analyse rapide de la cause (matériel, système, MySQL).
2. Si la panne dépasse 5 minutes, déclencher le PRA niveau 1.
3. Promotion du REPLICA en PRIMARY :

   ```
   STOP SLAVE;
   RESET SLAVE ALL;
   ```
4. Reconfigurer l’application WMS pour pointer vers le nouveau PRIMARY.
5. Informer le support NTL de la reprise du service.

RTO estimé : 5 à 15 minutes
RPO estimé : 0 à quelques secondes selon la réplication

---

## 3.2. Scénario 2 – Corruption des données sur le PRIMARY

Symptômes :

* Erreurs SQL
* Incohérences détectées dans les données
* Alertes de corruption InnoDB

Procédure :

1. Interrompre immédiatement les écritures du WMS.
2. Basculer l’application sur le REPLICA si celui-ci est sain.
3. Lancer une sauvegarde d’urgence du REPLICA.
4. Reconstruire ou réparer le PRIMARY à partir du REPLICA.

RTO estimé : 20 minutes
RPO estimé : 0 à quelques secondes

---

## 3.3. Scénario 3 – Perte totale du datacenter de Lille (PRIMARY et NAS)

Symptômes :

* Perte totale du PRIMARY
* NAS indisponible
* Infrastructure principale hors service

Procédure :

1. Vérifier la disponibilité du datacenter de Lens.
2. Promouvoir le REPLICA en PRIMARY.
3. Restaurer si nécessaire la sauvegarde locale secondaire.
4. Relancer le WMS sur ce nouveau PRIMARY.

RTO estimé : 10 à 30 minutes
RPO estimé : 0 à quelques secondes (selon le retard de réplication)

---

## 3.4. Scénario 4 – Perte totale des deux serveurs MySQL et du NAS (désastre majeur)

Seules les sauvegardes Azure restent disponibles.

Procédure :

Étape 1 – Provisionner un nouveau serveur MySQL

* Soit dans un nouveau datacenter on-premise
* Soit dans une machine virtuelle Azure

Étape 2 – Télécharger les sauvegardes Azure
Avec azcopy :

```
azcopy cp "https://<container>/full_backup.sql.gz" /tmp/
azcopy cp "https://<container>/binlogs/" /tmp/binlogs/ --recursive
```

Étape 3 – Restaurer la sauvegarde complète

```
gunzip < full_backup.sql.gz | mysql
```

Étape 4 – Rejouer les binlogs jusqu’au point de reprise souhaité

```
mysqlbinlog binlog.000123 binlog.000124 | mysql
```

Étape 5 – Redémarrer l’application WMS sur ce nouveau serveur

* Mise à jour de la configuration
* Tests fonctionnels
* Validation du service

RTO estimé : 30 à 60 minutes
RPO estimé : inférieur ou égal à 15 minutes

---

# 4. Rôles et Responsabilités

| Rôle                    | Responsabilités                                |
| ----------------------- | ---------------------------------------------- |
| Responsable IT NTL      | Décision d’activation du PRA, communication    |
| Administrateur Systèmes | Provisionnement des serveurs, gestion VM/Azure |
| Administrateur MySQL    | Restauration des données, bascule HA           |
| Support WMS             | Tests fonctionnels et validation du service    |

---

# 5. Tests et Validation du PRA

Les tests doivent être réalisés selon le calendrier suivant :

* Trimestriellement : test de bascule PRIMARY vers REPLICA
* Semestriellement : test de restauration depuis le NAS
* Annuellement : test complet de restauration à partir d’Azure

Chaque test doit donner lieu à un rapport comprenant :

* le temps réel de reprise
* la conformité aux objectifs RTO/RPO
* les éventuels écarts et mesures correctives

---

# 6. Documentation et Outillage

Documents nécessaires :

* Guide de restauration MySQL
* Procédures de bascule HA
* Scripts de sauvegarde
* Scripts de rotation et export des binlogs
* Scripts de synchronisation NAS vers Azure

---

# 7. Conclusion

La stratégie PRA proposée assure :

* La continuité du service par la réplication entre datacenters
* Une restauration rapide grâce aux sauvegardes locales sur NAS
* Une garantie de reprise après sinistre majeur grâce aux sauvegardes externalisées Azure
* Le respect des exigences opérationnelles :

  * RTO inférieur ou égal à 1 heure
  * RPO inférieur ou égal à 15 minutes

