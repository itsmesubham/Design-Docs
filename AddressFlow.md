# Grabbit Address & Location – Diagrams

*(User flows + backend interaction diagrams for address selection, location permission, and checkout)*

---

## Diagram 1: New User – Location Permission GRANTED

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant App
    participant Addr as AddressService
    participant Loc as Redis(Location)

    U->>App: Login (OTP)
    App->>U: Ask location permission
    U-->>App: Permission Granted
    App->>Addr: POST /users/{id}/location (lat,lng)
    Addr->>Loc: Store location (TTL)
    App->>Addr: POST /checkout/address-context (lat,lng)
    Addr-->>App: serviceable + warehouseId
    App->>U: Confirm address details
    App->>Addr: POST /users/{id}/addresses
    App->>Addr: PUT /users/{id}/default-address
    App->>U: Enter Home Feed
```

---

## Diagram 2: New User – Location Permission DENIED

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant App
    participant Addr as AddressService

    U->>App: Login (OTP)
    App->>U: Ask location permission
    U-->>App: Permission Denied
    App->>U: Show manual address search
    U->>App: Search / Drop Pin
    App->>Addr: POST /checkout/address-context (lat,lng)
    Addr-->>App: serviceable + warehouseId
    App->>U: Confirm address details
    App->>Addr: POST /users/{id}/addresses
    App->>Addr: PUT /users/{id}/default-address
    App->>U: Enter Home Feed
```

---

## Diagram 3: Existing User – Address Switcher

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant App
    participant Addr as AddressService

    App->>Addr: GET /users/{id}/addresses
    Addr-->>App: Address list + default
    U->>App: Open Address Switcher
    U->>App: Select address
    App->>Addr: POST /checkout/address-context (addressId)
    Addr-->>App: serviceable + warehouseId
    App->>U: Refresh catalog (if needed)
```

---

## Diagram 4: Checkout – Ordering for Someone Else (Location Available)

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant App
    participant Addr as AddressService
    participant Loc as Redis(Location)

    U->>App: Proceed to Checkout
    App->>Addr: POST /checkout/address-context (addressId)
    Addr->>Loc: Fetch current location
    Addr->>Addr: Compute H3 + Haversine distance
    Addr-->>App: distance + nudge flag
    App->>U: Show "Ordering for someone else?"
    U->>App: Confirm selection
    App->>Addr: Create Order (address snapshot)
```

---

## Diagram 5: Checkout – No Location Permission

```mermaid
sequenceDiagram
    autonumber
    participant U as User
    participant App
    participant Addr as AddressService

    U->>App: Proceed to Checkout
    App->>Addr: POST /checkout/address-context (addressId)
    Addr->>Addr: Skip distance calculation
    Addr-->>App: nudge (non-distance based)
    App->>U: Show soft nudge
    U->>App: Confirm selection
    App->>Addr: Create Order (address snapshot)
```

---

## Diagram 6: Backend Decision Flow (Simplified)

```mermaid
flowchart TD
    A[Address Selected] --> B{Location Available?}
    B -->|Yes| C[Compute H3 + Haversine]
    B -->|No| D[Skip Distance]
    C --> E{Far From User?}
    D --> F{Address Unusual?}
    E -->|Yes| G[Show Distance-based Nudge]
    F -->|Yes| H[Show Soft Nudge]
    G --> I[User Confirms]
    H --> I
    I --> J[Place Order]
```

---


## Key Design Principle

> **Location enhances experience. Manual address always works.**

---
