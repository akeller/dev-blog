# Amara Graham's blog

This repo/blog was created by following https://github.com/mathieudutour/medium-to-own-blog!

Some gotchas I ran into:
* Use Netlify's domain management to get a TLS certificate from Let's Encrypt in the HTTPS section. It may take a few seconds to fetch the cert and display on the page. 
* siteUrl - make sure you include `https://`
* Fuzzy images - make sure you do a `npm run build` after running the script before deploying to Netlify
