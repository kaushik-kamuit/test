Flowchart

flowchart LR
  %% ======= KAMUIT E2E SWIMLANE =======
  subgraph Rider
    R1[Enter pickup & drop (Places)]
    R2[See route, ETA, fare]
    R3[Confirm request]
    R4[Track driver ETA / live]
    R5[Rate & receipt]
  end

  subgraph Driver
    D0[Go Online\n(set dest, seats, max detour)]
    D1{Started route?}
    D2[Offer shows pickup,\nδ detour, ETA]
    D3[Accept]
    D4[Navigate to pickup → In ride]
    D5[Complete ride]
  end

  subgraph Match Engine / Backend
    M0[New ride request]
    M1[Fetch ONLINE drivers]
    M2[Circular radius filter\nST_DWithin(driver, pickup, R)]
    M3{Driver EN_ROUTE?}
    M4[CONE filter (ahead-of-driver)\nheading ± θ within R_cone]
    M5[POLYLINE corridor/overlap\non-route & ahead-of-progress]
    M6[Compute detour δ via Routes API\n(simulate insertion)]
    M7[Rank by (δ, pickup distance)\n→ send offer]
    M8[Fallback queue:\nretries • radius expand • timeout decay]
  end

  subgraph Payments (Stripe)
    P1[Pre-auth on accept]
    P2[Final fare (detour-aware split)]
    P3[Capture • Connect transfer]
  end

  subgraph Admin / Trust
    A1[Verify docs • monitor rides]
    A2[Disputes & cancels]
    A3[KPIs: match %, fallback time, detour%]
  end

  %% ======= FLOW ACROSS LANES =======
  R3 --> M0 --> M1 --> M2 --> M3
  D0 --> D1
  M3 -->|NO (idle / not yet started)| M6
  M3 -->|YES (en-route)| M4 --> M5 --> M6
  M6 --> M7 --> D2
  D2 -->|accept| D3 --> R4 --> D4 --> D5 --> P2 --> P3 --> R5
  D2 -->|reject/timeout| M8 --> D2
  A1 -. onboarding .- D0
  A2 -. incidents .- D5
  P1 --> P2
