# Documentation Maintenance Guide

This folder contains the Docusaurus source code for the LinqUp documentation site. It is linked to the [https://github.com/LinqUp-ng/documentation](https://github.com/LinqUp-ng/documentation) repository.

---

## 🚀 How to Update Documentation

### 1. Add or Edit Content
*   All documentation pages live in the `docs/docs/` directory.
*   You can create new `.md` or `.mdx` files here.
*   **Avoid standard HTML comments** `<!-- -->`; use MDX comments `{/* */}` instead.

### 2. Update the Sidebar (Navigation)
If you want to change the order of the documentation in the sidebar or group them into categories, edit:
👉 `docs/sidebars.ts`

### 3. Update the Top Navbar
To add new links to the top navigation bar or change the site title, edit:
👉 `docs/docusaurus.config.ts`

### 4. Deploy Changes
Since the `docs` folder is a separate Git repository, you need to push changes from **within** that folder:

```bash
cd docs
git add .
git commit -m "docs: describe your changes here"
git push origin main
```

Once you push, the **GitHub Action** will automatically build the site and deploy the updates to the live URL within 2 minutes.

---

## 🛠️ Local Preview
To see your changes on your own computer before pushing:

1. Open your terminal in the `docs` folder.
2. Run: `npm start`
3. Open `http://localhost:3000` in your browser.
