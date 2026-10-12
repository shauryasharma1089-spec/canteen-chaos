# Debug log


## CC-10 — "Sorting by price is backwards"

**Reproduced:** I opened the app at localhost:3000 and chose "price: low to high" in the sort dropdown. The most expensive dishes came first. When I chose "high to low", the cheapest dishes came first. Both options were reversed, and it happened every time.

**Cause:** In backend/logic/search.js, the two price sort functions had a and b swapped. 'price-asc' used `b.price - a.price` and 'price-desc' used `a.price - b.price`. JavaScript's sort puts a first when the result is negative, so `a.price - b.price` gives the cheapest first.

**Fix:** I swapped both lines back. Now 'price-asc' is `a.price - b.price` and 'price-desc' is `b.price - a.price`. The server decides the sort order in search.js, so I fixed it there and did not need to change the frontend.

**Checked:** I refreshed the app and tried both options. Low to high now
shows the cheapest dish first, and high to low shows the most expensive
first.

**Time:** about 25 minutes, including finding the right file in the backend.