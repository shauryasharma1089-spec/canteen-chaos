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


## CC-08 — "An old coupon still works"

**Reproduced:** I added a dish to the cart and applied the coupon FRESHERS24. The discount was applied even though the offer expired in
2024 (coupons.json has "expiresAt": "2024-09-30" and the description
says "Expired freshers offer"). The coupon list endpoint was fine, so
the problem was only when a coupon is actually applied.

**Cause:** In backend/logic/pricing.js, `applyCoupon` checks that the
coupon exists, that `usesLeft` is above zero, the minimum order, veg-only and the time slot, but it never checks `expiresAt`. The function even receives `now` as a parameter but never uses it. The expiry was only checked in GET /api/coupons, which hides expired coupons from the list, so a user who typed the code by hand could still use it.

**Fix:** I added a check in `applyCoupon`, right after the `usesLeft`
check: if the coupon has an `expiresAt` and that date is not after `now`, it returns `valid: false` with the reason "This coupon has expired". I fixed it in `applyCoupon` because that is the one place every order goes through to apply a coupon, so the rule is enforced on the server and not just in the list.

**Checked:** FRESHERS24 now shows "This coupon has expired" and gives no
discount. Valid coupons (BYTE10, FLAT50) still apply normally.

**Time:** about 20 minutes.


## CC-01 — "The search suggestions are behind everything"

**Reproduced:** I opened the app and typed a letter in the search box.
The suggestions list appeared, but the menu cards were drawn on top of
it, so only the top part could be clicked.

**Cause:** In frontend/style.css, `.suggest-box` has `z-index: 100`, but
it sits inside `.search-wrap`, which had `z-index: 1`. A child cannot
rise above the stacking level of its parent, so the whole search area
(level 1) was still below `#menu` (2) and the dish cards (3).

**Fix:** I changed `.search-wrap` to `z-index: 45`. That puts it above
the menu and the filter bar (40) but still below the site header (50). I
fixed it on the parent because raising the number on `.suggest-box`
alone could never work.

**Checked:** After a refresh, I typed in the search box and clicked the
top, middle and bottom suggestions. All of them respond now.

**Time:** about 15 minutes.



## CC-02 — "Can't read anything in dark mode"

**Reproduced:** I switched the site to dark mode. The dish names and
prices were almost invisible, while the grey line under the name was
fine.

**Cause:** In frontend/style.css (line 294), `.dish-body` had a fixed
`color: #2b2118` (dark brown). Dark mode only changes the CSS variables
like `--ink`, so this hardcoded colour stayed dark on a dark background.
The grey line was fine because it uses `var(--muted)`. I found this with
DevTools: Inspect on the dish name, then "Inherited from" in the Styles
panel.

**Fix:** I changed it to `color: var(--ink)`, which is dark in light mode and light in dark mode.

**Checked:** Dish names and prices are readable in dark mode, and still
readable in light mode.

**Time:** about 15 minutes.


## Could not fix

### CC-03 — "The menu is wider than my phone"

**What I tried:** Checked at 360px and zoomed to 500%. Used the console
to list elements wider than the screen.

**Where I got to:** Only the filter bar elements (search box, select,
category tabs) went past the edge. The dish cards did not. I could not
reproduce the cut-off Add to Cart buttons.

**What I would try next:** Check the filter bar CSS (min-width, no
wrapping) and test on a real phone.


## CC-05 — "The category bar scrolls away on my phone"

**Reproduced:** On a narrow screen the category and filter bar did not
stay at the top while I scrolled through the menu. I could not use the
device toolbar, so I zoomed the browser to 300% to get a narrow layout.

**Cause:** `.filters` is `position: sticky; top: 0`, which is correct.
But in style.css there is a rule inside `@media (max-width: 480px)`:
`.view { overflow-x: hidden; }`. The filters sit inside `.view`, and
`overflow-x: hidden` turns `.view` into its own scroll container, so
sticky stops following the page scroll.

**Fix:** I changed it to `overflow-x: clip`. `clip` still hides anything
sticking out sideways, but it does not create a scroll container, so
sticky works again.

**Checked:** At 300% zoom the filter bar stays at the top while the menu
scrolls. At that zoom the bar takes up a lot of the screen, so I could
not judge it on a real phone.

**Time:** about 25  minutes.