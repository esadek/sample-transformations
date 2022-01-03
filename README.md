# Sample RudderStack Transformations

[RudderStack Transformations](https://rudderstack.com/product/transformations/) give you the ability to code custom JavaScript functions to implement specific use-cases on your event data. This repository contains some useful transformation templates that you can use to create your own transformations. For more information on RudderStack Transformations, check out the [documentation](https://rudderstack.com/docs/transformations/).

## Table of Contents

- [Getting Started](#getting-started)
- [Filtering](#filtering)
  - [Blacklist Event Names](#blacklist-event-names)
  - [Whitelist Email Domains](#whitelist-email-domains)
- [Sampling](#sampling)
  - [User Based](#user-based)
- [Enrichment](#enrichment)
  - [Geolocation Data](#geolocation-data)
  - [User Data](#user-data)
  - [Browser Data](#browser-data)
  - [Dynamic Header](#dynamic-header)
  - [Dynamic Path](#dynamic-path)
- [Masking](#masking)
  - [Replace PII](#replace-pii)
- [Cleaning](#cleaning)
  - [Remove Null Properties](#remove-null-properties)
- [Miscellaneous](#miscellaneous)
  - [Source ID](#source-id)
  - [Change Event Type](#change-event-type)
  - [Batch](#batch)

## Getting Started

The sample transformations included in this repository can be added via the RudderStack dashboard.

Adding a new user-defined transformation function is quite simple:

1. Log into the [RudderStack dashboard](https://app.rudderstack.com/).
2. Click the [Transformations](https://app.rudderstack.com/transformations) link.
3. Click the [CREATE NEW](https://app.rudderstack.com/transformations/add) button, assign a name to the transformation, and add the transformation code within the `transformEvent` function.

For detailed steps on adding a new transformation, check out the [documentation](https://rudderstack.com/docs/transformations/#adding-a-transformation).

## Filtering

### Blacklist Event Names

1. Drop event if blacklist includes event name
2. Return event otherwise

```javascript
export function transformEvent(event, metadata) {
    const eventNames = ["game_load_time", "lobby_fps"];
    const eventName = event.event;
    if (eventName && eventNames.includes(eventName)) return;
    return event;
}
```

### Whitelist Email Domains

1. Return event if whitelist includes email domain
2. Drop event otherwise

```javascript
export function transformEvent(event, metadata) {
    const domains = ["rudderstack.com", "rudderlabs.com"];
    const email = event.context?.traits?.email;
    if (email && domains.includes(email.split("@").pop())) return event;
    return;
}
```

## Sampling

### User Based

1. Import `cyrb53` function from [hash](libraries/hash.js) library 
2. Drop event if remainder of hashed user ID less than 5
3. Return event otherwise

```javascript
import { cyrb53 } from "hash";

export function transformEvent(event, metadata) {
    const userId = event.context?.traits?.userId;
    if (userId && cyrb53(userId) % 10 < 5) return;
    return event;
}
```

## Enrichment

### Geolocation Data

1. Fetch geolocation data from external IP2Location API
2. Add data to event
3. Return event

```javascript
export async function transformEvent(event, metadata) {
    if (event.request_ip) {
        const res = await fetch("https://api.ip2location.com/v2/?ip=" + event.request_ip.trim() + "&addon=<required addon e.g.geotargeting>&lang=en&key=<IP2Location_API_Key>&package=<package as required e.g. WS10>");
        event.geolocation = res;
    }
    return event;
}
```

### User Data

1. Get user data from external Clearbit API
2. Add data to event's traits
3. Return event

```javascript
export async function transformEvent(event) {
    const email = event.context?.traits?.email;
    if (email) {
        const res = await fetch("https://person.clearbit.com/v2/combined/find?email=" + email, {
            headers: {
                "Authorization": "Bearer <your_clearbit_secure_key"
            }
        });
        event.context.traits.enrichmentInfo = res;
    }
    return event;
}
```

### Browser Data

1. Import `UAParser` function from [user agent parser](libraries/userAgentParser.js) library
2. Add browser data to event if present in user agent
3. Return event

```javascript
import { UAParser } from "userAgentParser";

export function transformEvent(event, metadata) {
    const userAgent = event.context?.userAgent;
    if (userAgent) {
        const parser = new UAParser();
        const browser = parser.setUA(userAgent).getBrowser();
        if (browser.name) event.context.browser = browser;
    }
    return event;
}
```

### Dynamic Header

1. Add dynamnic header to event
2. Return event

```javascript
export function transformEvent(event, metadata) {
    event.header = {
        dynamic_header_1: "dynamic_header_1_value",
        dynamic_header_2: "dynamic_header_2_value"
    };
    return event;
}
```

### Dynamic Path

1. Add dynamic path to event's endpoint base URL
2. Return event

```javascript
export function transformEvent(event, metadata) {
  event.appendPath = `/path/${var1}/search?param=${var2}`;
  return event;
}
```

## Masking

### Replace PII

1. Import `walk` function from [fuzzy find replace](libraries/fuzzyFindReplace.js) library
2. Replace values where keys approximately match target keys
3. Return event

```javascript
import { walk } from "fuzzyFindReplace";

export function transformEvent(event, metadata) {
    const targetKeys = [
        "SSN",
        "Social Security Number",
        "social security no.",
        "social sec num",
        "ssnum"
    ];
    walk(event, targetKeys, "XXX-XX-XXXX");
    return event;
}
```

## Cleaning

### Remove Null Properties

1. Remove properties with null values
2. Return event

```javascript
export function transformEvent(event) {
    if (event.properties) {
        let keys = Object.keys(event.properties);
        if (keys) {
            keys.forEach(key => {
                if (event.properties[key] === null) delete event.properties[key];
            })
        }
    }
    return event;
}
```

## Miscellaneous

### Source ID

1. Do something if event from specified source
2. Return event

```javascript
export function transformEvent(event, metadata) {
    if (metadata(event).sourceId === "12345") {
        // Do something
    }
    return event;
}
```

### Change Event Type

1. Change event from `track` to `identify` if conditions are met
2. Return event

```javascript
export function transformEvent(event) {
    let updatedEvent = event;
    if (
        event.type === "track" && 
        event.event === "ide-authentication" && 
        event.properties?.email && 
        event.properties.email !== ""
    ) {
        updatedEvent.type = "identify";
        let updatedContext = event.context || {}; 
        updatedContext.traits = updatedContext.traits || {};
        updatedContext.traits.email = event.properties.email;
        updatedEvent.context = updatedContext;
    }
   return updatedEvent;
}
```

### Batch

1. Do something for each event
2. Return events

```javascript
export function transformBatch(events, metadata) {
    events.forEach(event => {
        // Do something
    });
    return events;
}
```

## License

[MIT](LICENSE)