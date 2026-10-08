# 🛍️ 1Fi Marketplace UI

> A new **"1Fi Marketplace"** section built inside the existing **Shop** page of a mobile fintech app. Users can browse products, pick variants, choose a **no-cost EMI** plan, review the purchase and complete the flow.

![React Native](https://img.shields.io/badge/React_Native-0.74-61DAFB?logo=react&logoColor=white)
![Expo](https://img.shields.io/badge/Expo-SDK_51-000020?logo=expo&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-5.3-3178C6?logo=typescript&logoColor=white)
![React Navigation](https://img.shields.io/badge/React_Navigation-6-6B52AE)

---

## 📑 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Screens & User Flow](#-screens--user-flow)
- [Architecture](#️-architecture)
- [Tech Stack](#️-tech-stack)
- [Project Structure](#-project-structure)
- [Data Models](#-data-models)
- [EMI Calculation](#-emi-calculation)
- [Getting Started](#️-getting-started)
- [Simulating Errors](#-simulating-errors)
- [Design Decisions](#-design-decisions)
- [Future Improvements](#-future-improvements)
- [Author](#-author)

---

## 🎯 Overview

The existing Shop screen has tabs for **Top Brands** and **Nearby Stores**. This project adds a third tab, **1Fi Marketplace**, where users can:

1. Browse a catalogue of products by category
2. Search for products
3. Open a product, pick a storage/colour variant and an EMI plan
4. Review the purchase in a confirmation sheet
5. Land on a success screen showing their EMI commitment

The UI is built against a **service layer that simulates a real backend** (network latency and optional failures), so it can be pointed at a real API later without changing screens or hooks.

---

## 🚀 Features

- 🧭 **Embedded marketplace tab** inside the Shop screen (plus a standalone route)
- 🗂️ **Category filter chips** — All, Electronics, Mobiles, Laptops, Home, Travel
- 🔍 **Search bar** with live product filtering
- 🧱 **Product grid** with price, discount and rating
- 📱 **Product details** — hero image, description, specifications, ratings
- 🎛️ **Variant selection** (storage / colour) with optional price deltas
- 💳 **EMI plan selection** — duration, monthly amount, interest rate, processing fee
- ✅ **Review modal** before confirming a purchase
- 🎉 **Success screen** after proceeding with an EMI plan
- ⏳ **Loading, error (with retry) and empty states** for every data view
- 🎨 **Centralised theme** — colours, spacing and typography tokens
- 🇮🇳 **INR formatting** using the `en-IN` locale (e.g. `₹79,900`)
- 🔒 **Strongly typed** navigation and domain models with TypeScript

---

## 🔄 Screens & User Flow

```text
Bottom Tabs:  Home │ Shop │ EMI Dues │ Limit │ Profile
                      │
                      ▼
        ┌────────────────────────────┐
        │  Shop Screen               │
        │  [Top Brands] [Nearby      │
        │   Stores] [1Fi Marketplace]│
        └─────────────┬──────────────┘
                      │  (Marketplace tab)
                      ▼
        ┌────────────────────────────┐
        │  Marketplace Screen        │
        │  categories + search + grid│
        └─────────────┬──────────────┘
                      │  tap product
                      ▼
        ┌────────────────────────────┐
        │  Product Details Screen    │
        │  variants + EMI plans      │
        └─────────────┬──────────────┘
                      │  "Continue with EMI"
                      ▼
        ┌────────────────────────────┐
        │  Review Purchase (modal)   │
        └─────────────┬──────────────┘
                      │  "Proceed"
                      ▼
        ┌────────────────────────────┐
        │  Marketplace Success Screen│
        └────────────────────────────┘
```

| Screen | Purpose |
| ------ | ------- |
| `ShopScreen` | Hosts the Top Brands / Nearby Stores / 1Fi Marketplace tabs |
| `MarketplaceScreen` | Category chips, search and product grid. Supports an `embedded` mode for use inside the Shop tab |
| `ProductDetailsScreen` | Product info, variant + EMI selection, review modal |
| `MarketplaceSuccessScreen` | Confirmation with product name, EMI duration and monthly amount |
| `HomeScreen` / `PlaceholderScreen` | Stand-ins for the rest of the app's tabs (EMI Dues, Limit, Profile) |

---

## 🏗️ Architecture

The code is layered so UI never talks to data directly:

```text
┌─────────────────────────────────────────────┐
│  Screens           (what the user sees)     │
│  ShopScreen · MarketplaceScreen · Details   │
└───────────────────────┬─────────────────────┘
                        │ uses
                        ▼
┌─────────────────────────────────────────────┐
│  Reusable Components   (marketplace UI kit) │
│  ProductCard · ProductGrid · SearchBar ...  │
└───────────────────────┬─────────────────────┘
                        │ state from
                        ▼
┌─────────────────────────────────────────────┐
│  Custom Hooks      (state + logic)          │
│  useMarketplace · useProductDetails         │
└───────────────────────┬─────────────────────┘
                        │ calls
                        ▼
┌─────────────────────────────────────────────┐
│  Service Layer     (simulated backend)      │
│  marketplaceService → async + latency       │
└───────────────────────┬─────────────────────┘
                        │ reads
                        ▼
┌─────────────────────────────────────────────┐
│  Mock Data  +  Shared Types  +  Utils       │
│  data/marketplace · types · emiCalculator   │
└─────────────────────────────────────────────┘
```

**Key rules**

- Screens and hooks **always go through the service layer** — they never import the mock arrays directly.
- Shared types live in a single module to avoid duplicated or conflicting shapes.
- Styling uses **theme tokens** (`colors`, `spacing`, `typography`) rather than hard-coded values.

---

## 🛠️ Tech Stack

| Category | Technology |
| -------- | ---------- |
| Framework | React Native `0.74` with Expo `~51` |
| Language | TypeScript `~5.3` |
| Navigation | React Navigation 6 — Native Stack + Bottom Tabs |
| UI helpers | `react-native-safe-area-context`, `react-native-screens` |
| Icons | `@expo/vector-icons` (Ionicons) |
| State | React hooks (`useState`, `useMemo`, `useCallback`) |
| Data | Mock data behind an async service layer |

---

## 📁 Project Structure

```text
1Fi-marketplace-ui/
│
├── App.tsx
├── babel.config.js
├── package.json
├── tsconfig.json
│
└── src/
    ├── navigation/
    │   ├── AppNavigator.tsx            # Root native stack
    │   ├── MainTabNavigator.tsx        # Home / Shop / EMI Dues / Limit / Profile
    │   └── types.ts                    # Typed route params
    │
    ├── screens/
    │   ├── HomeScreen.tsx
    │   ├── ShopScreen.tsx              # Hosts the 3 shop tabs
    │   ├── PlaceholderScreen.tsx
    │   └── marketplace/
    │       ├── MarketplaceScreen.tsx
    │       ├── ProductDetailsScreen.tsx
    │       └── MarketplaceSuccessScreen.tsx
    │
    ├── components/marketplace/
    │   ├── CategoryChip.tsx
    │   ├── EMIPlanCard.tsx
    │   ├── EMIPlanSelector.tsx
    │   ├── EmptyState.tsx
    │   ├── ErrorState.tsx
    │   ├── LoadingState.tsx
    │   ├── MarketplaceButton.tsx
    │   ├── MarketplaceHeader.tsx
    │   ├── MarketplaceTabs.tsx
    │   ├── PriceDisplay.tsx
    │   ├── ProductCard.tsx
    │   ├── ProductGrid.tsx
    │   ├── ProductImagePlaceholder.tsx
    │   ├── SearchBar.tsx
    │   ├── StoreCard.tsx
    │   └── VariantSelector.tsx
    │
    ├── hooks/marketplace/
    │   ├── useMarketplace.ts           # Products, search, category, loading/error
    │   └── useProductDetails.ts        # Variants, EMI plan, effective price
    │
    ├── services/
    │   └── marketplaceService.ts       # Simulated async API
    │
    ├── data/
    │   └── marketplace.ts              # Mock products, brands, stores, categories
    │
    ├── types/
    │   └── marketplace.ts              # Shared domain types
    │
    ├── theme/
    │   ├── colors.ts
    │   ├── spacing.ts
    │   └── typography.ts
    │
    └── utils/
        └── emiCalculator.ts            # EMI maths + currency formatting
```

> The `@/` import alias maps to `src/` (see [Path alias](#path-alias)).

---

## 🧩 Data Models

Defined in `src/types/marketplace.ts`:

```ts
type MarketplaceCategoryId = 'all' | 'electronics' | 'mobiles' | 'laptops' | 'home' | 'travel';
type ShopTabId = 'topBrands' | 'nearbyStores' | 'marketplace';

interface Product {
  id: string;
  name: string;
  brand: string;
  category: Exclude<MarketplaceCategoryId, 'all'>;
  image: string;
  imageColors: [string, string];
  rating?: number;
  ratingCount?: number;
  price: number;
  originalPrice?: number;
  discount?: number;
  description: string;
  variants: ProductVariant[];          // storage | color, optional priceDelta
  specifications: ProductSpecification[];
  emiPlans: EMIPlan[];
}

interface EMIPlan {
  id: string;
  durationMonths: number;
  monthlyAmount: number;
  interestRate: number;
  processingFee: number;
  totalPayable: number;
  isNoCost: boolean;
}
```

Also defined: `Brand`, `NearbyStore`, `MarketplaceCategory`, `ProductVariant`, `ProductSpecification` and `EMICalculationResult`.

### Navigation types

```ts
type RootStackParamList = {
  MainTabs: NavigatorScreenParams<MainTabParamList>;
  Marketplace: undefined;
  ProductDetails: { productId: string };
  MarketplaceSuccess: { productName: string; durationMonths: number; monthlyAmount: number };
};
```

---

## 💰 EMI Calculation

`src/utils/emiCalculator.ts` exposes two helpers:

- **`calculateEMI(principal, months, annualInterestRate = 0, processingFee = 0)`**
  - **No-cost (0%) plans:** `EMI = principal / months`, no interest charged.
  - **Interest-bearing plans:** standard reducing-balance formula

    ```text
    EMI = P × r × (1 + r)^n / ((1 + r)^n − 1)

    P = principal   r = monthly rate (annual ÷ 12 ÷ 100)   n = months
    ```

  - Returns `monthlyEMI`, `totalInterest`, `processingFee` and `totalPayable`.
  - Throws if `months <= 0`.

- **`formatCurrency(amount)`** — formats as Indian Rupees, e.g. `79900` → `₹79,900`.

---

## ⚙️ Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) 18+
- npm or yarn
- [Expo Go](https://expo.dev/go) on a phone, **or** an Android emulator / iOS simulator

### 1. Clone the repository

```bash
git clone https://github.com/shubhamk23b/1Fi-marketplace-ui.git
cd 1Fi-marketplace-ui
```

### 2. Install dependencies

```bash
npm install
```

### 3. Start the app

```bash
npm start
```

Then press `a` for Android, `i` for iOS, `w` for web — or scan the QR code with Expo Go.

### Available scripts

| Script | Description |
| ------ | ----------- |
| `npm start` | Start the Expo dev server |
| `npm run android` | Run on an Android device/emulator |
| `npm run ios` | Run on an iOS simulator |
| `npm run web` | Run in the browser |
| `npm run typecheck` | Run the TypeScript compiler (`tsc --noEmit`) |

### Path alias

The code imports with `@/` (e.g. `@/types/marketplace`). Configure it in `tsconfig.json`:

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "baseUrl": ".",
    "paths": { "@/*": ["src/*"] }
  }
}
```

Expo resolves `tsconfig` paths automatically in recent SDKs. If your setup doesn't, add `babel-plugin-module-resolver` with the same alias to `babel.config.js`.

---

## 🧪 Simulating Errors

`marketplaceService` mimics a real network: each call waits ~600 ms and can be forced to fail so you can preview the error + retry UI.

```ts
import { __setSimulateFailure } from '@/services/marketplaceService';

__setSimulateFailure(true);   // every service call now throws
__setSimulateFailure(false);  // back to normal
```

| Function | Returns |
| -------- | ------- |
| `getProducts()` | `Product[]` |
| `getProductById(id)` | `Product \| undefined` |
| `getBrands()` | `Brand[]` |
| `getNearbyStores()` | `NearbyStore[]` |
| `getCategories()` | `MarketplaceCategory[]` |

---

## 💡 Design Decisions

| Decision | Reason |
| -------- | ------ |
| **Service layer over direct imports** | UI is built exactly as it would be against a real backend; swapping in a real API only touches one file. |
| **Custom hooks for state** | Keeps screens declarative — `useMarketplace` and `useProductDetails` own filtering, selection and loading/error logic. |
| **`embedded` prop on `MarketplaceScreen`** | The same screen works inline in the Shop tab and as a standalone route, without duplicating code. |
| **Dedicated loading / error / empty components** | Every data view handles all three states consistently. |
| **Theme tokens** | One place to change colours, spacing and typography. |
| **Typed navigation** | `RootStackParamList` is declared globally, so `navigate()` calls are type-checked. |
| **Review modal before success** | Gives the user a final check of product, variant, price and EMI terms before committing. |
| **`navigation.replace` to success** | Prevents the user from navigating back into a completed purchase flow. |

---

## 🚀 Future Improvements

- [ ] Connect to a real backend API
- [ ] Wishlist / saved products
- [ ] Sorting and price/brand filters
- [ ] Real product images and gallery
- [ ] EMI calculator with adjustable tenure
- [ ] Pagination / infinite scroll
- [ ] Unit tests for `calculateEMI` and hooks
- [ ] Accessibility audit and dark mode
- [ ] Analytics and error monitoring

---

## 👨‍💻 Author

**Shubham Kanojiya**
AI/ML & Backend Developer

GitHub: [@shubhamk23b](https://github.com/shubhamk23b)

---

⭐ If you find this project useful, consider giving it a star on GitHub!
