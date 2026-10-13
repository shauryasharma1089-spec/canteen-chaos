# Debug log


## CC-10 — "Sorting by price is backwards"

**Reproduced:** I opened the app at localhost:3000 and chose "price: low to high" in the sort dropdown. The most expensive dishes came first. When I chose "high to low", the cheapest dishes came first. Both options were reversed, and it happened every time.

**Cause:** In backend/logic/search.js, the two price sort functions had a and b swapped. 'price-asc' used `b.price - a.price` and 'price-desc' used `a.price - b.price`. JavaScript's sort puts a first when the result is negative, so `a.price - b.price` gives the cheapest first.

**Fix:** I swapped both lines back. Now 'price-asc' is `a.price - b.price` and 'price-desc' is `b.price - a.price`. The server decides the sort order in search.js, so I fixed it there and did not need to change the frontend.

**Checked:** I refreshed the app and tried both options. Low to high now
shows the cheapest dish first, and high to low shows the most expensive
first.

**Time:** about 25 minutes, including finding the right file in the backend.


## CC-09 — "The menu shows more dishes than it should"

**Reproduced:** I opened the app at localhost:3000 and refreshed. A very
long list of dishes was already loaded on the first screen, and when I
scrolled down, the same dishes kept repeating and the scroll never ended.In the server terminal the frontend was asking for `page=1 limit=5`, so the server should only have returned 5 dishes per page.

**Cause:** In backend/logic/search.js, the `paginate` function correctly
cut out one page with `list.slice(start, start + perPage)` and stored it
in `items`, but the object it returned had `items: list`. That sent the
whole list instead of the current page. menu.routes.js uses `paged items` as the dishes, so every request returned all the dishes.This is why the page started with too many dishes and why scrolling kept
adding the same ones again.

**Fix:** I changed `items: list` to `items` in the return value of
`paginate`, so it returns the sliced page. I fixed it in `paginate`
because the page is cut there, and the route and frontend were already
doing the right thing with what they received.

**Checked:** After the change and a refresh, the first load shows only
5 dishes, and scrolling loads the next 5 instead of repeating the same
ones.

**Time:** about 40 minutes.