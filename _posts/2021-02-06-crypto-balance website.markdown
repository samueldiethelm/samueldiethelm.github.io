---
layout: post
categories: crypto
title: Crypto-balance.online cryptocurrency tracker
tags: [crypyocurrencies, coincap, vuejs, fiat, ledger]
---

Some time ago I read that many people are storing their cryptocurrency balances on apps and this is not good from a privacy point. These apps are very convenient but have the downside that when you store your balances, this information in synced to online servers for _your convenience_. However, the crypto world is quickly expanding and some of those sites are not always very secure and can leak precious information. Ledger itself, being a very reputable hardware wallet manufacturer had a security breach which exposed thousands of their customer's details: [July 2020 Ledger leak](https://www.ledger.com/addressing-the-july-2020-e-commerce-and-marketing-data-breach){:target="_blank"}. This information will then be potentially used for targeted attacks on crpytocurrency _hodlers_. This is why I decided to create my own anonymous cryptocurrency tracker tool.

<!--more-->

![Crypto-Balance.online](/img/crypto-balance.png){: .center-image }

In fact, I started creating this online little project before the Ledger leak, but still this argument adds to my motivation now that I write about it in this small post. My goal was to run a minimalistic single page app that could pull crypto prices from some public API and store any information locally. I decided to go with Vue.js and Coincap.io public API. I was also going to try Coinmarketcap's API, but they require requesting a token, so Coincap.io was more straight forward. Also since the app is publicly available because it runs entirely on the browser, I didn't want to expose any credentials.

I have already done some experiments with Vue.js and it very easy to learn and quick to implement. After creating a simple layout and pulling the top cryptocurrencies from Coincap API, I got a simple list displaying prices. Add some responsive input boxes and you can update balances instantly. The API is only used to retrieve the prices and balances are calculated on the browser, so nothing is shared with coincap. Later I also added a fiat currency conversion option, so that it would be possible to display prices not only in US Dollars but in any local currency. I found this API which did the trick nicely: [exchangeratesapi.io](https://exchangeratesapi.io/){:target="_blank"}. Again, only rates are retrieved here so data stays on the browser.

Finally, I needed to be able to save the balances in the browser to add the convenience factor but I wanted to avoid using cookies because those are sent over any web request and it could be argued that this can be exloited. Therefore, I decided to go with [localStorage](https://www.w3schools.com/html/html5_webstorage.asp){:target="_blank"} which lets you store information on the browser and is not shared unless a script running on the site explicitly does so. However, this is a web app, so the code can be checked by anyone. When saving, the balances, and any currency preferences are stored in a JSON string into the browser storage and it would all automatically populate the next time you visit the app.

# Next steps
I spent very little time on this site but I might do some improvements over time. Yesterday I fixed the layout which was not displaying correcly on smartphones. Eventually I would like to add password protection and encrypt the settings in localStorage and more options to export the balances. Right now I allow exporting and importing via a JSON file only. I have some more little ideas that I might explore further on.

Feel free to have a look at it at [crypto-balance.online](https://crypto-balance.online){:target:"_blank"}.
