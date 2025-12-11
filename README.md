# MSPR 3 – Conception et protection d'une base de données

## Contexte

Projet réalisé dans le cadre du Bachelor CYB3R XP (EPSI / WIS).

Client : **NFL IT**, pour la PME **NordTransit Logistics (NTL)**.  
Objectif : concevoir et sécuriser la nouvelle base de données du WMS (Warehouse Management System), 
assurer sa haute disponibilité, sa sauvegarde et sa supervision.

RTO cible : **1 heure**  
RPO cible : **15 minutes**

---

## Répartition des rôles

- **Maryama** : Modélisation des données  
  - MCD
  - MLD

- **Anas** : Architecture & SGBD  
  - Benchmark comparatif (MySQL / PostgreSQL / MariaDB → choix final : MySQL)
  - Architecture HA MySQL (Primary Lille → Replica Lens)
  - Conception du PRA (NAS + Azure Blob Storage)
  - Documentation d’architecture globale

- **Kerim** : Sécurité, sauvegardes, supervision  
  - Politiques de sécurité SQL (rôles, comptes, GRANT – moindre privilège)
  - Stratégies de sauvegarde (binlogs + full backups – conformes RPO/RTO)
  - Indicateurs de supervision (KPI) et procédures de remédiation

---

## Livrables principaux

- Document d’architecture technique (MCD, MLD, SGBD, HA/PRA, sécurité, sauvegardes, supervision)
- Plan de reprise d’activité MySQL (PRA)
- Guide de supervision (indicateurs, seuils, remédiation)
- Support de soutenance (présentation)
