# Déploiement

Ce document décrit comment le site **Les Jardiniers des Collines** est mis en ligne.

- **Site en production :** https://site-jardiniers.pages.dev/
- **Hébergeur :** Cloudflare Pages
- **Dépôt source :** GitHub — [`thomoreau/site-jardiniers`](https://github.com/thomoreau/site-jardiniers), branche `main`
- **Type de site :** site statique, **sans étape de build** (voir [CLAUDE.md](CLAUDE.md))

> ℹ️ **Migration en cours.** Le site était auparavant hébergé sur Netlify
> (`jardiniers-des-collines.netlify.app`). Cloudflare Pages est désormais
> l'hébergeur de référence. Voir la section [Netlify (ancien)](#netlify-ancien).

---

## Comment ça marche

Le site n'a **rien à compiler** : `index.html` est servi tel quel et charge
`content/site.json` au chargement de la page. Déployer revient donc simplement
à **publier les fichiers du dépôt à la racine**.

Le déclenchement est automatique à chaque `push` sur `main` :

```
Modification du contenu (2 façons)
│
├─ Via l'admin  :  /admin (Decap CMS + DecapBridge)
│                  → commit automatique sur content/site.json (branche main)
│
└─ Via le code  :  commit + push manuel sur main
│
▼
GitHub (branche main)
│
▼
Cloudflare Pages détecte le push → redéploie → site en ligne mis à jour
```

Autrement dit : **dès qu'un commit arrive sur `main`, le site se met à jour tout
seul.** L'éditeur de contenu (le propriétaire) passe par `/admin` et n'a jamais
besoin de toucher à Cloudflare.

---

## Configuration Cloudflare Pages

> ✅ **Vérifié le 15/07/2026** dans le tableau de bord Cloudflare
> (Workers & Pages → projet **site-jardiniers** → Settings). Tout est configuré
> dans l'interface web de Cloudflare — il n'y a **aucun fichier de config**
> (`wrangler.toml`, etc.) dans le dépôt.

Réglages actuels du projet Cloudflare Pages :

| Réglage | Valeur |
|---|---|
| Nom du projet | `site-jardiniers` |
| Git repository | `thomoreau/site-jardiniers` (connecté) |
| Branche de production | `main` |
| Déploiements automatiques | **Activés** |
| Build command | *(vide — rien à compiler)* |
| Build output | *(vide → la racine du dépôt est publiée)* |
| Root directory | *(vide → racine du dépôt)* |
| Build watch paths | `*` (tous les fichiers) |
| Build system version | Version 3 |
| Build cache | Désactivé |
| Deploy hooks | Aucun |
| Account ID Cloudflare | `a7858815d18d22e46872e5644be4f314` |

Points d'attention :

- **Aucune commande de build.** Si Cloudflare a détecté un framework et propose
  une commande de build, la laisser **vide** — sinon le déploiement peut échouer
  ou publier le mauvais dossier.
- Le dossier publié doit être **la racine** du dépôt, car `index.html`,
  `content/`, `images/`, `admin/` y sont tous directement.
- Le site doit être **servi en HTTP(S)** (ce qui est le cas sur Cloudflare) pour
  que le `fetch` de `content/site.json` fonctionne. Ouvrir le fichier en
  `file://` ne marche pas — c'est déjà expliqué dans [CLAUDE.md](CLAUDE.md).

### Domaine

Le site est accessible via l'URL Cloudflare par défaut :
**https://site-jardiniers.pages.dev/**

Aucun nom de domaine personnalisé n'est configuré (vérifié le 15/07/2026 :
onglet *Custom domains* vide). Pour en ajouter un plus tard :
*Cloudflare Pages → le projet → Custom domains → Set up a custom domain*.

---

## Publier une mise à jour

### Cas 1 — Modifier le contenu (le plus fréquent)

Passer par l'espace d'administration :

1. Aller sur **https://site-jardiniers.pages.dev/admin/**
2. Se connecter (authentification DecapBridge).
3. Modifier les textes / photos / tarifs, puis **Enregistrer / Publier**.
4. Decap enregistre un commit sur `content/site.json` (branche `main`).
5. Cloudflare Pages redéploie automatiquement. Le site est à jour en ~1 minute.

### Cas 2 — Modifier le code du site

Pour toute modification de `index.html`, de la config admin, etc. (commandes
PowerShell) :

```powershell
git add .
git commit -m "Décrire la modification"
git push origin main
```

Le `push` sur `main` déclenche automatiquement un nouveau déploiement Cloudflare.

### Suivre / dépanner un déploiement

- Voir l'état des déploiements : **Cloudflare Dashboard → Workers & Pages → le
  projet → onglet Deployments** (logs, succès/échec, possibilité de *rollback*
  vers un déploiement précédent).
- Si le site ne se met pas à jour après un commit : vérifier que le commit est
  bien sur la branche `main` et que le dernier déploiement Cloudflare est en
  succès (pas en échec de build).

---

## Netlify (ancien)

Avant la migration, le site était déployé sur Netlify
(`jardiniers-des-collines.netlify.app`), également connecté au dépôt GitHub.

Tâches de nettoyage :

- [ ] **À faire côté interface Netlify** — débrancher ou supprimer le projet
      Netlify pour éviter les déploiements en double sur chaque commit.
- [x] Mettre à jour les URL qui pointaient vers Netlify (fait le 15/07/2026) :
  - `admin/config.yml` → `site_url` pointe désormais sur `site-jardiniers.pages.dev`
  - `CLAUDE.md` → hébergeur mis à jour (Cloudflare Pages, Netlify noté comme ancien)

> Tant que le projet Netlify n'est pas débranché, chaque commit sur `main`
> peut continuer à déclencher un déploiement Netlify en parallèle de Cloudflare.
