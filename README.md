# Gestion Automatisée des Demandes Étudiantes

<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/5/53/N8n-logo-new.svg" width="180" alt="n8n logo">
</p>

> Un système d'automatisation intelligent pour la gestion des demandes étudiantes — développé avec **n8n**, **Google Gemini** et la suite **Google Workspace**.

![Status](https://img.shields.io/badge/Statut-Actif-brightgreen?style=for-the-badge)
![n8n](https://img.shields.io/badge/n8n-Automatisation-orange?style=for-the-badge&logo=n8n)
![Gemini](https://img.shields.io/badge/Google-Gemini_IA-blue?style=for-the-badge&logo=google)
![License](https://img.shields.io/badge/Licence-Académique-purple?style=for-the-badge)

---

## Présentation

La gestion manuelle des demandes étudiantes entraîne des retards, des erreurs et une surcharge de travail pour l'administration. Ce projet automatise l'ensemble du cycle de vie d'une demande — de la soumission jusqu'à l'archivage — grâce à un workflow n8n intelligent alimenté par l'IA Google Gemini.

**Année universitaire :** 2025 – 2026

---

## Fonctionnalités principales

- **Centralisation des demandes** via un formulaire Google Forms
- **Validation et classification par IA** avec Google Gemini (filtrage spam, détection de catégorie et d'urgence)
- **Tableau de bord administratif** dans Google Sheets pour gérer facilement les demandes
- **Notifications e-mail automatiques** à chaque étape (reçu, répondu, traité)
- **Archivage automatique** des demandes traitées avec nettoyage du tableau de bord
- **Traitement quasi immédiat** — le workflow se déclenche toutes les 45 secondes

---

## Architecture générale du système

```
Étudiant
  └─▶ Google Forms
         └─▶ Google Sheets (Demandes)
                └─▶ Workflow n8n
                       ├─▶ Agent IA 1 (Gemini) — Validation + Classification + Résumé
                       │      ├─▶ [Non valide] → Suppression de la demande
                       │      └─▶ [Valide]     → Google Sheets (Administration) + E-mail : "Reçu"
                       │
                       └─▶ Agent IA 2 (Gemini) — Reformulation de la réponse
                              ├─▶ [Réponse présente] → E-mail avec réponse reformulée
                              └─▶ [Sans réponse]     → E-mail : "Demande traitée"
                                     └─▶ Feuille Archive + Suppression du tableau de bord
```

---

## Nodes n8n utilisés

| Node | Rôle |
|------|------|
| `Schedule Trigger` | Exécute le workflow toutes les 45 secondes |
| `Google Sheets (Read)` | Lit les nouvelles demandes depuis la feuille "Demandes" |
| `Set` | Prépare les champs (nom, e-mail, contenu) avant traitement IA |
| `AI Agent (Gemini)` | Valide, résume et classifie la demande |
| `IF` | Gère les conditions (valide/non valide, statut, réponse vide ou non) |
| `Google Sheets (Append)` | Insère dans la feuille "Administration" ou "Archive" |
| `Google Sheets (Update)` | Met à jour certaines colonnes si nécessaire |
| `Gmail` | Envoie les e-mails de notification (reçu / répondu / traité) |
| `Google Sheets (Delete)` | Supprime les lignes non valides ou archivées |

---

## Agents d'intelligence artificielle

### Agent 1 — Validation et classification
Déclenché à chaque nouvelle soumission. Produit une sortie JSON structurée :

```json
{
  "valide": true,
  "categorie": "Information | Réclamation | Urgent | Autre",
  "resume": "Résumé court et neutre de la demande",
  "statut": "Reçu | Urgent"
}
```

**Contraintes appliquées à l'agent :**
- Aucune réponse à la place de l'administration
- Aucun conseil ou solution proposée
- Sortie uniquement en format JSON

### Agent 2 — Reformulation de la réponse
Déclenché uniquement lorsque l'administration a saisi une réponse. Il reformule le message pour le rendre clair, concis et professionnel dans un ton académique, sans ajouter d'informations ni de signature.

---

## 📋 Statuts des demandes

| Statut | Signification |
|--------|--------------|
| 🟡 **Reçu** | Demande enregistrée et en attente de traitement |
| 🔴 **Urgent** | Demande prioritaire détectée par l'IA |
| 🟢 **Traité** | Demande résolue par l'administration |

---

## Notifications e-mail automatiques

Trois types d'e-mails sont envoyés automatiquement via Gmail :

1. **Confirmation de réception** — envoyée dès que la demande est ajoutée au tableau de bord
2. **Réponse finale** — envoyée quand l'administration saisit une réponse et marque la demande comme *Traité*
3. **Traitement sans réponse** — envoyée quand l'administration marque *Traité* sans réponse écrite

---

## Technologies utilisées

| Outil | Rôle |
|-------|------|
| **Google Forms** | Formulaire de soumission côté étudiant |
| **Google Sheets** | Stockage des données, tableau de bord, archive |
| **n8n** | Moteur d'automatisation du workflow |
| **Google Gemini API** *(version gratuite)* | Analyse textuelle et reformulation par IA |
| **Gmail** | Envoi automatique des e-mails de notification |

---

## Mise en place

### Prérequis
- Une instance [n8n](https://n8n.io/) (cloud ou auto-hébergée)
- Un compte Google avec accès à Forms, Sheets et Gmail
- Une clé API Google Gemini (la version gratuite suffit)

### Étapes d'installation

1. **Cloner le dépôt**
   ```bash
   git clone https://github.com/votre-username/gestion-demandes-etudiantes.git
   cd gestion-demandes-etudiantes
   ```

2. **Importer le workflow dans n8n**
   - Ouvrir votre instance n8n
   - Aller dans **Workflows → Importer depuis un fichier**
   - Sélectionner `n8n_projet.json`

3. **Configurer les identifiants dans n8n**
   - Ajouter vos identifiants **Google Sheets** (OAuth2)
   - Ajouter vos identifiants **Gmail** (OAuth2)
   - Ajouter votre **clé API Google Gemini** dans les nodes Agent IA

4. **Préparer Google Sheets**
   Créer un tableur Google avec trois feuilles :
   - `Demandes` — liée aux réponses de votre Google Forms
   - `Administration` — le tableau de bord administratif
   - `Archive` — pour les demandes traitées et archivées

5. **Mettre à jour les IDs des feuilles**
   Dans le workflow importé, mettre à jour les nodes Google Sheets avec l'ID de votre tableur et les noms corrects des feuilles.

6. **Activer le workflow**
   Basculer le workflow sur **Actif** dans n8n — il s'exécutera automatiquement toutes les 45 secondes.

---

## Structure du dépôt

```
gestion-demandes-etudiantes/
│
├── README.md
├── n8n_projet.json
│
└── docs/
    └── screenshots/
        ├── formulaire.png
        ├── workflow.png
        ├── dashboard.png
        └── archive.png
```

---

## Limitations connues

- Le workflow fonctionne en mode polling toutes les 45 secondes (Google Forms ne supporte pas nativement les webhooks en temps réel)
- La version gratuite de Google Gemini a des limites de débit — un volume élevé de soumissions peut nécessiter une limitation
- L'administration doit mettre à jour manuellement le statut à *Traité* dans Google Sheets

---

## Licence

Ce projet a été développé dans le cadre d'un projet académique. Il est destiné à des fins éducatives.

---

## Remerciements

- [Documentation n8n](https://docs.n8n.io/)
- [Google Workspace APIs](https://developers.google.com/workspace)
- [Google Gemini API](https://ai.google.dev/)
