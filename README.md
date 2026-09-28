# Pizzeria Studio

[![CI](https://github.com/MykolaDotsenko/pizzeria-studio/actions/workflows/ci.yml/badge.svg)](https://github.com/MykolaDotsenko/pizzeria-studio/actions/workflows/ci.yml)
![React 19](https://img.shields.io/badge/React-19.3-149ECA?logo=react&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-strict-3178C6?logo=typescript&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)

**A React + TypeScript menu editor where the interesting work is persistence, money, image handling and recovery rather than the CRUD itself.**

[**Open the live app →**](https://pizzeria-studio.vercel.app/) · [Architecture](./ARCHITECTURE.md)

![Pizzeria Studio desktop interface](./docs/screenshots/pizzeria-desktop.png)

## What it does

- add and edit pizzas with name, description, category, price and photo;
- search/filter the menu;
- reorder items by drag-and-drop or explicit move controls;
- delete with Undo;
- keep edits, order and uploaded images across reloads;
- open dedicated pizza detail routes;
- migrate older stored data safely.

## Engineering details

### Exact money

Prices enter domain state as integer cents:

```text
€12.90 → 1290
```

Floating-point values are not the authoritative money representation.

### Persisted data is validated

Browser storage uses a versioned envelope. The current code understands the original unversioned format plus versions 1–3.

Corrupt records are rejected/isolated where possible. If a newer unknown schema is found, the application refuses to overwrite it with an older format.

### Images are processed before storage

A selected image is not written directly to `localStorage`.

The browser boundary:

1. validates file type and source size;
2. decodes the image;
3. preserves aspect ratio;
4. resizes within bounded dimensions;
5. converts/compresses to JPEG;
6. reduces output until it fits the storage budget.

### State updates use one authoritative snapshot

Reducer transitions own create/edit/reorder/delete/Undo behaviour. Same-event mutations are composed from the same state snapshot so persistence does not lag behind the UI because of stale React state.

### Reordering is not drag-only

Mouse users can drag cards; keyboard/touch users also have explicit move controls.

## Architecture

```text
React UI
   ↓
provider / dispatch
   ↓
pure reducer + pizza rules
   ↓
persistence adapter

browser image APIs
   ↓
image-processing boundary
   ↓
validated image value
```

There is no Redux/Zustand layer or backend because the current single-device scope does not require synchronization between users.

## Stack

- React 19
- React Router 8
- TypeScript 6 strict
- Vite 8
- Zod 4
- Web Storage
- Canvas/image browser APIs
- Vitest + React Testing Library
- Playwright
- ESLint + Prettier
- GitHub Actions
- Vercel

## Browser coverage

The production build is exercised in Chromium, Firefox, WebKit and mobile Chromium.

Scenarios include:

- create → detail → search;
- edit + reload persistence;
- category filtering;
- accessible and drag-and-drop reorder;
- delete + Undo;
- validation failures;
- rejected image formats;
- optimized image persistence;
- missing routes;
- unknown future storage schemas.

Screenshots in this repository are generated from the running production build.

## Quality

```bash
npm ci
npm run check
npx playwright install chromium firefox webkit
npm run test:e2e
```

The static gate covers formatting, linting, type checking, tests, production build and dependency audit.

## Run locally

```bash
npm ci
npm run dev
```

## Deployment

Canonical deployment:

**https://pizzeria-studio.vercel.app/**

The app uses `BrowserRouter`, so `vercel.json` includes an SPA rewrite to keep direct detail URLs working after refresh.

## Scope

Pizzeria Studio is a single-device menu editor, not a restaurant POS or multi-user SaaS.

Accounts, shared editing, payments or multi-device synchronization would justify server storage and object storage for images; those are intentionally outside the current product.

## License

MIT © Mykola Dotsenko
