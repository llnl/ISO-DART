Transmission & Interconnections
================================

This page documents the transmission and DC tie interconnection data products available through the ERCOT API.

DC Tie Flows
------------

**Report:** NP6-626-CD
**Method:** ``get_dc_tie_flows()``
**Update Frequency:** Hourly
**Retention:** 7-8 days via API

Actual hourly flows for ERCOT's four DC interconnections with neighboring grids.

ERCOT's DC Ties
~~~~~~~~~~~~~~~

ERCOT has four non-synchronous DC interconnections:

1. **DC_N** - North tie to SPP (Southwest Power Pool)
2. **DC_E** - East tie to Mexico (CFE)
3. **DC_L** - Laredo tie to Mexico (CFE)
4. **DC_R** - Railroad tie to Mexico (CFE)

**Flow Convention:**

* **Negative values** = Imports into ERCOT
* **Positive values** = Exports from ERCOT

Basic Usage
~~~~~~~~~~~

.. code-block:: python

    from datetime import datetime
    from lib.iso.ercot import ERCOTClient, ERCOTConfig

    config = ERCOTConfig.from_ini_file()
    client = ERCOTClient(config)

    # Get DC tie flows for a specific day
    start_dt = datetime(2025, 8, 15, 0, 0, 0)
    end_dt = datetime(2025, 8, 15, 23, 59, 59)

    dc_flows = client.get_dc_tie_flows(
        post_datetime_from=start_dt,
        post_datetime_to=end_dt
    )

    # Analyze flows by tie
    import pandas as pd
    df = pd.DataFrame(dc_flows['data'])

    print("\nDC Tie Flow Summary:")
    by_tie = df.groupby('dcTieName')['powerFlow'].agg(['mean', 'min', 'max', 'sum'])
    print(by_tie)

    # Calculate net position
    net_flow = df['powerFlow'].sum() / len(df)
    if net_flow < 0:
        print(f"\nERCOT was a NET IMPORTER: {abs(net_flow):,.1f} MW avg")
    elif net_flow > 0:
        print(f"\nERCOT was a NET EXPORTER: {net_flow:,.1f} MW avg")
    else:
        print("\nERCOT was BALANCED")

    client.cleanup()

**Data Fields:**

* ``postDatetime`` / ``operatingTime`` - Timestamp of measurement
* ``dcTieName`` - DC tie identifier (DC_N, DC_E, DC_L, DC_R)
* ``powerFlow`` - Power flow in MW (negative = import, positive = export)
* ``operatingDate`` - Operating date
* ``operatingHour`` - Operating hour

Advanced Analysis
~~~~~~~~~~~~~~~~~

.. code-block:: python

    # Analyze import/export patterns by time of day
    df['hour'] = pd.to_datetime(df['operatingTime']).dt.hour

    hourly_pattern = df.groupby('hour')['powerFlow'].mean()
    print("\nAverage Flow by Hour:")
    print(hourly_pattern.round(1))

    # Find peak import/export hours
    peak_import_hour = hourly_pattern.idxmin()
    peak_export_hour = hourly_pattern.idxmax()

    print(f"\nPeak Import Hour: {peak_import_hour}:00")
    print(f"  Avg Flow: {hourly_pattern[peak_import_hour]:,.1f} MW")

    print(f"\nPeak Export Hour: {peak_export_hour}:00")
    print(f"  Avg Flow: {hourly_pattern[peak_export_hour]:,.1f} MW")

    # Analyze by tie and direction
    import_hours = df[df['powerFlow'] < 0].groupby('dcTieName')['powerFlow'].agg(['count', 'mean'])
    export_hours = df[df['powerFlow'] > 0].groupby('dcTieName')['powerFlow'].agg(['count', 'mean'])

    print("\nImports by Tie:")
    print(import_hours)
    print("\nExports by Tie:")
    print(export_hours)

**Use Cases:**

* Import/export pattern analysis
* Grid interconnection studies
* Cross-border energy flow tracking
* Net interchange calculations
* Regional market integration analysis

**Dashboard:** https://www.ercot.com/gridmktinfo/dashboards/dctieflows

**Important Notes:**

* Real-time dashboard updates every 5 minutes
* API data may have timing differences from dashboard
* Only 7-8 days of data available via API
* For historical data beyond 8 days, use EIA-930 API

---

SCED Binding Transmission Constraints
--------------------------------------

**Report:** NP6-86-CD
**Method:** ``get_sced_binding_transmission_constraints()``
**Update Frequency:** Hourly (when constraints occur)
**Retention:** 7 days via API

Shadow prices and details for binding or violated transmission constraints in SCED, including when DC ties or interfaces hit their limits.

Understanding Shadow Prices
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Shadow prices** represent the marginal cost ($/MW) of relieving a transmission constraint. High shadow prices indicate:

* Significant congestion on the constrained element
* Large price separation across the constraint
* High cost to deliver additional power through the constraint

A shadow price of $15/MW means that relieving the constraint by 1 MW would reduce total system costs by $15.

Basic Usage
~~~~~~~~~~~

.. code-block:: python

    from datetime import datetime

    # Get binding constraints for a specific day
    start_dt = datetime(2025, 8, 15, 0, 0, 0)
    end_dt = datetime(2025, 8, 15, 23, 59, 59)

    constraints = client.get_sced_binding_transmission_constraints(
        sced_timestamp_from=start_dt,
        sced_timestamp_to=end_dt
    )

    # Find most expensive constraints
    df = pd.DataFrame(constraints['data'])

    if not df.empty:
        top_10 = df.nlargest(10, 'shadowPrice')[
            ['contingencyName', 'overloadedElementName', 'shadowPrice',
             'elementFlow', 'elementLimit']
        ]

        print("\nTop 10 Most Expensive Constraints:")
        print(top_10)

        print(f"\nTotal Constraint-Hours: {len(df)}")
        print(f"Average Shadow Price: ${df['shadowPrice'].mean():,.2f}/MW")
        print(f"Max Shadow Price: ${df['shadowPrice'].max():,.2f}/MW")
    else:
        print("No binding constraints during this period")

**Data Fields:**

* ``scedTimestamp`` - SCED timestamp
* ``contingencyName`` - Contingency identifier
* ``contingencyType`` - Type of contingency
* ``overloadedElementName`` - Name of constrained element
* ``fromStation`` / ``toStation`` - Element endpoints
* ``voltageLevel`` - Voltage level
* ``shadowPrice`` - Shadow price in $/MW
* ``maxShadowPrice`` - Maximum penalty price
* ``elementLimit`` - Element capacity limit in MW
* ``elementFlow`` - Actual flow in MW
* ``constraintType`` - Type of constraint

DC Tie Constraint Analysis
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    # Check if DC ties were constrained
    dc_tie_constraints = df[
        df['overloadedElementName'].str.contains('DC|TIE', case=False, na=False)
    ]

    if not dc_tie_constraints.empty:
        print(f"\n⚠️  Found {len(dc_tie_constraints)} DC tie constraint intervals")

        print("\nDC Tie Constraints:")
        for _, row in dc_tie_constraints.iterrows():
            element = row['overloadedElementName']
            shadow_price = row['shadowPrice']
            flow = row['elementFlow']
            limit = row['elementLimit']
            overload_pct = ((flow - limit) / limit * 100) if limit > 0 else 0

            print(f"\n  {element}")
            print(f"    Shadow Price: ${shadow_price:,.2f}/MW")
            print(f"    Flow: {flow:,.1f} MW / Limit: {limit:,.1f} MW")
            print(f"    Overload: {overload_pct:.1f}%")
    else:
        print("\n✓ No DC tie constraints detected")

Congestion Pattern Analysis
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    # Analyze constraint frequency by element
    constraint_freq = df['overloadedElementName'].value_counts().head(10)
    print("\nMost Frequently Constrained Elements:")
    print(constraint_freq)

    # Calculate total congestion cost (approximate)
    # Shadow price * (actual flow - limit) estimates the cost per interval
    df['overload_MW'] = (df['elementFlow'] - df['elementLimit']).clip(lower=0)
    df['congestion_cost'] = df['shadowPrice'] * df['overload_MW']

    total_cost = df['congestion_cost'].sum()
    print(f"\nEstimated Total Congestion Cost: ${total_cost:,.0f}")

    # Analyze by time of day
    df['hour'] = pd.to_datetime(df['scedTimestamp']).dt.hour
    hourly_constraints = df.groupby('hour').size()

    print("\nConstraint Count by Hour:")
    print(hourly_constraints)

**Use Cases:**

* Transmission congestion analysis
* Identifying bottlenecks
* DC tie limit monitoring
* Interface constraint tracking
* Congestion cost calculations
* Transmission planning insights

---

Historical Data Beyond 7 Days
------------------------------

For DC tie flow data beyond the 7-8 day API window, use the **EIA-930 API**:

.. code-block:: python

    # EIA-930 provides hourly ERCOT interchange from July 2015-present
    # Free public API (no authentication required)
    # Dashboard: https://www.eia.gov/electricity/gridmonitor/

    # Note: EIA-930 may aggregate ties differently than ERCOT's
    # individual DC_N, DC_E, DC_L, DC_R breakdown

Consult your project's EIA client for accessing historical interchange data.

---

CLI Usage
---------

All transmission data products can be accessed via the command-line interface:

.. code-block:: bash

    # DC tie flows (last 7 days available)
    python isodart.py ercot transmission --trans-type dc_ties --start 2025-08-15 --duration 7

    # Binding transmission constraints
    python isodart.py ercot transmission --trans-type binding_constraints --start 2025-08-15 --duration 1

---

Example Scripts
---------------

See ``examples/ercot_transmission_analysis.py`` for a comprehensive example demonstrating:

* DC tie flow pattern analysis
* Net import/export calculations
* Binding constraint identification
* Congestion analysis

.. code-block:: bash

    # Run all analyses
    python examples/ercot_transmission_analysis.py --date 2025-08-15

    # Run specific analysis
    python examples/ercot_transmission_analysis.py --date 2025-08-15 --analysis ties
    python examples/ercot_transmission_analysis.py --date 2025-08-15 --analysis constraints

---

Related Documentation
---------------------

* `ERCOT DC Tie Dashboard <https://www.ercot.com/gridmktinfo/dashboards/dctieflows>`_
* `ERCOT Data Product Catalog <https://www.ercot.com/mp/data-products>`_
* :doc:`pricing` - For LMP and congestion analysis
* :doc:`api-guide` - General API usage patterns
