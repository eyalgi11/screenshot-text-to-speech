# Real OCR Image Fixtures

These files are real-world images downloaded from Wikimedia Commons for OCR
stress testing. They are kept as local fixtures so the stress test remains
portable after the initial web fetch.

| File | Source | License | SHA-256 |
| --- | --- | --- | --- |
| `commons-grocery-receipt-vienna.jpg` | [Grocery Store Receipt in Vienna](https://commons.wikimedia.org/wiki/File:Grocery_Store_Receipt_in_Vienna.jpg) | Public domain | `24f913c22b2cc62df2020882f2cb944cfcd23b9b2e9f7a533c7e50d140c5618f` |
| `commons-restaurant-menu-lyon.jpg` | [Menu du jour d'un restaurant asiatique lyonnais en 2017](https://commons.wikimedia.org/wiki/File:Menu_du_jour_d%27un_restaurant_asiatique_lyonnais_en_2017.jpg) | CC0 | `1152722f61ea191d28bdc3be0abb8b13f013bdf46f11626b87f2885e5e9c0bf3` |
| `commons-cheese-store-sign.jpg` | [The Cheese Store of Beverly Hills (sign)](https://commons.wikimedia.org/wiki/File:The_Cheese_Store_of_Beverly_Hills_(sign).JPG) | CC0 | `f7b7730313718226c89061588b536e2fbad1b56ee92617bfab9d40a4ecbbd867` |
| `commons-hebrew-auto-display-sign.jpg` | [Hebrew sign at auto display](https://commons.wikimedia.org/wiki/File:Hebrew_sign_at_auto_display.jpg) | Public domain | `1eec1a7cd3a374c9d1ed3b55d02ffa5ffa16a6c09c37b793ad9607379e94acd4` |

`scripts/stress-ocr` auto-orients and resizes these images into the report
output directory before OCR, but it does not synthesize or redraw their text.
