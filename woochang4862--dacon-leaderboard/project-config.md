---
trigger: always_on
description: You are an Expert Shopify Theme Developer with advanced knowledge of Liquid, HTML, CSS, JavaScript, and the latest Shopify Online Store 2.0 features.
---


    You are an Expert Shopify Theme Developer with advanced knowledge of Liquid, HTML, CSS, JavaScript, and the latest Shopify Online Store 2.0 features.
---
description: Best practices for Shopify theme development with Liquid, JavaScript, and CSS
globs: **/*.liquid, assets/*.js, assets/*.css, sections/*.liquid, snippets/*.liquid, templates/**/*.liquid, blocks/*.liquid
alwaysApply: true
---
# Liquid Development Guidelines

## Liquid Rules

### Valid Filters
* **Cart**
    * `item_count_for_variant`: `cart | item_count_for_variant: {variant_id}`
    * `line_items_for`: `cart | line_items_for: object`
* **HTML**
    * `class_list`: `settings.layout | class_list`
    * `time_tag`: `string | time_tag: string`
    * `inline_asset_content`: `asset_name | inline_asset_content`
    * `highlight`: `string | highlight: string`
    * `link_to`: `string | link_to: string`
    * `placeholder_svg_tag`: `string | placeholder_svg_tag`
    * `preload_tag`: `string | preload_tag: as: string`
    * `script_tag`: `string | script_tag`
    * `stylesheet_tag`: `string | stylesheet_tag`
* **Collection**
    * `link_to_type`: `string | link_to_type`
    * `link_to_vendor`: `string | link_to_vendor`
    * `sort_by`: `string | sort_by: string`
    * `url_for_type`: `string | url_for_type`
    * `url_for_vendor`: `string | url_for_vendor`
    * `within`: `string | within: collection`
    * `highlight_active_tag`: `string | highlight_active_tag`
* **Color**
    * `brightness_difference`: `string | brightness_difference: string`
    * `color_brightness`: `string | color_brightness`
    * `color_contrast`: `string | color_contrast: string`
    * `color_darken`: `string | color_darken: number`
    * `color_desaturate`: `string | color_desaturate: number`
    * `color_difference`: `string | color_difference: string`
    * `color_extract`: `string | color_extract: string`
    * `color_lighten`: `string | color_lighten: number`
    * `color_mix`: `string | color_mix: string, number`
    * `color_modify`: `string | color_modify: string, number`
    * `color_saturate`: `string | color_saturate: number`
    * `color_to_hex`: `string | color_to_hex`
    * `color_to_hsl`: `string | color_to_hsl`
    * `color_to_rgb`: `string | color_to_rgb`
    * `hex_to_rgba`: `string | hex_to_rgba`
* **String**
    * `hmac_sha1`: `string | hmac_sha1: string`
    * `hmac_sha256`: `string | hmac_sha256: string`
    * `md5`: `string | md5`
    * `sha1`: `string | sha1: string`
    * `sha256`: `string | sha256: string`
    * `append`: `string | append: string`
    * `base64_decode`: `string | base64_decode`
    * `base64_encode`: `string | base64_encode`
    * `base64_url_safe_decode`: `string | base64_url_safe_decode`
    * `base64_url_safe_encode`: `string | base64_url_safe_encode`
    * `capitalize`: `string | capitalize`
    * `downcase`: `string | downcase`
    * `escape`: `string | escape`
    * `escape_once`: `string | escape_once`
    * `lstrip`: `string | lstrip`
    * `newline_to_br`: `string | newline_to_br`
    * `prepend`: `string | prepend: string`
    * `remove`: `string | remove: string`
    * `remove_first`: `string | remove_first: string`
    * `remove_last`: `string | remove_last: string`
    * `replace`: `string | replace: string, string`
    * `replace_first`: `string | replace_first: string, string`
    * `replace_last`: `string | replace_last: string, string`
    * `rstrip`: `string | rstrip`
    * `slice`: `string | slice`
    * `split`: `string | split: string`
    * `strip`: `string | strip`
    * `strip_html`: `string | strip_html`
    * `strip_newlines`: `string | strip_newlines`
    * `truncate`: `string | truncate: number`
    * `truncatewords`: `string | truncatewords: number`
    * `upcase`: `string | upcase`
    * `url_decode`: `string | url_decode`
    * `url_encode`: `string | url_encode`
    * `camelize`: `string | camelize`
    * `handleize`: `string | handleize`
    * `url_escape`: `string | url_escape`
    * `url_param_escape`: `string | url_param_escape`
    * `pluralize`: `number | pluralize: string, string`
* **Localization**
    * `currency_selector`: `form | currency_selector`
    * `translate`: `string | t`
    * `format_address`: `address | format_address`
* **Customer**
    * `customer_login_link`: `string | customer_login_link`
    * `customer_logout_link`: `string | customer_logout_link`
    * `customer_register_link`: `string | customer_register_link`
    * `avatar`: `customer | avatar`
    * `login_button`: `shop | login_button`
* **Format**
    * `date`: `string | date: string`
    * `json`: `variable | json`
    * `structured_data`: `variable | structured_data`
    * `weight_with_unit`: `number | weight_with_unit`
* **Font**
    * `font_face`: `font | font_face`
    * `font_modify`: `font | font_modify: string, string`
    * `font_url`: `font | font_url`
* **Default**
    * `default_errors`: `string | default_errors`
    * `default`: `variable | default: variable`
    * `default_pagination`: `paginate | default_pagination`
* **Payment**
    * `payment_button`: `form | payment_button`
    * `payment_terms`: `form | payment_terms`
    * `payment_type_img_url`: `string | payment_type_img_url`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Woochang4862) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
