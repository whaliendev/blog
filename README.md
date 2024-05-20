## whalien's Blog 🐳

a mini blog for my daily life, thoughts, and projects.

Powered by [Hugo](https://gohugo.io/) and Themed by [Hugo Mini Theme](https://github.com/nodejh/hugo-theme-mini)

---

The following is configuration guide excerpted from the theme's [README](https://github.com/nodejh/hugo-theme-mini/README.md).

## 2. Getting started

After installing the theme successfully it requires a just a few more steps to get your site running.


### 2.1 The config file

Take a look inside the [`exampleSite`](https://github.com/nodejh/hugo-theme-mini/tree/master/exampleSite) folder of this theme. You'll find a file called [`config.yaml`](https://github.com/nodejh/hugo-theme-mini/blob/master/exampleSite/config.yaml). To use it, copy the [`config.yaml`](https://github.com/nodejh/hugo-theme-mini/blob/master/exampleSite/config.yaml) in the root folder of your Hugo site. Feel free to change the strings in this theme.

> ⚠️ You may need to delete the line: `themesDir: ../../` 

### 2.2 Default Content Language

You can set default content language by `defaultContentLanguage`:

```yaml
defaultContentLanguage: en
```

Default is `en`. Now support:

- `en`: English
- `zh`: Chinese
- `nl`: Dutch 
- `fr`: French
- `es`: Spanish
- `da`: Danish

More about multiple languages: [Multilingual Mode](https://gohugo.io/content-management/multilingual/).

### 2.3 Add Comments

To enable comments, add following to your config file:

- Disqus shortname: `disqusShortname: your-disqus-shortname`
- Enable Comment:

    ```yaml
    params:
      enableComments: true
    ```

### 2.4 Google Analytics

To enable google analytics, add following to your config file:

- Google Analytics ID: `googleAnalytics: your-google-analytics-id`
- Enable Google Analytics:

    ```yaml
    params:
      enableGoogleAnalytics: true
    ```

### 2.5 Logo and favicon

You can replace the log in the top of each page and favicon with your own images. To do that put your own logo and favicon into the `images` directory of your website static directory, then named them `avatar.png` and `favicon.ico`. For example:

```
- content
- static
└── images
    ├── avatar.png
    └── favicon.ico
```

### 2.6 Nearly finished

In order to see your site in action, run Hugo's built-in local server.

```bash
$ hugo server
```

Now enter http://localhost:1313 in the address bar of your browser.

### 2.7 Production

To run in production (e.g. to have Google Analytics show up), run HUGO_ENV=production before your build command. For example:

```bash
HUGO_ENV=production hugo
```

Note: The above command will not work on Windows. If you are running a Windows OS, use the below command:

```bash
set HUGO_ENV=production
hugo
```


## 3. Optional Configuration

### 3.1 Table of Content

To enable table of content, you could set `showToc` to `true`.

For example:

```yaml
showToc: true
```

### 3.2 Disable Comments on a single post

You can set `enableComments` to `false` in front matter to disable disqus comments on a single post.

For example:

```yaml
---
title: Some title
enableComments: false
---
```

### 3.3 Custom CSS and JS

You can put your custom css and js files to `static` directory, or use remote css and js files which start with `http://` or `https://`.

For example:

```yaml
customCSS:
  - css/custom.css # local css in `static/css/custom.css`
  - https://example.com/custom.css # remote css
customJS:
  - js/custom.js # local js in `static/js/custom.js`
  - https://example.com/custom.js # remote js
```

### 3.4 Math Typesetting

Mathematical notation is enabled by [KaTeX](https://katex.org/).

- To enable KaTex globally set the parameter `math` to `true` in project’s configuration
- To enable KaTex on a per page basis include the parameter `math` to `true` in content files

### 3.5 Hidden Post Summary in Home Page 

To hidden post summary in home page, you could set `hiddenPostSummaryInHomePage` to `true`, default is `false`.

For example:

```yaml
hiddenPostSummaryInHomePage: true
```


