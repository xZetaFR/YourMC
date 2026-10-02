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

## Architecture technique

<details>
<summary><strong>Détails pour les développeurs</strong> (API, schéma de données, patterns)</summary>

### Plateforme YourMC — backend

```
src/
├── controllers/     auth, product, order, user
├── routes/           endpoints REST (Express Router)
├── middlewares/       auth.middleware (vérif JWT), admin.middleware, errorHandler
├── services/          appels aux services externes
├── utils/             jwt.utils, validation.utils (Zod), crypto.utils (bcrypt)
├── prisma/             schema.prisma + seed
└── server.ts           point d'entrée Express
```

`Node.js 20` · `Express 4` · `TypeScript 5` · `Prisma 5` · `jsonwebtoken` · `bcrypt` · `Zod` · `Helmet` · `cors`

### Plateforme YourMC — frontend

```
src/
├── components/ui/     Button, Input, Card, Modal, Loading (design system interne)
├── pages/               Home, Products, Login, Register, Dashboard
├── services/api.ts      client Axios + intercepteur JWT
├── store/                Zustand (authStore, productStore)
└── App.tsx               routing (React Router)
```

`React 19` · `TypeScript` · `Vite` · `Tailwind CSS 4` · `React Router 7` · `Zustand` · `Framer Motion` · `React Hook Form` + `Zod`

### Modèle de données (Prisma / MySQL, simplifié)

```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  username  String   @unique
  password  String   // bcrypt
  role      Role     @default(USER)
  licenses  License[]
  orders    Order[]
}

model Product {
  id          Int    @id @default(autoincrement())
  name        String
  price       Decimal
  category    String
  orderItems  OrderItem[]
}

model Order {
  id     Int         @id @default(autoincrement())
  userId Int
  items  OrderItem[]
  total  Decimal
  status String      // pending | completed | failed
}

model License {
  id        Int       @id @default(autoincrement())
  userId    Int
  productId Int
  key       String    @unique
  status    String    // active | revoked | expired
  expiresAt DateTime?
}
```

### Flux d'authentification

```
POST /api/auth/register  { email, username, password }
  → validation Zod (8+ car., majuscule, minuscule, chiffre)
  → bcrypt.hash(password, 10)

POST /api/auth/login  { email, password }
  → bcrypt.compare
  → jwt.sign({ userId, role }, SECRET, { expiresIn: "7d" })

Client → localStorage.setItem("token", jwt)
Axios  → intercepteur ajoute `Authorization: Bearer <token>` à chaque requête
API    → middleware auth.middleware vérifie le JWT sur les routes protégées
```

### Endpoints principaux (API REST)

| Méthode | Route | Description | Auth |
|---|---|---|---|
| `POST` | `/api/auth/register` | Création de compte | — |
| `POST` | `/api/auth/login` | Connexion, retourne un JWT | — |
| `GET` | `/api/auth/me` | Profil de l'utilisateur courant | JWT |
| `GET` | `/api/products` | Catalogue produits | — |
| `POST` | `/api/products` | Création produit | JWT + admin |
| `GET` | `/api/orders` | Commandes de l'utilisateur | JWT |

### YourBOT — architecture

```
src/
├── commands/      commandes Discord (slash commands)
├── events/         event handlers (ready, interactionCreate, guildMemberAdd…)
├── interactions/   boutons, menus, modals
├── jobs/           tâches planifiées (cron)
└── modules/        modération, tickets, réaction-rôle
```

`discord.js v14` · `TypeScript` · `Prisma` (base de données propre au bot) · communication avec la plateforme via API REST + webhooks.

### YourShop Plugin — architecture

Plugin serveur Minecraft (Paper API), écrit en Java et buildé avec Gradle. À l'achat (in-game ou via le site), le serveur interroge l'API YourMC pour valider la licence, puis livre le produit au joueur via des commandes RCON/exécution directe. Deux lignes de version maintenues en parallèle (1.21.x et 26.1) pour couvrir plusieurs versions de serveur.

### YourLauncher — architecture

Application desktop Electron. Authentification via le même JWT que la plateforme web (stocké localement), synchronisation des profils et des mods, téléchargement et vérification des fichiers client avant lancement du jeu.

### Déploiement

VPS Debian, Nginx en reverse proxy devant l'API et le frontend buildé, process Node.js gérés par PM2 (API, bots Discord), MySQL en base partagée, HTTPS via certificats TLS sur tous les sous-domaines.

</details>

---

<sub>Documentation de portfolio — le code source de chaque produit reste privé.</sub>
