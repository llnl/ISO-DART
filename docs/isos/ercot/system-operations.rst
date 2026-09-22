System Operations
=================

This page documents the system operations and market data products available through the ERCOT API.

Actual System Lambda
--------------------

**Report:** NP6-905-CD
**Method:** ``get_actual_system_lambda()``
**Update Frequency:** Every 5 minutes (SCED)

System lambda represents the marginal cost of serving the next MW of load in the ERCOT system.

Understanding System Lambda
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**System lambda** is the system-wide energy price component that reflects:

* The marginal cost of the next MW of generation
* Operating reserve scarcity conditions
* The Operating Reserve Demand Curve (ORDC) adder

**High lambda values indicate:**

* Tight supply conditions
* Scarcity pricing in effect
* Potential need for demand response
* High Operating Reserve Demand Curve (ORDC) adders

**Typical ranges:**

* **Normal operations:** $20-50/MWh
* **Tight conditions:** $100-500/MWh
* **Scarcity pricing:** >$1,000/MWh
* **Emergency conditions:** >$5,000/MWh (approaching cap of $9,000/MWh)

Basic Usage
~~~~~~~~~~~

.. code-block:: python

    from datetime import datetime
    from lib.iso.ercot import ERCOTClient, ERCOTConfig

    config = ERCOTConfig.from_ini_file()
    client = ERCOTClient(config)

    # Get system lambda for a specific day
    start_dt = datetime(2025, 8, 15, 0, 0, 0)
    end_dt = datetime(2025, 8, 15, 23, 59, 59)

    lambda_data = client.get_actual_system_lambda(
        sced_timestamp_from=start_dt,
        sced_timestamp_to=end_dt
    )

    # Analyze lambda values
    import pandas as pd
    df = pd.DataFrame(lambda_data['data'])

    print("\nSystem Lambda Statistics:")
    print(f"  Peak:    ${df['systemLambda'].max():>10,.2f}/MWh")
    print(f"  Average: ${df['systemLambda'].mean():>10,.2f}/MWh")
    print(f"  Minimum: ${df['systemLambda'].min():>10,.2f}/MWh")

    client.cleanup()

**Data Fields:**

* ``scedTimestamp`` - SCED timestamp (5-minute intervals)
* ``systemLambda`` - System lambda in $/MWh
* ``ordcAdder`` - Operating Reserve Demand Curve adder in $/MWh (if available)

Scarcity Pricing Analysis
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    # Identify scarcity periods
    high_scarcity = df[df['systemLambda'] > 1000]
    extreme_scarcity = df[df['systemLambda'] > 5000]

    total_intervals = len(df)

    print(f"\nScarcity Periods:")
    print(f"  High (>$1000/MWh):    {len(high_scarcity):>4} intervals ({len(high_scarcity)/total_intervals*100:.1f}%)")
    print(f"  Extreme (>$5000/MWh): {len(extreme_scarcity):>4} intervals ({len(extreme_scarcity)/total_intervals*100:.1f}%)")

    if not high_scarcity.empty:
        print("\nHigh Scarcity Events:")
        for _, row in high_scarcity.head(10).iterrows():
            timestamp = row['scedTimestamp']
            lambda_val = row['systemLambda']
            print(f"  {timestamp}: ${lambda_val:,.2f}/MWh")

Time-of-Day Patterns
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    # Analyze lambda by time of day
    df['timestamp'] = pd.to_datetime(df['scedTimestamp'])
    df['hour'] = df['timestamp'].dt.hour

    hourly_avg = df.groupby('hour')['systemLambda'].agg(['mean', 'min', 'max'])
    print("\nAverage System Lambda by Hour:")
    print(hourly_avg.round(2))

    # Find peak price hours
    peak_hour = hourly_avg['mean'].idxmax()
    off_peak_hour = hourly_avg['mean'].idxmin()

    print(f"\nPeak Hour: {peak_hour}:00 (${hourly_avg.loc[peak_hour, 'mean']:.2f}/MWh avg)")
    print(f"Off-Peak Hour: {off_peak_hour}:00 (${hourly_avg.loc[off_peak_hour, 'mean']:.2f}/MWh avg)")

**Use Cases:**

* Real-time market monitoring
* Scarcity pricing analysis
* Energy trading decisions
* Grid reliability assessment
* Operating reserve evaluation
* Demand response trigger identification

---

60-Day DAM Settlement Point Prices
-----------------------------------

**Report:** NP4-188-CD
**Method:** ``get_dam_60day_settlement_point_price()``
**Update Frequency:** Daily (after DAM clearing)
**Retention:** Rolling 60-day window

Historical Day-Ahead Market settlement point prices with a 60-day rolling window.

Basic Usage
~~~~~~~~~~~

.. code-block:: python

    from datetime import date

    # Get DAM settlement point prices for a specific hub
    spp_60d = client.get_dam_60day_settlement_point_price(
        delivery_date_from=date(2025, 7, 1),
        delivery_date_to=date(2025, 8, 31),
        settlement_point="HB_HOUSTON"
    )

    # Calculate statistics
    df = pd.DataFrame(spp_60d['data'])

    print(f"\nHB_HOUSTON DAM Price Statistics:")
    print(f"  Average: ${df['price'].mean():,.2f}/MWh")
    print(f"  Median:  ${df['price'].median():,.2f}/MWh")
    print(f"  Peak:    ${df['price'].max():,.2f}/MWh")
    print(f"  Minimum: ${df['price'].min():,.2f}/MWh")
    print(f"  Std Dev: ${df['price'].std():,.2f}/MWh")

**Data Fields:**

* ``deliveryDate`` - Date of delivery
* ``hourEnding`` - Hour ending (1-24)
* ``settlementPoint`` - Settlement point name
* ``settlementPointType`` - Type (Hub, Load Zone, Resource Node)
* ``price`` - Settlement point price in $/MWh

Common Settlement Points
~~~~~~~~~~~~~~~~~~~~~~~~

**Hubs:**

* ``HB_HOUSTON`` - Houston hub
* ``HB_NORTH`` - North hub
* ``HB_SOUTH`` - South hub
* ``HB_WEST`` - West hub
* ``HB_BUSAVG`` - Bus average

**Load Zones:**

* ``LZ_HOUSTON`` - Houston load zone
* ``LZ_NORTH`` - North load zone
* ``LZ_SOUTH`` - South load zone
* ``LZ_WEST`` - West load zone
* ``LZ_SOUTH_HOUSTON`` - South Houston
* ``LZ_AEN`` - AEN load zone
* ``LZ_CPS`` - CPS load zone
* ``LZ_LCRA`` - LCRA load zone

Price Trend Analysis
~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    # Compare prices across multiple hubs
    hubs = ["HB_HOUSTON", "HB_NORTH", "HB_SOUTH", "HB_WEST"]

    hub_prices = {}
    for hub in hubs:
        data = client.get_dam_60day_settlement_point_price(
            delivery_date_from=date(2025, 8, 1),
            delivery_date_to=date(2025, 8, 31),
            settlement_point=hub
        )
        df = pd.DataFrame(data['data'])
        hub_prices[hub] = df['price'].mean()

    print("\nAverage DAM Prices by Hub (August 2025):")
    for hub, avg_price in sorted(hub_prices.items(), key=lambda x: x[1], reverse=True):
        print(f"  {hub:<15} ${avg_price:>7,.2f}/MWh")

Peak vs Off-Peak Analysis
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    # Define peak hours (HE 7-22 on weekdays)
    df['hour'] = df['hourEnding']
    df['date'] = pd.to_datetime(df['deliveryDate'])
    df['dayofweek'] = df['date'].dt.dayofweek  # Monday=0, Sunday=6

    # Identify peak hours
    df['is_peak'] = (
        (df['dayofweek'] < 5) &  # Weekday
        (df['hour'] >= 7) &      # Hour 7 or later
        (df['hour'] <= 22)       # Hour 22 or earlier
    )

    peak_price = df[df['is_peak']]['price'].mean()
    off_peak_price = df[~df['is_peak']]['price'].mean()

    print(f"\nPeak Hours (Weekdays HE 7-22):    ${peak_price:,.2f}/MWh")
    print(f"Off-Peak Hours:                   ${off_peak_price:,.2f}/MWh")
    print(f"Peak Premium:                     ${peak_price - off_peak_price:,.2f}/MWh ({(peak_price/off_peak_price - 1)*100:.1f}%)")

Volatility Analysis
~~~~~~~~~~~~~~~~~~~

.. code-block:: python

    # Calculate daily volatility
    daily_stats = df.groupby('deliveryDate')['price'].agg(['mean', 'std', 'min', 'max'])
    daily_stats['range'] = daily_stats['max'] - daily_stats['min']

    print("\nDaily Price Volatility:")
    print(f"  Average Daily Range: ${daily_stats['range'].mean():,.2f}/MWh")
    print(f"  Max Daily Range:     ${daily_stats['range'].max():,.2f}/MWh")
    print(f"  Average Std Dev:     ${daily_stats['std'].mean():,.2f}/MWh")

    # Find most volatile days
    most_volatile = daily_stats.nlargest(5, 'range')
    print("\nMost Volatile Days:")
    print(most_volatile[['mean', 'min', 'max', 'range']].round(2))

**Use Cases:**

* Historical price analysis
* Price forecasting
* Market trend identification
* Financial settlement validation
* Load serving entity (LSE) planning
* Hedging strategy development

---

CLI Usage
---------

All system operations data products can be accessed via the command-line interface:

.. code-block:: bash

    # System lambda (scarcity pricing)
    python isodart.py ercot system-operations --ops-type lambda --start 2025-08-15 --duration 1

    # Load vs forecast
    python isodart.py ercot system-operations --ops-type load_vs_forecast --start 2025-08-01 --duration 7

    # 60-day DAM prices for a specific settlement point
    python isodart.py ercot system-operations --ops-type dam_60d_prices --start 2025-07-01 --duration 60 --settlement-point HB_HOUSTON

---

Example Scripts
---------------

See ``examples/ercot_generation_analysis.py`` for scarcity pricing analysis examples:

.. code-block:: bash

    # Run scarcity analysis
    python examples/ercot_generation_analysis.py --date 2025-08-15 --analysis scarcity

---

Related Documentation
---------------------

* :doc:`pricing` - For real-time LMP data
* :doc:`load` - For load forecasting and actual load data
* :doc:`generation` - For generation mix and renewable data
* :doc:`api-guide` - General API usage patterns
