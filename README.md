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
        
        local revenue = message.value_micros / 1000000
        print("Ad revenue: " .. revenue .. " " .. message.currency_code)
        
        -- Send to your analytics (Firebase, Adjust, AppsFlyer, etc.)
    end
end
```

#### Event Data

| Field | Type | Description |
|-------|------|-------------|
| `value_micros` | number | Revenue in micros (divide by 1,000,000 for actual value) |
| `currency_code` | string | Currency code (e.g., "USD", "EUR") |
| `precision` | number | 0=UNKNOWN, 1=ESTIMATED, 2=PUBLISHER_PROVIDED, 3=PRECISE |

#### Supported Ad Types

`EVENT_PAID` is fired for all ad types:
- App Open
- Interstitial
- Rewarded
- Rewarded Interstitial
- Banner

> **Note:** Test ads always return `value_micros = 0` and `precision = 0`. Real revenue data is only available with production ad units.
