---
layout: default
title: XPath Notes
permalink: /xpath-notes
---

# XPath Notes
_2026-04-26_

## Select elements
```xpath
//div
//table
//a
```

## By attribute
```xpath
//div[@class="content"]
//a[@href]
```

## Text match
```xpath
//a[text()="Next"]
//div[contains(text(), "example")]
```

## Partial match
```xpath
//div[contains(@class, "item")]
//a[contains(@href, "page")]
```

## Index
```xpath
(//div)[1]
(//tr)[last()]
```

## Hierarchy
```xpath
//div/span
//table//td
```

## Parent
```xpath
//span/..
```

## Useful patterns

### Table rows
```xpath
//table//tr
```

### Links
```xpath
//a/@href
```

### Text extraction
```xpath
//div/text()
```
