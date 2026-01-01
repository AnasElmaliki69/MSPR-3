# PLAN DE REPRISE D’ACTIVITÉ (PRA)
## Base de Données MySQL – Projet WMS NTL / NFL IT

---

## 1. Objectifs du PRA

Le Plan de Reprise d’Activité (PRA) vise à garantir la continuité du service du système de gestion d’entrepôt (WMS) en cas d’incident majeur affectant la base de données MySQL.

Objectifs définis par la direction de NTL :

- **RTO (Recovery Time Objective)** : 1 heure  
  Délai maximal pour restaurer le service.

- **RPO (Recovery Point Objective)** : 15 minutes  
  Perte maximale de données admissible.

Le PRA couvre toutes les situations pouvant entraîner une interruption totale ou partielle des services MySQL : panne matérielle, corruption de données, erreur humaine, cyberattaque, incendie, perte d’un site d’hébergement, etc.

---

## 2. Architecture de Résilience

La stratégie de continuité repose sur trois niveaux complémentaires, en cohérence avec l’architecture de haute disponibilité mise en place.

### 2.1. Haute Disponibilité (HA) – Plan de Continuité d’Activité (PCA)

- Une instance MySQL **PRIMARY** hébergée sur une machine virtuelle Linux au datacenter de Lille  
- Une instance MySQL **REPLICA locale** hébergée sur une machine virtuelle distincte au datacenter de Lille  
- Une instance MySQL **REPLICA distante** hébergée sur une machine virtuelle Linux sur le site de Lens  
- Réplication MySQL configurée en **semi-synchronisation entre le PRIMARY et la REPLICA locale**  
- Réplication MySQL **asynchrone vers le site distant de Lens**, via le lien WAN existant  
- Surveillance du retard de réplication (replication lag)  
- Bascule manuelle ou semi-automatisée en cas de défaillance du serveur PRIMARY  

Cette organisation permet d’assurer une haute disponibilité locale tout en couvrant les scénarios de perte du site principal.

---

### 2.2. Sauvegardes locales (NAS)

- Sauvegarde complète quotidienne de la base de données réalisée avec **mysqldump**  
- Archivage régulier des **binary logs**, avec une fréquence toutes les 15 minutes  
- Stockage des sauvegardes sur un NAS dédié, isolé des serveurs MySQL  
- Politique de rétention des sauvegardes :
  - 7 sauvegardes quotidiennes  
  - 4 sauvegardes hebdomadaires  
  - 6 sauvegardes mensuelles  

Les mécanismes de sauvegarde sont indépendants de la réplication afin de garantir la restauration même en cas de corruption propagée.

---

### 2.3. Sauvegardes externalisées (Azure Storage)

- Externalisation quotidienne des sauvegardes complètes stockées sur le NAS  
- Externalisation des archives de binary logs nécessaires à la reprise fine des données  
- Sauvegardes transférées sous forme chiffrée vers **Azure Storage**  
- Stockage conforme au principe de sauvegarde **3-2-1**  
- Azure est utilisé exclusivement comme support de sauvegarde externalisé, sans hébergement de base MySQL active  

---

## 3. Scénarios de Sinistre et Procédures de Reprise

### 3.1. Scénario 1 – Panne du serveur MySQL PRIMARY

**Symptômes :**
- Perte d’accès à l’instance MySQL PRIMARY  
- Interruption ou dégradation de la réplication  
- Dégradation du service WMS  

**Procédure :**
1. Analyse rapide de la cause de la panne (matériel, système, service MySQL).  
2. Si l’incident dépasse 5 minutes, déclenchement du PRA niveau 1.  
3. Promotion de la REPLICA locale du datacenter de Lille en PRIMARY.  
4. Reconfiguration de l’application WMS pour pointer vers la nouvelle instance PRIMARY.  
5. Information du support NTL de la reprise du service.  

**RTO estimé** : 5 à 15 minutes  
**RPO estimé** : 0 à quelques secondes selon l’état de la réplication locale.

---

### 3.2. Scénario 2 – Corruption des données sur le PRIMARY

**Symptômes :**
- Erreurs SQL répétées  
- Incohérences détectées dans les données  
- Alertes de corruption InnoDB  

**Procédure :**
1. Suspension immédiate des écritures du WMS.  
2. Bascule de l’application sur la REPLICA locale si celle-ci est saine.  
3. Réalisation d’une sauvegarde d’urgence de la REPLICA.  
4. Reconstruction ou réparation du PRIMARY à partir de la REPLICA.  

**RTO estimé** : 20 minutes  
**RPO estimé** : 0 à quelques secondes.

---

### 3.3. Scénario 3 – Perte totale du datacenter de Lille (PRIMARY et NAS)

**Symptômes :**
- Perte du serveur MySQL PRIMARY  
- Perte de la REPLICA locale  
- Indisponibilité du NAS  

**Procédure :**
1. Vérification de la disponibilité de l’infrastructure du site de Lens.  
2. Promotion de la REPLICA distante de Lens en PRIMARY.  
3. Reconfiguration de l’application WMS vers cette nouvelle instance PRIMARY.  
4. Reprise des opérations applicatives.  

**RTO estimé** : 10 à 30 minutes  
**RPO estimé** : dépendant du retard de réplication WAN, compatible avec l’objectif de 15 minutes.

---

### 3.4. Scénario 4 – Perte totale des serveurs MySQL et du NAS (désastre majeur)

Dans ce scénario, seules les sauvegardes externalisées sur Azure Storage sont disponibles.

**Procédure :**
- Provisionnement d’un nouveau serveur MySQL (on-premise ou machine virtuelle Azure).  
- Téléchargement des sauvegardes complètes et des binary logs depuis Azure Storage.  
- Restauration de la dernière sauvegarde complète.  
- Relecture des binary logs jusqu’au point de reprise souhaité.  
- Reconfiguration et redémarrage de l’application WMS.  
- Validation fonctionnelle du service.  

**RTO estimé** : 30 à 60 minutes  
**RPO estimé** : inférieur ou égal à 15 minutes.

---

## 4. Rôles et Responsabilités

| Rôle | Responsabilités |
|-----|------------------|
| Responsable IT NTL | Décision d’activation du PRA, communication |
| Administrateur Systèmes | Provisionnement des serveurs, gestion VM et Azure |
| Administrateur MySQL | Restauration des données, bascule HA |
| Support WMS | Tests fonctionnels et validation du service |

---

## 5. Tests et Validation du PRA

Les tests doivent être réalisés selon le calendrier suivant :

- Trimestriellement : test de bascule PRIMARY vers REPLICA  
- Semestriellement : test de restauration depuis le NAS  
- Annuellement : test complet de restauration à partir d’Azure  

Chaque test donne lieu à un rapport comprenant :
- le temps réel de reprise  
- la conformité aux objectifs RTO/RPO  
- les éventuels écarts et mesures correctives  

---

## 6. Documentation et Outillage

Documents nécessaires :
- Guide de restauration MySQL  
- Procédures de bascule HA  
- Scripts de sauvegarde  
- Scripts de rotation et d’export des binary logs  
- Scripts de synchronisation NAS vers Azure  

---

## 7. Conclusion

La stratégie PRA proposée assure :
- La continuité du service par la réplication entre sites  
- Une restauration rapide grâce aux sauvegardes locales sur NAS  
- Une reprise après sinistre majeur grâce aux sauvegardes externalisées sur Azure  
- Le respect des exigences opérationnelles :
  - RTO inférieur ou égal à 1 heure  
  - RPO inférieur ou égal à 15 minutes  
