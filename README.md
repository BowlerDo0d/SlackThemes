## How to use:
- Copy the code snippet below
- Paste the copied code into slack's `ssb-interop.js` file
  - Usual slack app location:
    - MacOS: `/Applications/Slack.app/`
    - Windows: `%homepath%\AppData\Local\slack\{latest_version}\`
  - File location: `{slack_app}/Contents/Resources/app.asar.unpacked/src/static/ssb-interop.js`
- Replace `{cssUrl}` with the direct https link to a raw css file
  - Example: `https://raw.githubusercontent.com/BowlerDo0d/SlackThemes/master/darkness.css`

## Code snippet

```js
// First make sure the wrapper app is loaded
document.addEventListener("DOMContentLoaded", function() {

  // Then get its webviews
  let webviews = document.querySelectorAll(".TeamView webview");

  // Fetch our CSS in parallel ahead of time
  const cssPath = '{cssUrl}';
  let cssPromise = fetch(cssPath).then(response => response.text());

  // Insert a style tag into the wrapper view
  cssPromise.then(css => {
     let s = document.createElement('style');
     s.type = 'text/css';
     s.innerHTML = css;
     document.head.appendChild(s);
  });

  // Wait for each webview to load
  webviews.forEach(webview => {
     webview.addEventListener('ipc-message', message => {
        if (message.channel == 'didFinishLoading')
           // Finally add the CSS into the webview
           cssPromise.then(css => {
              let script = `
                    let s = document.createElement('style');
                    s.type = 'text/css';
                    s.id = 'slack-custom-css';
                    s.innerHTML = \`${css}\`;
                    document.head.appendChild(s);
                    `
              webview.executeJavaScript(script);
           })
     });
  });
});
```
