---
trigger: always_on
description: Guidance for Claude Code when working in this repository. Conversational rules
---

# CLAUDE.md

Guidance for Claude Code when working in this repository. Conversational rules
shared by the maintainer already live under `.claude/rules/`; this file covers
**project-specific** architecture, conventions, and gotchas that are not
derivable from the code in seconds.

---

## 1. What this project is

`django-cart` is a lightweight, session-backed shopping cart library for Django
4.2+. It is distributed on PyPI and has been maintained since ~2012. The public
API surface is the `Cart` class in `cart/cart.py`; everything else (models,
templatetags, signals, pluggable calculators) exists to support that class.

- Package name on PyPI: **`django-cart`**
- Importable app name: **`cart`** (not `django_cart`)
- Python: **3.10+**, Django: **4.2+** (CI matrix up to Py 3.14 / Dj 6.0)
- Current version: see `pyproject.toml` (`version` field). Bump this when
  releasing; CI publishes on tag push.
- License: **MIT** (`LICENSE`). Relicensed from LGPL-3.0 in v3.0.11 —
  see `CHANGELOG.md` for the rationale. Copying code in from other MIT /
  BSD / Apache-2.0 projects is fine; avoid copying from GPL / LGPL /
  AGPL sources without clearing it first.

---

## 2. Repository map

```
cart/                        # the installable Django app
├── __init__.py              # (legacy default_app_config — see gotcha §7.1)
├── apps.py                  # CartConfig
├── admin.py                 # CartAdmin + ItemInline (Discount NOT registered — §7.7)
├── cart.py                  # Cart class + exceptions + CART_ID — main API
├── models.py                # Cart, Item (GFK), ItemManager, Discount, DiscountType
├── signals.py               # 5 Django signals (optional import in cart.py)
├── session.py               # CartSessionAdapter, DjangoSessionAdapter, CookieSessionAdapter
│                            #   ⚠️ declared but NOT wired into Cart — see §7.2
├── tax.py                   # TaxCalculator base + DefaultTaxCalculator + get_tax_calculator()
├── shipping.py              # ShippingCalculator base + default + factory
├── inventory.py             # InventoryChecker base + default + factory
├── templatetags/cart_tags.py  # cart_item_count, cart_summary, cart_is_empty, cart_link
├── management/commands/clean_carts.py  # cron-friendly purge command
├── migrations/              # 0001..0005 — do NOT squash without version bump
└── views.py                 # intentionally empty

tests/
├── settings.py              # used by pytest (pyproject sets DJANGO_SETTINGS_MODULE)
├── urls.py                  # admin only — most tests don't hit HTTP
├── test_app/                # FakeProduct + FakeProductNoPrice (test-only product models)
├── fixtures/fake_products.json  # present but UNUSED by tests (§7.9)
├── test_cart.py             # ~2200 lines, 177 tests — the bulk of the suite
├── test_v300.py             # discounts/tax/shipping/inventory (64 tests)
├── test_integration.py      # "integration" — still uses MagicMock requests (§7.3)
├── test_performance.py      # 3 loose timing benchmarks
├── test_session.py          # session adapters (14 tests)
├── test_signals.py          # 7 tests
└── test_templatetags.py     # 13 tests

docs/
├── PROJECT_ANALYSIS.md              # Mar 2026 analysis (partly stale — ref v2.2.13)
├── PROJECT_ANALYSIS_2026_03_29_0243am.md  # earlier snapshot
└── ROADMAP.md                       # thin "future considerations" list

pyproject.toml               # pytest + coverage config (runtests.py deleted in Phase 8; .coveragerc folded in at Phase 0)
.pre-commit-config.yaml      # black, isort, flake8, mypy — NOT run in CI (§7.8)
.github/workflows/ci.yml     # test matrix + publish-on-tag to PyPI
.github/dependabot.yml       # weekly pip + gh-actions updates
```

---

## 3. Core architecture

### 3.1 The `Cart` facade

`cart.cart.Cart` is a thin wrapper around one `cart.models.Cart` row. On
construction:

1. Look up `CART-ID` in `request.session`.
2. If it points to a non-checked-out DB cart, reuse it; otherwise create one
   and write its id back into `request.session`.
3. Initialise `self._cache = {}` — an **in-memory** cache for `count()` and
   `summary()` invalidated by every mutation. Not a Django cache.

Everything else (`add`, `update`, `remove`, `merge`, `apply_discount`, `tax`,
`shipping`, `total`, …) operates on `self.cart` and its related `items`.

### 3.2 Products via ContentType (generic FKs)

`Item` stores a product as `(content_type, object_id)`. The custom
`ItemManager._inject_content_type` translates `product=<instance>` kwargs in
`.get()` / `.filter()` into `content_type=…, object_id=…` — so call sites read
naturally (`Item.objects.filter(cart=c, product=p)`) while the schema stays
generic.

`Item.product` is a cached property: first access resolves via
`content_type.model_class().objects.get(pk=object_id)` and stores on
`_product_cache`. This avoids N+1 when iterating a single `Cart` instance but
does NOT share cache across items — a 50-item cart iteration is still 50
queries for products. See ROADMAP for prefetch work.

### 3.3 Pluggable subsystems

Tax, shipping, and inventory all follow the same pattern:

```
cart/<subsystem>.py:
    class XxxBase(ABC):              # abstract interface
    class DefaultXxx(XxxBase):       # no-op default

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bmentges/django-cart](https://github.com/bmentges/django-cart) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
