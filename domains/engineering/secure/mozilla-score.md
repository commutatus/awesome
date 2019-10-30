---
title: Mozilla Observatory Scores
nav_order: 4
parent: Secure
grand_parent: Engineering
---
In order to help webmasters better protect their websites and users, Mozilla has built an online scanner [Mozilla Observatory](https://observatory.mozilla.org/) that can check if web servers have the best security settings in place.

Steps to score an A+ security rating from Mozilla Observatory:

- **REACT/ANGULAR**

1. As the first step, Enfore **HTTPS**! Always make sure to add a redirection rule from HTTP to HTTPS.

2. Next, we are going to use [HELMET](https://helmetjs.github.io/) for increasing the rating. Go to your project, and do `npm i helmet`.

3. For **React**, In the `server.js` file, import `helmet` and use it with your app. Make sure to use helmet before handling any custom routes.

    ```
    const express = require('express')
    const helmet = require('helmet')
    const app = express()
    const port = process.env.PORT || 3000

    app.use(helmet())

    ...other configuration


    app.listen(port, () => {
    console.log(`Running on port : ${port}`)
    })
    ```
    For **Angular**, In `server/app.ts` file, import `helmet` before handling any custom routes.

    ```
    import * as helmet from 'helmet';

    app.use(helmet());
    ```

4. Helmet with default configuration will increase the security rating from **F** to **B**. We need to do one more thing to score an **A+** which is to implement CSP ([Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)).

5. For **React**, Just below the `app.use(helmet())` line in the `server.js` file, add the following:

    ```
    app.use(
        helmet({
            contentSecurityPolicy: {
                directives: {
                scriptSrc: ["'self'"]
                baseUri: ["'self'"],
                objectSrc: ["'self'"],
                styleSrc: ["'self'"],
                fontSrc: ["'self'"],
                frameAncestors: ["'self'"],
                defaultSrc: ["'self'"],
                connectSrc: ["'self'"],
                imgSrc: ["'self'"],
                formAction: ["'self'"]
                }
            }
        })
    )
    ```
    Make the above changes for **Angular** in `server/app.ts` file.

6. The above configuration will block all external sources except resources from the same origin. Whitelist the sources you want to load as follows: 

    [![content-security-policy-source-reference](/assets/images/content-security-policy-source-reference.png)](/assets/images/content-security-policy-source-reference.png)

    An example of production configuration for `styleSrc` and `connectSrc` would be:

    ```
    //allow styles to load from same origin, google and cloudfront.
    styleSrc: ["'self'",'https://fonts.googleapis.com','https://*.cloudfront.net']

    //allow content to load from same origin and c66.
    connectSrc: ["'self'", 'http://*.c66.me/graphql']

    ```
7. Re-run the mozilla observatory test and you should now see an **A+**

    [![mozilla-observatory-max-score](/assets/images/mozilla-observatory-max-score.png)](/assets/images/mozilla-observatory-max-score.png)

---

- **RAILS**

---



Supplementary resources:

[Content Security Policy Reference](https://content-security-policy.com/).
