# Cache busting with IIS Rewrite module
For this process to work, you must have the IIS Rewrite 2.1 module installed.

The cache busting works by replacing any references to static resources (CSS, JS, PNG, JPG) in HTML and CSS files with a unique URL.

For example, the following `<script>` tag in an HTML file:

```html
<script src="scripts/main.js"></script>
```

would be replaced with:

```html
<script src="scripts/main.js?c=1234"></script>
```

This changes are done in `<a>`, `<img>`, `<link>` and `<script>` tags in HTML. This can be customized to support more tags.

In CSS files, usage of the `url` directive is replaced with a new URL in a similar manner.

For example, the following CSS style:

```css
.bg-style {
    background-image: url('assets/images/bg.png');
}
```

becomes:

```css
.bg-style {
    background-image: url('assets/images/bg.png?c=1234');
}
```

The rules for rewriting are specified in the `web.config` file under the `<system.webServer>/<rewrite>` rules.

## Setup procedure
1. Clone or download this repository to a machine with IIS installed.
1. Create a new website in IIS and point it to the cloned or downloaded folder. Default settings are fine.

## Process for changes
Whenever changes happen in the web application and a new version needs to be deployed, follow the two steps below to get the caches busted:

1. In the `web.config` file, locate the two instances of `?c=<some number>` and increment the number.

    ![](/docs/webconfig-cvalues.png)

2. The HTML file also needs to be modified from the previous release. If it has not been modified, try adding a custom tag attribute or comment to the file to mark it as modified.

    In this scenario, a custom attribute is added to the `<html>` tag with the same number value from the first step.

    ![](/docs/html-cvalue.png)

## Process to verify cache busting
These steps describe the process for verifying cache busting in Chrome.

* Open Chrome and the Chrome dev tools. Keep the Network tab active with the `Preserve log` checkbox enabled and the `All` filter selected.

    ![](/docs/chrome-devtools-setup.png)

    You will also observe that the static resources (`styles.css` and `iis.png`) URLs have a query string appended.

* Navigate to the app URL. You will see that all resources were retrieved without cache:

    ![](/docs/chrome-first-load.png)

* Refresh the page normally. This can be done multiple times.

    ![](/docs/chrome-subsequent-loads.png)

    Note that:
    * The static resources are being loaded from the browser cache.
    * The `index.html` file is also loaded from cache (as indicated by the `304` status code.)

* Make changes to the CSS file (colors, fonts, etc.), optionally replace the image with another one. You can also change the HTML content.

* Follow the steps in [Process for Changes](#process-for-changes) to prepare the new release.

* Back in the browser, do a normal refresh and you should see the updated version without needing to clear the cache or do a hard refresh.

    ![](/docs/chrome-new-release.png)
