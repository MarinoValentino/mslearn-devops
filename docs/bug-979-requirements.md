# Bug 979: Search results showing out-of-stock items

Source: [Azure Boards work item 979](https://dev.azure.com/marinovalentinonexplore/AZ-401/_workitems/edit/979/).

## Status

This document records the work item's requirements. It does not implement a
bug fix, satisfy the acceptance criteria, or resolve the work item.

Implementation is blocked because this repository's `main` branch does not
contain the storefront described in the work item's discussion: a
`CatalogController`, Razor product-card views, and `Product.AvailableStock`.
The separate catalog API under `sre-agent-src` is not that storefront.
The storefront source must be provided before these requirements can be
implemented and validated.

## Problem

Search results include products that are out of stock without any visual
indication, causing customers to click through only to find the item unavailable.

## Acceptance criteria

- [ ] Out-of-stock products display an "Out of Stock" badge.
- [ ] Out-of-stock items appear at the bottom of search results.
- [ ] A filter option lets customers hide out-of-stock items.
- [ ] Unit tests cover the new filtering logic.

## Technical context from the discussion

- Search is handled by `CatalogController`.
- Product availability is stored in `Product.AvailableStock`.
- The frontend uses Razor views for product cards.

These are work-item statements, not verified descriptions of this repository's
implementation.

## Required validation once the source is available

- Unit tests for sorting logic and filtering.
- UI tests for badge visibility.
- An integration test for filter functionality.
