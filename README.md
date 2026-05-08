# AdMob Extension for Defold (Fork)

Defold [native extension](https://www.defold.com/manuals/extensions/) which provides access to AdMob functionality on Android and iOS.

[Manual, API and setup instructions](https://www.defold.com/extension-admob/) is available on the official Defold site.

## Fork Changes

### Added `EVENT_PAID` - Ad Revenue Tracking

This fork adds support for ad revenue data via the `EVENT_PAID` event. This allows you to track revenue from ad impressions for analytics and LTV calculations.

#### Usage

```lua
local function admob_callback(self, message_id, message)
    if message.event == admob.EVENT_PAID then
        -- message.value_micros - revenue in micros (1/1,000,000 of currency)
        -- message.currency_code - currency code ("USD", "EUR", etc.)
        -- message.precision - precision type
        -- message.ad_unit_id - the ad unit that generated revenue
        -- message.response_id - unique identifier of the ad response
        -- message.mediation_adapter_class_name - mediation adapter that served the ad
        -- message.ad_source_name - human readable ad source (e.g. "AdMob Network")

        local revenue = message.value_micros / 1000000
        print("Ad revenue: " .. revenue .. " " .. message.currency_code)

        -- Send to your analytics (Firebase, Adjust, AppsFlyer, Tenjin, etc.)
    end
end
```

#### Event Data

| Field | Type | Description |
|-------|------|-------------|
| `value_micros` | number | Revenue in micros (divide by 1,000,000 for actual value) |
| `currency_code` | string | Currency code (e.g., "USD", "EUR") |
| `precision` | number | 0=UNKNOWN, 1=ESTIMATED, 2=PUBLISHER_PROVIDED, 3=PRECISE |
| `ad_unit_id` | string | Ad unit ID that generated the impression |
| `response_id` | string | Unique identifier of the ad response (may be absent) |
| `mediation_adapter_class_name` | string | Class name of the mediation adapter that served the ad (may be absent) |
| `ad_source_name` | string | Human readable name of the ad source (may be absent) |

#### Supported Ad Types

`EVENT_PAID` is fired for all ad types:
- App Open
- Interstitial
- Rewarded
- Rewarded Interstitial
- Banner

> **Note:** Test ads always return `value_micros = 0` and `precision = 0`. Real revenue data is only available with production ad units.
> Optional fields (`response_id`, `mediation_adapter_class_name`, `ad_source_name`) are only present when AdMob provides them.

#### Tenjin MMP example

The fields above match the JSON body required by Tenjin's `eventAdImpressionAdMob` integration ([Android docs](https://tenjin.com/docs/android-admob/)). Forward them to your platform-specific Tenjin extension:

```lua
local function admob_callback(self, message_id, message)
    if message.event == admob.EVENT_PAID then
        tenjin.event_ad_impression_admob({
            ad_unit_id = message.ad_unit_id,
            currency_code = message.currency_code,
            response_id = message.response_id,
            value_micros = message.value_micros,
            mediation_adapter_class_name = message.mediation_adapter_class_name,
            precision_type = message.precision,
        })
    end
end
```
