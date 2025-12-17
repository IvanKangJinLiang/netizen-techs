# 🐾 Meow & Match

A fast, Tinder style web app built for cat lovers. Swipe through a deck of cat cards and save the ones you like to your personal collection.

👉 **Live Demo:** https://ivankangjinliang.github.io/netizen-techs/

---

## Features

### Swipe Interaction
- **Natural card movement** – cards rotate and slide smoothly as you drag, giving a real deck-of-cards feel.
- **Instant feedback** – clear **LIKE** and **NOPE** stamps appear based on swipe direction.
- **Multiple ways to interact**
  - Mouse: click and drag  
  - Touch: mobile-friendly swipe gestures  
  - Keyboard: left / right arrow keys  
  - Buttons: simple on-screen controls  

### Performance Focused
- **Image preloading** – upcoming cats are loaded in the background so swiping feels instant.
- **First-load protection** – the app waits for the first image to load before showing the card, avoiding blank states.
- **Browser caching** – image URLs are structured to allow efficient caching for faster repeat visits.
- **Rendering optimizations** – uses modern CSS features like `will-change` and `content-visibility` to keep animations smooth.
- **Persistent collection** – liked cats are saved in `localStorage`, so your collection stays even after refresh.

### Responsive & Clean UI
- **Modern design** – minimal UI with a subtle patterned background.
- **Mobile-first layout** – comfortable touch targets and spacing on smaller screens.
- **Collection modal** – view your saved cats anytime without losing your place in the stack.

---

## Tech Stack

- **Framework:** Vue 3 (Composition API)
- **Build Tool:** Vite
- **HTTP Client:** Axios
- **Gestures:** Custom swipe handling (mouse + touch)
- **API:** Cataas (Cat as a Service)

---

## Project Setup

If you want to run this project locally on your machine:

### 1. Clone the repository
```sh
git clone [https://github.com/IvanKangJinLiang/netizen-techs.git](https://github.com/IvanKangJinLiang/netizen-techs.git)
cd netizen-techs
```

## 2. Install dependencies
```bash
npm install
```
## 3. Run Development Server
```bash
npm run dev
```
## 4. Build for Production
```bash
npm run build
```

## 5. Deployment
This project is configured for GitHub Pages.
To deplopy a new version:
```bash
npm run build
npm run deploy
```