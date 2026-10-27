---
layout: page_v2
title: Transfon Security Real-Time Data Submodule
display_name: Transfon Security RTD Module
description: Checks each page view for bots and invalid traffic before the auction, passes the result to bidders and the ad server, and can stop bid requests for invalid traffic.
page_type: module
module_type: rtd
module_code : transfonSecurityRtdProvider
enable_download : true
vendor_specific: true
sidebarType : 1
---

# Transfon Security Real-Time Data Submodule
{:.no_toc}

* TOC
{:toc}

## Overview

```text
Module Name: Transfon Security Rtd Provider
Module Type: Rtd Provider
Maintainer: support@transfon.com
```

The Transfon Security module checks each page view against the Transfon Security traffic quality service (bots, data centre, proxy, VPN and Tor networks, header and session anomalies) before the auction. It passes the verdict to bidders and the ad server, and can optionally stop bid requests for invalid traffic.

Page views are reported to the Transfon Security project, where the traffic quality breakdown is available in the Transfon Security dashboard.

A Transfon Security project license is required: create a project in [Transfon Security](https://www.transfon.com/security/prebid-rtd) and copy its license (10 characters, for example `5badb34d98`) from the Installation page.

## Integration

Build Prebid.js with the RTD core module and this provider:

```bash
gulp build --modules=rtdModule,transfonSecurityRtdProvider,...
```

Configure the provider:

```javascript
pbjs.setConfig({
  realTimeData: {
    auctionDelay: 300,
    dataProviders: [{
      name: 'transfonSecurity',
      waitForIt: true,
      params: {
        licenseId: 'YOUR_LICENSE',
        blockIvt: false
      }
    }]
  }
});
```

## Parameters

{: .table .table-bordered .table-striped }
| Name | Type | Description | Default |
| :-- | :-- | :-- | :-- |
| name | String | Must be `transfonSecurity` | |
| waitForIt | Boolean | Wait for the verdict (up to `auctionDelay`) before the auction starts | `false` |
| params.licenseId | String | Transfon Security project license, 10 lowercase hex characters (required) | |
| params.blockIvt | Boolean | Remove all bid requests when `tss_qs` is `0` or the page view matches the block rules set in the Transfon Security project | `false` |
| params.timeout | Number | Verdict timeout in milliseconds; when the Transfon Security tag is on the page but has not answered after half of it, the module asks the edge itself with the rest | `1000` |

## Output

{: .table .table-bordered .table-striped }
| Where | Key | Values |
| :-- | :-- | :-- |
| Global first party data | `ortb2.site.ext.data.tss_qs` | `1` valid traffic, `0` invalid traffic (project block rules do not change it) |
| Ad server targeting (per ad unit) | `tss_qs` | `"1"`, `"0"` |

The page is also marked invalid when the browser shows an unambiguous automation signal (`navigator.webdriver`, automation framework globals, a bot user agent or a software WebGL renderer).

If the verdict is not available in time, or the request fails, and no browser signal fired, the auction continues unchanged and no signal is added.

When the Transfon Security tag is also on the page, the module reuses the tag's verdict and does not send its own page view.

## Using tss_qs in the ad server

The `tss_qs` key-value is set on every ad unit in the auction, including ad units without bids.

* **Google Ad Manager (display):** `pbjs.setTargetingForGPTAsync()` sends `tss_qs` with the other Prebid keys. Create a key named `tss_qs` with the values `0` and `1`, then target line items with `tss_qs` is not `0`. Use "is not 0" rather than "is 1": page views without a verdict carry no key and keep serving.
* **Google Ad Manager (video):** the GAM video module adds `tss_qs` to `cust_params` when the ad unit is passed to `buildVideoUrl`.
* **Xandr (AST):** `pbjs.setTargetingForAst()` sends `tss_qs` with the other Prebid keys.
* **Other ad servers:** read `tss_qs` from `pbjs.getAdserverTargeting()` and pass it the same way as the `hb_` keys.
* **No ad server:** bidders still receive `site.ext.data.tss_qs`, and `blockIvt` stops bid requests for invalid traffic.

## Privacy

The module is registered under TCF Global Vendor List ID 1370. The verdict request carries cookies, and a first party visitor id (`__unis_uid`) is stored, only when device access is allowed for this vendor. Without consent the verdict is still computed from the request itself.

The page view is only sent when Prebid's activity controls allow this module to report analytics (TCF Purpose 7 for vendor 1370 when GDPR enforcement is on).
