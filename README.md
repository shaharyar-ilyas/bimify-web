# Bimify

Bimify is a full-stack **MERN** web application that lets architects, designers, and engineers browse, search, and preview **BIM (Building Information Modeling) objects** — 3D furniture, fixtures, and architectural components — before downloading or purchasing them. It combines a WooCommerce product catalog, Algolia-powered search, interactive in-browser 3D previews, and enterprise-grade authentication into a single storefront experience.

> Live front end: https://bimifyalgolia.netlify.app

## Overview

Manufacturers publish BIM objects (CAD/BIM model files such as `.gltf`) as WooCommerce products. Bimify's storefront sits in front of that catalog and gives end users a modern shopping experience purpose-built for BIM content: faceted search by category, brand, file type, and dimensions; an interactive 3D viewer so users can inspect a model before downloading it; ratings and reviews; and a wishlist that can be shared with a colleague. Authentication is handled by Azure AD B2C so the app can be embedded in an enterprise identity flow.

## Key Features

- **Interactive 3D model preview** — every product can be inspected in-browser as a rotatable 3D model using `three.js` / `react-three-fiber`, with `.gltf` assets streamed from Azure Blob Storage, instead of relying on static product images alone.
- **Faceted search & filtering** — Algolia InstantSearch-driven catalog with filters for category (hierarchical), brand, file type, dimensions (min/max sliders), and rating.
- **WooCommerce-backed catalog** — product data (name, description, pricing, categories, related products) is pulled live from a WooCommerce store via the official REST API, so the storefront always reflects the manufacturer's real catalog.
- **Enterprise authentication** — sign-in/sign-up/password-reset flows via Azure AD B2C (MSAL), including silent SSO support.
- **Wishlist / favorites with sharing** — signed-in users can save products to a personal cart/wishlist and share that list with another user by email.
- **Ratings & reviews** — users can rate and review individual products; the running average rating is recalculated and persisted per product.
- **Multi-language support** — UI copy is externalized and translated via `i18next`, with automatic browser language detection.
- **Analytics** — Google Tag Manager is wired up for page-view and event tracking.
- **Blog / content pages** alongside the core catalog experience.

## Tech Stack

**Client**
- React 17, React Router 6
- Redux Toolkit (global state for user session, cart, product/search state)
- `@react-three/fiber`, `@react-three/drei`, `three.js` — WebGL 3D model rendering
- `react-instantsearch` / `algoliasearch` — search & filtering UI
- `@azure/msal-browser`, `@azure/msal-react` — Azure AD B2C authentication
- Bootstrap 5, MUI (Material UI) — UI components
- `i18next` — internationalization
- Sass (SCSS modules per component)

**Server**
- Node.js + Express
- MongoDB + Mongoose — persists users, carts, wishlists, product ratings/reviews
- `@woocommerce/woocommerce-rest-api` — proxies/reads product data from the WooCommerce store
- CORS, dotenv, body-parser

**Infrastructure**
- Client deployed on **Netlify**
- Server deployed on **Heroku**
- 3D model assets served from **Azure Blob Storage**

## Architecture

```
Bimify/
├── client/                     # React SPA (Create React App)
│   └── src/
│       ├── api/                 # REST client + endpoint definitions
│       ├── components/          # Feature components (Home, Catalog, Product, Favorite, Blog, Navbar, Widget, ...)
│       ├── pages/                # Route-level pages (Home, Catalog, Product, Favorite, Blog)
│       ├── redux/                # Redux Toolkit store, slices/reducers
│       ├── authConfig.js         # Azure AD B2C / MSAL configuration
│       └── i18Next.js            # i18next setup
│
└── server/                     # Express REST API
    ├── config/                  # MongoDB connection + WooCommerce API client
    ├── controllers/             # Business logic for products & users
    ├── models/                  # Mongoose schemas (product, user)
    └── routes/                  # /api/product, /api/user route definitions
```

**Data flow:** the client renders WooCommerce product data fetched directly from the WooCommerce REST API (for browsing) and Algolia (for search), while the Express/MongoDB backend owns everything WooCommerce doesn't: user records, cart/wishlist state, wishlist sharing, and product ratings & reviews. Azure AD B2C issues the identity used to key a user's cart and reviews.

### API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/user/create` | Create a user record on first sign-in |
| POST | `/api/user/get` | Fetch a user's profile, cart, and wishlist |
| POST | `/api/user/addtocart` | Add/remove a product from a user's cart |
| POST | `/api/user/friend-list` | Share a wishlist with another user by email |
| POST | `/api/product/add-review-and-rating` | Submit a rating/review for a product |
| GET | `/api/product/get-review-and-rating` | Fetch all stored ratings/reviews |
| POST | `/api/product/get/slug` | Look up a WooCommerce product by slug |
| POST | `/api/product/get/ids` | Batch-fetch WooCommerce products by ID |

## Getting Started

### Prerequisites
- Node.js 16+
- A MongoDB instance (local or Atlas)
- WooCommerce store REST API keys
- An Azure AD B2C tenant (for authentication)
- An Algolia application/index (for search)

### Server

```bash
cd server
npm install
```

Copy `server/.env.example` to `server/.env` and fill in your own values:

```
PORT=5000
MONGODB_URL=<your MongoDB connection string>
DOMAIN=http://localhost:3000
WOOCOMMERCE_URL=<your WooCommerce store URL>
WOOCOMMERCE_CONSUMER_KEY=<your WooCommerce consumer key>
WOOCOMMERCE_CONSUMER_SECRET=<your WooCommerce consumer secret>
```

```bash
npm start
```

### Client

```bash
cd client
npm install
npm start
```

The client expects the API base URL and WooCommerce/Algolia/MSAL credentials to be configured in `client/src/api/api-routes.js` and `client/src/authConfig.js` respectively.
