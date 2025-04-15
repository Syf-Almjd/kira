
# kira_tech

## 🚀 Project Setup

```bash
git clone https://github.com/Syf-Almjd/kira.git
cd kira
npm install
```

## 💻 Compiles and Hot-Reloads for Development

```bash
npm run serve
```

Access the app at: [http://localhost:8080](http://localhost:8080)

## 📦 Compiles and Minifies for Production

```bash
npm run build
```

The build files will be located in the `dist/` folder.

## 🔍 Lints and Fixes Files

```bash
npm run lint
```


## ⚙️ Customize Configuration

See [Vue CLI Configuration Reference](https://cli.vuejs.org/config/)

## 🧰 Useful Commands

### Clean Cache (if issues arise)

```bash
rm -rf node_modules package-lock.json
npm cache clean --force
npm install
```

### Preview Production Build Locally

```bash
npm install -g serve
serve -s dist
```
