# luci-app-statistics-plot

Proof of concept for displaying OpenWrt LuCI Statistics using client-side SVG plots instead of RRDtool-generated graph images.

## Overview

This project is a modified version of OpenWrt's `luci-app-statistics`.

The original application uses the RRDtool command-line utility to generate graph images on the router.

This implementation retrieves numerical data through a native `rpcd` plugin and renders interactive SVG diagrams using `luci-plot`.

The existing collectd infrastructure, RRD databases and LuCI statistics definitions are retained.

## Architecture

```text
collectd
   |
   v
RRD database files
   |
   v
librrd / rrd_xport()
   |
   v
rrd-ubus (rpcd plugin)
   |
   | ubus JSON response
   v
luci-app-statistics
   |
   v
luci-lib-plot
   |
   v
SVG diagrams in the browser
```

## Goals

- Replace server-side RRDtool graph generation
- Reuse existing collectd RRD databases
- Preserve existing LuCI statistics configuration
- Reuse existing diagram definitions where possible
- Render graphs directly in the browser
- Reduce dependence on RRDtool's graphics components

## Implementation

The modified `statistics/rrdtool.js` requests data through:

```javascript
rpc.declare({
    object: 'rrd',
    method: 'xport',
    params: ['start', 'end', 'step', 'definitions']
});
```

The frontend constructs RRDtool export definitions from the existing statistics configuration and converts the returned data into series for `luci-plot`.

The `rrd.xport` call returns numerical data rather than a PNG or other graph image.

## Dependencies

- OpenWrt with LuCI
- `collectd`
- `collectd-mod-rrdtool`
- `rrd-ubus`
- `luci-lib-plot`

The initial implementation uses a custom RRDtool library build for the native export plugin.

## Current Status

**Working proof of concept.**

Successfully demonstrated:

- Retrieval of real CPU statistics through `rrd.xport`
- JSON data transfer through ubus
- SVG graph rendering using `luci-plot`
- Seven CPU data series with individual colors
- Integration with the existing LuCI Statistics interface
- Dynamic export step calculation based on the selected time range

The implementation has been tested on an OpenWrt device using real collectd data.

## Known Limitations and Remaining Work

- Not all existing statistics diagram definitions have been tested
- Time ranges from one hour to one year require further testing
- Calculated series and special graph configurations need verification
- Error handling and edge cases need further work
- The native RRD export plugin requires additional validation and security review
- Packaging and dependencies need review before upstream integration

This repository is intended to preserve and develop the proof of concept. It is not currently a drop-in replacement for the official OpenWrt package.

## Related Projects

- [rrd-ubus](https://github.com/Reinerost/rrd-ubus) — native RRD export plugin
- [luci-plot](https://github.com/Reinerost/luci-plot) — SVG plotting library
- [luci-plot-module](https://github.com/Reinerost/luci-plot-module) — LuCI integration of the plotting library
- [OpenWrt LuCI](https://github.com/openwrt/luci) — upstream LuCI project

## License

This project is derived from OpenWrt LuCI. The applicable upstream license and copyright notices must be preserved.

