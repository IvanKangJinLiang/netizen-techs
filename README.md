# 🐾 Paws & Preferences

A Tinder-style web application designed for cat lovers, where users swipe right to collect their favorite kitties and build a personalized collection.

**[View Live Demo](https://ivankangjinliang.github.io/netizen-techs/)**

---

## Features

### Interactive Swiping
* **Tinder-Style Physics:** Cards follow your mouse/finger with real-time rotation and opacity adjustments.
* **Visual Feedback:** "LIKE" and "NOPE" stamps appear dynamically as you drag.
* **Multi-Input Support:**
    **Mouse:** Click and drag.
    **Touch:** Swipe on mobile devices.
    **Keyboard:** Use `Left Arrow` and `Right Arrow` keys.
    **Buttons:** Manual control buttons for accessibility.

### Performance & Polish
* **Zero-Lag Experience:** Implemented an image preloading system that fetches the next batch of cats in the background.
* **Smart History:** Prevents duplicate cats from entering your collection.
* **Persistence:** Your collection is saved to `localStorage`, so your liked cats remain even after refreshing the page.

### Responsive Design
* Fully optimized for both Desktop and Mobile views.
* Includes a "History Modal" to view your full collection without leaving the game.

---

## Tech Stack

* **Framework:** [Vue 3](https://vuejs.org/) (Composition API)
* **Build Tool:** [Vite](https://vitejs.dev/)
* **HTTP Client:** [Axios](https://axios-http.com/)
* **Gestures:** [VueUse](https://vueuse.org/) (useSwipe)
* **API:** [TheCatAPI](https://thecatapi.com/)

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