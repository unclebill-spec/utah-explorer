# Utah Explorer — agent handoff

Static Leaflet map for a nurse household, the same shared app as the Kentucky, Tennessee, Massachusetts, Maine, Vermont, Montana, Wyoming and Idaho Explorers (`app.js`, `areas.js`, `perm.js`, `profiles.js`, `extras.js`, `style.css` are byte-identical copies of `/workspace/kentucky/explorer/*`; another worker edits them there, so run `scripts/sync_shared.sh` right before every build/publish and log any shared-file edit in KY `explorer/AGENTS.md`). `sw.js` differs only in its cache prefix (`utx-`). Utah-specific settings live in `explorer/build.py` `STATE` (written by `scripts/port_build.py --restate`, marker PORT_ST).

Live: https://unclebill-spec.github.io/utah-explorer/ · repo unclebill-spec/utah-explorer · progress log: `/workspace/utah/STATUS.md` (newest first, ET).
Utah was copied from the Idaho code base (Oct 4 2026), including Idaho's bath-count fix. Every script in `scripts/` picks the state from the folder it runs in (`scripts/common.py` ST: UT, fips 49, ls prefix `utx_`) and reads the price cap from `common.CAP` (UT 600000).

## Caps (Bill, Oct 4 2026: "Let's do Utah now also 600,000 for the housing limit"; other states keep their own caps)
5+ acres $300k–$600k; 1+ acre 3bd/2ba < $600k; near-hospital 1,600+ sqft 3bd/2ba < $600k (townhomes/condos OK, good condition, ≤ 10 min of a hospital with a 10+ bed ER). No cabin category.

## Blocks
29 counties (`scripts/common.py` -> `data/statewide/raw/ut_blocks_500k.zip`).

## Data pipeline (run from /workspace/utah; pandas scripts use /workspace/kentucky/.venv/bin/python, the rest /usr/bin/python3)
1. `scripts/hospitals_research.py` (CMS + Utah Bureau of EMS trauma designations (ems.utah.gov, Oct 2026) + `EXTRA_UT` Holy Cross Mountain Point (Lehi) and West Valley campuses + border trauma centers EIRMC Idaho Falls, Portneuf, St. Mary's Grand Junction, UMC Las Vegas, Flagstaff) -> `data/hospitals.json`; `scripts/layers.py` -> block CSVs, schools (`scripts/fetch_nces.py`), SEDA, RN wages (O*NET/BLS OEWS May 2025; county -> BLS area map `data/statewide/raw/oews_area_of.json`).
2. `scripts/appeal_build.py` (OSRM drives, cached) -> appeal shading; `scripts/er_beds.py` -> `data/hospital_er_beds.json` (10+ bed ERs for the near-hospital rule; estimate rule, 39 of 40 acute hospitals).
3. Climate normals (`climate/raw`, `climate/build_clim.py` BOX UT) -> `data/clim.json`; activities `data/osm/wd_act.py` + `data/osm/wp_cat_act.py` (UT bbox + hand trails).
4. Compare areas: `compare/land.py salt-lake-county-ut utah-county-ut washington-county-ut weber-county-ut cache-county-ut` (run inside compare/) + `compare/areas_build.py` -> `data/areas.json` (Salt Lake City, Provo, St. George, Ogden, Logan).
5. Homes: `scripts/zsearch.py` (Zillow county searches at the $600k caps) -> `data/zsearch/`; `scripts/listings_build.py` (PER_COUNTY UT 14; `MAX_NEW_DETAIL` env, cache `data/zsearch/detail_cache.json` so reruns fetch only missing pages; whole-number bath totals count as full baths unless the description mentions a half bath / powder room) -> `listings.json` + `listing-photos/` (log `data/lb.log`); `scripts/bargains.py`; `scripts/top_lists.py`.
6. Permanent RN jobs: `scripts/perm_jobs.py` -> `data/perm_jobs.json` (reuses the KY readers). Utah sources: Intermountain (Workday imh "IntermountainCareers", queries nurse/RN, Utah locations only via LOCAL/OTHER regexes; Primary Children's jobs are left unmapped), University of Utah Health (iCIMS careers-uuhc.icims.com; JSON-LD title is "UNAVAILABLE" so the link text is used; Huntsman jobs unmapped), Holy Cross / CommonSpirit (Radancy, Utah facet 6252001-5549030), Lifepoint (Oracle ORC, Utah location 300000006719161: Castleview, Ashley Regional); HCA MountainStar last: careers.hcahealthcare.com answers 403, so `data/hca_websearch.json` (web-search list) is used.
7. Travel RN jobs: helpers in `/workspace/tj_ut` (`run.sh`: Vivian + Advantis Utah pages), then `scripts/travel_jobs.py` (UT ALIAS incl. HCA/HealthTrust/MountainStar by city; Utah State Hospital excluded) -> `data/travel_jobs.json`. Needs explorer/data/data.js (run after a build).
8. Phase 4: `data/airports/airports.py`; `data/attractions/wd2.py` + `make_attractions.py` (UT HANDS, 22 state-park campgrounds); Crexi `data/forsale/crexi_list.py`, `st_forsale.py` (hand review HAND_DROP / RELABEL_*), `make_forsale.py`; thumbnails `explorer/fetch_thumbs.py` (after a build).
9. Border items: `/workspace/border` (`scripts/static.py UT ID WY`, `scripts/make_border.py UT ID WY`), copy `out/<ST>.json` -> each `explorer/border.json`. ID and WY are covered maps (their homes/jobs come in both ways); CO/AZ/NV/NM come from the static layers.
10. Ski areas + peaks: `/workspace/mtn/scripts/make_state.py UT` -> `explorer/mtn.json` + `img/mtn/` (see /workspace/mtn/PROGRESS.md). Ticket prices come only from skiresort.com.
11. Publish: `sh scripts/sync_shared.sh && cd publish && PATH=/usr/bin:$PATH ./publish.sh -m "msg"` (flock /tmp/utx_publish.lock, pull, build, minify, secscan of dist + full history, push, waits for Pages; live check share/county-salt-lake.html).
12. Tests (state from the folder): `perf/smoke.py BASE TAG`, `perf/test_homes.py`, `perf/test_perm.py`, `perf/test_p4.py`, `perf/loadtime.py URL`, `perf/sw_check.py URL...`; screenshots in `perf/shots/`.

## Known gaps (Oct 4 2026)
(see the end of STATUS.md's newest entry; filled in at the end of the build)
