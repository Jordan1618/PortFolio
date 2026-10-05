# **1) What I wanted to do :**

I wanted people to find the site on a search engine and to give it a real address, not only a github.io link. This note keeps what I learned about the domain name, DNS, HTTPS and SEO (search engine optimization: making a site easy to find and understand for search engines). Values like IP addresses and keys are left out on purpose.

# **2) The domain name :**

- A **domain name** is the human address (`comprendrepourtous.fr`). It is rented each year from a **registrar**. Mine is OVHcloud, which also offered a free web hosting.
- **Why GitHub Pages and not the OVH hosting:** the free OVH hosting has no shell access, so it cannot run my Python build script on each push. GitHub Pages with GitHub Actions can, and deploys on every `git push` with no manual file transfer.
- **DNS** is the phone book of the internet. The records set at the registrar tell browsers where the site really is.
- **The two forms of the address:** the bare domain (`comprendrepourtous.fr`) and the `www` subdomain. They are two different names.
- **What failed first:** I tried a `CNAME` record on the bare domain. OVH refused it, because a CNAME cannot sit at the root of a domain that already has other records (NS, SOA). This is a rule of DNS, not an OVH choice.
- **What worked:** a `CNAME` record on `www` pointing to my GitHub Pages address. In GitHub Settings, Pages, Custom domain, I entered the `www` name. GitHub's "DNS check" turned green and the HTTPS certificate was issued automatically.
- **The bare domain:** a **permanent redirect (301)** at OVH from the bare domain to `https://www...`, set as "visible" and not "invisible", to keep the SEO value (a 301 tells search engines the move is permanent).
- **The `CNAME` file** at the root of the repository tells Pages which domain to serve. It must also be written into the built site, or the custom domain disappears at the next deployment.
- **A confusion I cleared:** deploying "from a branch" serves the repository files as they are, while "GitHub Actions" serves the output of my build. With the wrong setting the site was broken, and the HTTPS box stayed grey.
- **HTTPS ("Enforce HTTPS"):** I could not tick it on 7 August. It only works when the DNS is correct and the certificate is issued. The task stayed open until 18 September, after the DNS had been unstable for a while and then stabilized.
- **A 404 lesson:** a URL like `/ComprendrePourTous/guides/...` returned "page not found" although the file existed on GitHub. A GitHub path and a website path are different things, and the site root is the domain.

# **3) What my generator does for SEO :**

- **A unique title and a meta description on each page.** This is what appears in search results.
- **A canonical link** on each page, saying which address is official, so the same page reached by two addresses is not counted twice. The official one is the `www` address.
- **Open Graph tags** (title, description, type, URL, and since 28 September a share image): they control the preview when a link is shared. I removed the number of guides from the image, because a cached image would show an old number.
- **A `sitemap.xml`** (about 530 addresses) and a **`robots.txt`** that allows everything and gives the sitemap address.
- **Redirect pages with `noindex`** for the old URLs after I renamed three guides, with a canonical link to the new one.
- **A 404 page,** breadcrumbs, previous and next links and a table of contents on each chapter: internal links help search engines understand the structure.
- **Clean readable URLs** made from the titles (`/guides/pour-nous/`).
- **Fast pages:** static HTML, no framework, inline SVG, versioned CSS and JS files (cache-busting).
- **JSON-LD structured data** (16 September): a `DefinedTerm` on each notion, an `Article` on each chapter and a `WebSite` on the home page. It helps Google and generative search engines (ChatGPT, Perplexity) cite the site without guessing what a page is about.
- **An About page** for trust: my first name, my GitHub and LinkedIn links, the method, the limits, and the choice to stress honesty, rigor and free access rather than a sales pitch.
- **A favicon as a real SVG file** (a gradient with two speech bubbles).

# **4) Search Console and Bing, step by step :**

I asked Claude in the browser how to make the site appear in search results, since it did not show up, for example on Brave. The steps we followed or planned:

1. **Verify the domain in Google Search Console.** I added the domain as a property and Google gave me a `TXT` record to add in the OVH DNS zone. A TXT record is a line of text in the DNS that proves I control the domain. Once it was found, the property was validated and I got the dashboard. It works at the domain level, so it covers `www` and the bare domain.
2. **Submit a sitemap.** In the Sitemaps menu, I type only `sitemap.xml` and Search Console adds my domain in front. The field did not prefill. A sitemap is an XML file listing all the URLs of the site, a plan for machines, so Googlebot does not have to discover pages one by one by following links. When I first checked `/sitemap.xml` it answered 404, so I had to look at how my site was built. My generator writes `sitemap.xml` at the root of the built site, so the address to submit is the one on the `www` domain.
3. **Request indexing of the home page** with URL inspection, to speed up the first visit.
4. **Do the same on Bing Webmaster Tools.** It can import the site from Google Search Console in a few clicks. I was told that Bing also feeds ChatGPT search and sometimes other engines, but I did not check this myself.
5. **Remember that engines have their own index.** The site not showing on Brave Search did not mean there was an error on my side, since each engine builds its own index and takes its own time.

Result: the sitemaps now work and are accepted by Search Console.

Nb : the sitemap must be at the root of the published folder, not in a sub-folder, and it only exists on the site after a deployment.

# **5) What Google taught me (15 and 16 September) :**

- **Indexation was confirmed on Google** after the DNS had stabilized. I could then see how the site looked in real results.
- **Problem 1:** the warning banner was displayed as the description under the guide cards and in search snippets, instead of the real beginning of the text.
- **Problem 2:** the home page description ("The body, emotions and relationships, explained for real") was too short and vague, so Google looked for a more informative passage in the page and picked the warning banner again. I rewrote it so that the first sentence says what the site is. The lesson: if the meta description is weak, a search engine writes its own.
- **Problem 3:** the favicon was an inline data URI, and search engines did not use it (Bing showed a generic globe). I replaced it by a static versioned SVG file. Even after that, I still noticed the icon was missing on Bing, so I learned that an icon can take time to appear.
- **Site name:** I changed it to title case ("Comprendre Pour Tous") for a cleaner look in results.

# **6) What I learned :**

- A domain, DNS records, a custom domain setting and a certificate are four separate things, and each one can block the others
- A TXT record is the standard way to prove I own a domain to Google, with no file to upload
- A sitemap tells search engines what exists, and Search Console tells me what they actually did with it
- A 301 redirect is how to say "this address moved for good" without losing the SEO value, and it is also the answer for a bare domain that cannot have a CNAME
- Technical SEO only removes obstacles: duplicates, broken links, missing descriptions, slow pages. Content (sourced, dated, complete pages) is what earns the ranking
- A search engine often rewrites snippets, so the first lines of a page and the meta description must be written on purpose
- Renaming a URL without a redirect loses the links people already shared
- A new domain takes time to be indexed and trusted
- I have no analytics by choice, so I learn from what search engines show (results, icons, snippets), not from audience statistics
