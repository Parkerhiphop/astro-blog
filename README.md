# Astro Starter Kit: Blog

Features:

- ✅ Minimal styling (make it your own!)
- ✅ 100/100 Lighthouse performance
- ✅ SEO-friendly with canonical URLs and OpenGraph data
- ✅ Sitemap support
- ✅ RSS Feed support
- ✅ Markdown & MDX support

## 🚀 Project Structure

Inside of your Astro project, you'll see the following folders and files:

```
├── public/
├── src/
│   ├── components/
│   ├── layouts/
│   └── pages/
├── astro.config.mjs
├── README.md
├── package.json
└── tsconfig.json
```

Astro looks for `.astro` or `.md` files in the `src/pages/` directory. Each page is exposed as a route based on its file name.

There's nothing special about `src/components/`, but that's where we like to put any Astro/React/Vue/Svelte/Preact components.

Any static assets, like images, can be placed in the `public/` directory.

## 🧞 Commands

All commands are run from the root of the project, from a terminal:

| Command                | Action                                           |
| :--------------------- | :----------------------------------------------- |
| `pnpm install`          | Installs dependencies                            |
| `pnpm run dev`          | Starts local dev server at `localhost:3000`      |
| `pnpm run build`        | Build your production site to `./dist/`          |
| `pnpm run preview`      | Preview your build locally, before deploying     |
| `pnpm run astro ...`    | Run CLI commands like `astro add`, `astro check` |
| `pnpm run astro --help` | Get help using the Astro CLI                     |

## 👀 Want to learn more?

Check out [our documentation](https://docs.astro.build) or jump into our [Discord server](https://astro.build/chat).

## Credit

This theme is based off of the lovely [Bear Blog](https://github.com/HermanMartinus/bearblog/).

## TODO
- Styling
  - [x] Light / Dark Mode
  - [ ] Markdown
  - [ ] Accordion
- [ ] [TOC](https://docs.astro.build/en/guides/markdown-content/#markdown-plugins)
- [ ] i18n
	- [ ] 中、英、日（當練習英、日寫作）
	- [ ] 沒翻譯的讓 Language Selector 直接不顯示該語言選項（學 MDN）
  - 看看 https://hackmd.io/@Heidi-Liu/docusaurus-react-blog
- [ ] Last Updated
- [ ] Tags
- [ ] RSS(?)
- [ ] Email Subscription
- [ ] Latest
- [ ] Pinned
- [ ] Search
- [ ] Image/Video Optimization
	- [How to Lazy Load Images in React](https://www.freecodecamp.org/news/how-to-lazy-load-images-in-react/)
- [ ] Code Content
- [ ] Emoji Reaction ( Thumb, Love, Clap )
- [ ] Comment Area ( Vote(?) )
	- Disqus
	- [GitHub Apps - utterances · GitHub](https://github.com/apps/utterances)
	- 學巴哈自架
- [ ] Open Graph & Twitter
- [ ] Sponsor
	- Buy me a coffee
	- QRCode
- [ ] GA ( for PV )
	- 至少給自己看（？
	- 不知道能不自幹出類似 Medium 那種 Stat
