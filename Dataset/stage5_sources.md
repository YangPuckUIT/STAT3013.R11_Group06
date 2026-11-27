# Stage 5 data: sources and coding rules

## Revenue (`revenue_monthly.csv`)

| Game | Source | Period | Months |
|---|---|---|---|
| Genshin Impact | https://revenue.ennead.cc/games/genshin-impact | 2022-12 to 2026-09 | 46 |
| Honkai: Star Rail | https://revenue.ennead.cc/games/star-rail | 2023-04 to 2026-09 | 42 |

- Retrieved on 14 November 2026 from the site's *Historical Data* table (GLOBAL row, Total).
- Figures are **Sensor Tower estimates** of mobile revenue (App Store and Google Play) in USD per month, not official company data.
- Only **GLOBAL** revenue (excluding mainland China) is used. The site reports Chinese revenue only from 2024-01, and Chinese Android revenue is itself estimated as iOS × 1.75, so adding it would break the series. The `CN` and `CN_EST` columns are kept for reference only.
- 2026-10 is excluded because it was not final at retrieval.
- Figures up to 2024-10 are rounded to the nearest million (`IS_APPROX = 1`). HSR GLOBAL figures are rounded in every month.
- 2023-04 is the HSR launch month (released 26 April 2023, about five days of sales).
- All 88 values were checked against the source table.

## Banner calendar (`banner_calendar.csv`)

Each variable is 0/1 per game and month and counts in the month the event **starts**; several events in one month still give 1.

| Variable | = 1 when |
|---|---|
| `BANNER` | A new game version (x.y) starts that month |
| `HOT_CHAR` | A new limited 5-star character debuts that month (reruns, 4-star and standard characters excluded; collaboration and alternate-form characters included) |
| `PROMO_EVENT` | Anniversary, first top-up double bonus reset, or collaboration that month |

- Lunar New Year is **not** coded: the revenue data exclude China and HSR has no comparable event.
- Dates follow UTC+8. HSR version 4.3 starts 2026-06-01 (UTC+8) and is counted in June.
- `VERSION`, `NEW_5STAR` and `EVENT_NOTE` are check columns and are not used in the models.
- Genshin and HSR were coded independently by two people and cross-checked; the Genshin months most likely to be misassigned were checked against the Genshin Impact Wiki.

Sources:

- Genshin banner history: https://www.genshincodex.com/en/banners, https://gamingonphone.com/news/genshin-impact-all-wish-banner-history/, https://gamewith.net/genshin-impact/article/show/30371
- HSR version dates: https://hsrtierlist.net/hsr-release-date; HSR banner history: https://game8.co/games/Honkai-Star-Rail/archives/474951, https://game8.co/games/Honkai-Star-Rail/archives/415899
- Genshin Impact Wiki (Fandom), Version pages 4.7, 4.8, 7.0, 7.1 and character pages, used for cross-checking.
- Top-up resets and anniversaries: official HoYoLAB announcements, e.g. https://www.hoyolab.com/article/32169822 (Genshin 5.0), https://www.hoyolab.com/article/44584844 (HSR 4.2), https://www.hoyolab.com/article/37896315 (HSR 3.2); Genshin 4.1 reset: https://www.pcgamesn.com/genshin-impact/4-1-top-up-reset
- HSR × Fate collaboration: https://game8.co/games/Honkai-Star-Rail/archives/462460
