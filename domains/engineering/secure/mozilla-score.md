---
title: Mozilla Observatory Scores
nav_order: 4
parent: Secure
grand_parent: Engineering
---
In order to help webmasters better protect their websites and users, Mozilla has built an online scanner [Mozilla Observatory](https://observatory.mozilla.org/) that can check if web servers have the best security settings in place.

Steps to score an A+ security rating from Mozilla Observatory:

- **REACT/ANGULAR**

1. As the first step, Enfore **HTTPS**! Always make sure to add a redirection rule from HTTP to HTTPS. If your application is running on AWS-load balancer, enfore HTTPS on load balancer or if it's hosted elsewhere, add the following code in your app server.

    ```
    app.use(function (req, res, next) {
        if (!req.secure && (process.env.NODE_ENV === `production`) {
            res.redirect("https://" + req.headers.host + req.url)
        }
        else return next();
    });
    ```

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

1. Content Security Policy
   -  For RAILS 5.2 and higher go to `config/content_security_policy.rb` file and add
    ```ruby
      Rails.application.config.content_security_policy do |policy|
        policy.default_src :none
        policy.font_src    :self
        policy.img_src     :self
        policy.object_src  :none
        policy.script_src  :self
        policy.style_src   :self
        policy.report_uri "/csp-violation-report-endpoint"
      end
    ```
      If you want data from a specific url as well as self :-
    ```ruby
      Rails.application.config.content_security_policy do |p|
        ....   
        p.img_src :self, 'url', :http
        ...
      end
    ```
   - For RAILS version lower than 5.2 use gem [Secure_Headers](https://www.rubydoc.info/gems/secure_headers/3.6.3).

2. Cookies
    - For securing cookies in your rails application go to `config/environments/production.rb` file and add `config.force_ssl
    = true`.

3. Cross-origin Resource Sharing
   - add `gem 'rack-cors'` into your gemfile
   - if you are using Rails 3/4 add below code to `config/application.rb` file
    ```ruby
    config.middleware.insert_before 0, "Rack::Cors" do
        allow do
          origins '*'
          resource '*', headers: :any, methods: [:get, :post, :options]
        end
    end
    ```
   - if you are using Rails 5 add below code to `config/application.rb` file if you are using Rails 5
    ```ruby
    config.middleware.insert_before 0, Rack::Cors do
        allow do
          origins '*'
          resource '*', headers: :any, methods: [:get, :post, :options]
        end
    end
    ```
   - On production site we should only add the UI domain as the origin whereas in staging it can be "*".
     ```ruby
     config.middleware.insert_before 0, Rack::Cors do
         allow do
         origins 'www.xyz.com', '127.0.0.1:3000',
          /\Ahttp:\/\/192\.168\.0\.\d{1,3}(:\d+)?\z/
          # regular expressions can be used here
           resource '*', headers: :any, methods: [:get, :post, :options]
         end
     end
     ```

4. HTTP Strict Transport Security
  - For HSTS in your rails application `config/environments/production.rb` file add `config.force_ssl = true`.

5. Redirection
  - For redirecting from bad urls in your application `config/environments/production.rb`  file add `config.force_ssl = true`.

6. Referrer Policy
  - In Rails applications, security headers can be set in either `config/application.rb` or in specific `config/environments/` files.
  - In `config/environments/production.rb` file add
   ```ruby
    Rails.application.configure do
      config.action_dispatch.default_headers = {
            'Referrer-Policy' => 'strict-origin-when-cross-origin'
      }
    end
   ```

7.  X-Content-Type-Options
  - In Rails applications, security headers can be set in either `config/application.rb` or in specific `config/environments/` files.
  - In `config/environments/production.rb` file add
  ```ruby
    Rails.application.configure do
      config.action_dispatch.default_headers = {
            'X-Content-Type-Options' => 'nosniff'
      }
    end
  ```

8.  X-Frame-Options
  - In Rails applications, security headers can be set in either `config/application.rb` or in specific `config/environments/` files.
  - In `config/environments/production.rb` file add
  ```ruby
    Rails.application.configure do
      config.action_dispatch.default_headers = {
            'X-Frame-Options' => 'SAMEORIGIN'
      }
    end
  ```

9.   X-XSS-Protection
  - In Rails applications, security headers can be set in either `config/application.rb` or in specific `config/environments/` files.
  - In `config/environments/production.rb` file add
  ```ruby
    Rails.application.configure do
      config.action_dispatch.default_headers = {
            'X-XSS-Protection' => '1; mode=block'
      }
    end
  ```

  ---

Supplementary resources:

[Content Security Policy Reference](https://content-security-policy.com/).
