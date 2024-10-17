# **How to Set Up a Multilingual Decap CMS with GitLab**

Managing multilingual content can be a complex task, but with Decap CMS and GitLab, you can streamline the process effectively. This guide will walk you through setting up a multilingual content management system using Decap CMS with GitLab, handling slug fields, adding featured images, and linking translations.

## **Table of Contents**

1. [Prerequisites](#prerequisites)
2. [Setting Up Decap CMS with GitLab](#setting-up-decap-cms-with-gitlab)
3. [Configuring Multilingual Collections](#configuring-multilingual-collections)
4. [Switching to String Widget for Slug Fields](#switching-to-string-widget-for-slug-fields)
5. [Adding Featured Images to Posts](#adding-featured-images-to-posts)
6. [Linking Translations with Relation Fields](#linking-translations-with-relation-fields)
7. [Troubleshooting Common Issues](#troubleshooting-common-issues)
8. [Conclusion](#conclusion)

---

## **Prerequisites**

- A self-hosted GitLab instance.
- A static site generator (e.g., Hugo, Jekyll) configured with multilingual support.
- Basic knowledge of YAML and Git.
- Decap CMS installed in your project.

---

## **Setting Up Decap CMS with GitLab**

### **Step 1: Configure OAuth Application in GitLab**

1. **Navigate to User Settings:**
   - Go to `https://your-gitlab-instance/`.
   - Click on your avatar and select **"Preferences"** or **"Settings"**.

2. **Create a New OAuth Application:**
   - Go to **"Applications"** in the sidebar.
   - Click **"New Application"**.
   - **Name:** Enter a name like "Decap CMS".
   - **Redirect URI:** `https://your-website.com/admin/`.
   - **Scopes:** Select `read_user`, `api`, `read_repository`, `write_repository`.
   - **Confidential:** Uncheck the **"Confidential"** option.
   - Click **"Save application"**.

3. **Note the Application ID:**
   - After saving, copy the **Application ID**.

### **Step 2: Configure `config.yml` for Decap CMS**

Create or update `admin/config.yml` with the following:

```yaml
backend:
  name: gitlab
  repo: yourusername/yourrepository
  branch: main
  auth_type: pkce
  app_id: your_application_id
  api_root: https://your-gitlab-instance/api/v4
  base_url: https://your-gitlab-instance
  auth_endpoint: oauth/authorize
  scopes: 'read_user api read_repository write_repository'
media_folder: "static/uploads"
public_folder: "/uploads"
```

**Notes:**

- Replace `yourusername/yourrepository` with your GitLab repository path.
- Ensure the `api_root` and `base_url` match your GitLab instance URL.

### **Step 3: Add Admin Interface**

Create an `admin` folder in your project root and add an `index.html` file with the following content:

```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Content Manager</title>
  </head>
  <body>
    <!-- Include the Decap CMS script -->
    <script src="https://unpkg.com/decap-cms@latest/dist/decap-cms.js"></script>
    <!-- Initialize the CMS -->
    <script>
      CMS.init();
    </script>
  </body>
</html>
```

### **Step 4: Commit and Push Changes**

```bash
git add admin/index.html admin/config.yml
git commit -m "Set up Decap CMS with GitLab"
git push
```

---

## **Configuring Multilingual Collections**

Assuming your content is organized into language-specific directories:

```
content/
├── de/
│   ├── academics/
│   └── posts/
└── en/
    ├── academics/
    └── posts/
```

### **Step 1: Define Collections in `config.yml`**

Add separate collections for each language in your `config.yml`:

```yaml
collections:
  # German Collections
  - name: "academics_de"
    label: "Academics (German)"
    folder: "content/de/academics"
    create: true
    slug: "{{slug}}"
    fields:
      - { label: "Titel", name: "title", widget: "string" }
      - { label: "Slug", name: "slug", widget: "string", required: true }
      - { label: "Inhalt", name: "body", widget: "markdown" }
  - name: "posts_de"
    label: "Posts (German)"
    folder: "content/de/posts"
    create: true
    slug: "{{slug}}"
    fields:
      - { label: "Titel", name: "title", widget: "string" }
      - { label: "Slug", name: "slug", widget: "string", required: true }
      - { label: "Datum", name: "date", widget: "datetime" }
      - { label: "Inhalt", name: "body", widget: "markdown" }
  # English Collections
  - name: "academics_en"
    label: "Academics (English)"
    folder: "content/en/academics"
    create: true
    slug: "{{slug}}"
    fields:
      - { label: "Title", name: "title", widget: "string" }
      - { label: "Slug", name: "slug", widget: "string", required: true }
      - { label: "Body", name: "body", widget: "markdown" }
  - name: "posts_en"
    label: "Posts (English)"
    folder: "content/en/posts"
    create: true
    slug: "{{slug}}"
    fields:
      - { label: "Title", name: "title", widget: "string" }
      - { label: "Slug", name: "slug", widget: "string", required: true }
      - { label: "Date", name: "date", widget: "datetime" }
      - { label: "Body", name: "body", widget: "markdown" }
```

### **Step 2: Commit and Push Changes**

```bash
git add config.yml
git commit -m "Configure multilingual collections"
git push
```

---

## **Switching to String Widget for Slug Fields**

Due to compatibility issues with the `slug` widget, we'll use the `string` widget for slug fields.

### **Step 1: Update Slug Fields in `config.yml`**

Replace the `slug` widget with the `string` widget in all collections:

```yaml
- { label: "Slug", name: "slug", widget: "string", required: true }
```

### **Step 2: Remove `options` Property**

Since the `string` widget doesn't support `options`, ensure you remove that property.

### **Step 3: Commit and Push Changes**

```bash
git add config.yml
git commit -m "Switch slug fields to string widget"
git push
```

### **Step 4: Manually Enter Slugs in the CMS**

When creating or editing content, manually enter the slug in the provided field.

---

## **Adding Featured Images to Posts**

Enhance your posts by adding featured images.

### **Step 1: Add `featured_image` Field in `config.yml`**

Add the following field to your post collections:

```yaml
- { label: "Featured Image", name: "featured_image", widget: "image", required: false }
```

### **Example for `posts_en`:**

```yaml
fields:
  - { label: "Title", name: "title", widget: "string" }
  - { label: "Slug", name: "slug", widget: "string", required: true }
  - { label: "Date", name: "date", widget: "datetime" }
  - { label: "Featured Image", name: "featured_image", widget: "image", required: false }
  - { label: "Body", name: "body", widget: "markdown" }
```

### **Step 2: Commit and Push Changes**

```bash
git add config.yml
git commit -m "Add featured_image field to posts"
git push
```

### **Step 3: Upload Images via the CMS**

- **Open the CMS:** Navigate to your admin interface.
- **Create/Edit a Post:** Use the image upload field to add a featured image.

### **Step 4: Update Your Templates**

Modify your templates to display the featured image:

**Example in Hugo:**

```html
{{ if .Params.featured_image }}
  <img src="{{ .Params.featured_image }}" alt="{{ .Params.title }}">
{{ end }}
```

---

## **Linking Translations with Relation Fields**

Link corresponding posts in different languages using relation fields.

### **Step 1: Add Relation Fields in `config.yml`**

**Example for `posts_de`:**

```yaml
- label: "Englische Version"
  name: "english_version"
  widget: "relation"
  collection: "posts_en"
  search_fields: ["title", "slug"]
  value_field: "slug"
  display_fields: ["title"]
```

**Example for `posts_en`:**

```yaml
- label: "German Version"
  name: "german_version"
  widget: "relation"
  collection: "posts_de"
  search_fields: ["title", "slug"]
  value_field: "slug"
  display_fields: ["title"]
```

### **Step 2: Commit and Push Changes**

```bash
git add config.yml
git commit -m "Add relation fields for translations"
git push
```

### **Step 3: Link Posts in the CMS**

- **Edit a Post:** Use the relation field to select the corresponding translation.
- **Save Changes:** The slug of the linked post will be stored in the front matter.

### **Step 4: Update Your Templates**

Use the linked slugs to create language switcher links in your templates.

**Example in Hugo:**

```html
{{ if .Params.english_version }}
  <a href="/en/posts/{{ .Params.english_version }}">Read in English</a>
{{ end }}
```

---

## **Troubleshooting Common Issues**

### **Issue 1: Posts Not Appearing in the CMS**

**Solution:**

- **Check Front Matter:** Ensure all required fields are present.
- **Verify File Extensions:** Files should have the correct extension (e.g., `.md`).
- **Confirm Collection Paths:** The `folder` path in `config.yml` should match the content directory.
- **Refresh the CMS:** Clear cache or use an incognito window.

### **Issue 2: Relation Fields Not Displaying Options**

**Solution:**

- **Ensure Unique Slugs:** Slugs must be unique and present in the front matter.
- **Check Field Names:** Field names must be consistent and in lowercase.
- **Verify Collection Names:** The `collection` property should match the collection's `name`.
- **Refresh the CMS:** Clear cache to load the latest data.

### **Issue 3: Errors with the Slug Widget**

**Solution:**

- **Use String Widget:** Replace the `slug` widget with the `string` widget.
- **Remove Unnecessary Scripts:** Update `admin/index.html` to remove slug widget scripts.
- **Manually Enter Slugs:** Instruct editors to input slugs manually.

---

## **Conclusion**

By following this guide, you've successfully set up a multilingual content management system using Decap CMS with GitLab. You've configured collections for different languages, handled slug fields using the string widget, added the ability to include featured images in your posts, and linked translations using relation fields. This setup will streamline your content management workflow and enhance the multilingual capabilities of your website.

**Happy content managing!**
