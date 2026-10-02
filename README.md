<div align="center">

# YourMC

**Écosystème complet de solutions Minecraft** — plateforme e-commerce, bots Discord, CMS, launcher et plugin de boutique, conçus pour fonctionner ensemble.

![Status](https://img.shields.io/badge/status-production-success)
![Products](https://img.shields.io/badge/products-8-blue)
![License](https://img.shields.io/badge/license-proprietary-lightgrey)

</div>

---

> **À propos de ce dépôt.** YourMC est un projet complet et propriétaire ; le code source n'est pas publié ici. Ce dépôt documente l'architecture, les choix techniques et le fonctionnement de l'écosystème, à des fins de portfolio.

## Vue d'ensemble

YourMC centralise la gestion d'un serveur Minecraft orienté monétisation : vente de rangs/produits, administration communautaire sur Discord, site de présentation personnalisable, launcher de jeu maison et boutique in-game. Huit produits, une authentification commune, une base de données partagée.

```
                         ┌─────────────────────────┐
                         │   Plateforme YourMC      │
                         │   (API + Frontend web)   │
                         └────────────┬─────────────┘
                                      │
                 ┌────────────────────┼────────────────────┐
                 │                    │                     │
        ┌────────▼────────┐  ┌────────▼────────┐  ┌────────▼────────┐
        │  Minecraft       │  │  Discord         │  │  Web             │
        │  YourLauncher    │  │  YourBOT         │  │  YourCMS         │
        │  YourShop Plugin │  │  YourWhiteList   │  │  YourConfig      │
        │                  │  │  YourShop Bot    │  │                  │
        └──────────────────┘  └──────────────────┘  └──────────────────┘
                 │                    │                     │
                 └────────────────────┼─────────────────────┘
                                      │
                           ┌──────────▼──────────┐
                           │   Base de données     │
                           │   (utilisateurs,      │
                           │   licences, produits)  │
                           └───────────────────────┘
```

## Les 8 produits

| Produit | Rôle | Stack |
|---|---|---|
| **Plateforme YourMC** | API centrale, authentification, catalogue, compte utilisateur | Node.js · Express · TypeScript · React · MySQL |
| **YourBOT** | Bot Discord principal : modération, tickets, réaction-rôle, engagement | discord.js · TypeScript · Prisma |
| **YourCMS** | CMS avec éditeur de pages en glisser-déposer et thèmes | PHP 8 · MySQL · JS |
| **YourLauncher** | Launcher Minecraft desktop : profils, mods, compte synchronisé | Electron · JavaScript |
| **YourShop Plugin** | Plugin serveur Minecraft pour la boutique in-game | Java · Gradle · Paper API |
| **YourWhiteList Bot** | Automatise les candidatures de whitelist avec vérification Mojang | discord.js · TypeScript |
| **YourShop Bot** | Relie un achat sur la boutique à l'attribution d'un rôle Discord | discord.js · TypeScript |
| **YourConfig** | Interface web pour configurer et tester des plugins serveur | React · TypeScript · Docker |

## Comment les produits s'articulent

- **Authentification unique** : un compte YourMC (JWT) sert pour le site, le launcher et les bots.
- **Base de données partagée** : utilisateurs, licences et commandes sont lus et écrits par plusieurs produits.
- **Attribution automatique** : un achat sur YourShop (in-game ou Discord) déclenche l'octroi du produit via RCON côté serveur, et du rôle correspondant via YourShop Bot côté Discord.
- **Webhooks** : la plateforme notifie les bots Discord des événements (nouvelle commande, nouvelle licence, etc.).

## Trois parcours types

**Un joueur** visite le site, achète YourLauncher, reçoit sa clé d'activation et lance le jeu avec ses mods depuis le launcher.

**Un administrateur de serveur** installe YourShop Plugin, configure sa boutique depuis le dashboard YourMC, et ses joueurs achètent directement en jeu via `/shop` — produit livré automatiquement.

**Un gestionnaire de communauté Discord** déploie YourBOT (modération), YourWhiteList Bot (candidatures) et YourShop Bot (rôles après achat) pour administrer sa communauté sans intervention manuelle.

## Sécurité

Authentification JWT (expiration 7 jours), mots de passe hashés en bcrypt, requêtes préparées contre les injections SQL, échappement systématique contre le XSS, CORS restreint aux domaines autorisés, rate limiting sur les endpoints sensibles, HTTPS partout.

## Stack globale

`Node.js` `Express` `TypeScript` `React` `MySQL` `Prisma` `Electron` `Java / Paper API` `PHP 8` `discord.js` `Docker` `Nginx` `PM2`

---

<sub>Documentation de portfolio — le code source de chaque produit reste privé.</sub>
