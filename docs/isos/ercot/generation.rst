Generation Data
===============

This page documents the generation and renewable energy data products available through the ERCOT API.

Wind Power Production
---------------------

**Report:** NP4-732-CD
**Method:** ``get_wind_power_production()``
**Update Frequency:** Hourly

Hourly-averaged wind generation actuals across ERCOT.

.. code-block:: python

    from datetime import date
    from lib.iso.ercot import ERCOTClient, ERCOTConfig

    config = ERCOTConfig.from_ini_file()
    client = ERCOTClient(config)

    # Get wind production for August 2025
    wind_data = client.get_wind_power_production(
        delivery_date_from=date(2025, 8, 1),
        delivery_date_to=date(2025, 8, 31)
    )

    # Analyze peak production
    import pandas as pd
    df = pd.DataFrame(wind_data['data'])
    print(f"Peak Wind: {df['generation'].max():,.0f} MW")
    print(f"Average: {df['generation'].mean():,.0f} MW")

    client.cleanup()

**Data Fields:**

* ``deliveryDate`` - Date of generation
* ``hourEnding`` - Hour ending (1-24)
* ``generation`` - Wind generation in MW

**Use Cases:**

* Renewable energy forecasting
* Wind capacity factor analysis
* Grid integration studies
* Renewable portfolio standard (RPS) compliance

---

Solar Power Production
----------------------

**Report:** NP4-737-CD
**Method:** ``get_solar_power_production()``
**Update Frequency:** Hourly

Hourly-averaged solar generation actuals across ERCOT.

.. code-block:: python

    # Get solar production for August 2025
    solar_data = client.get_solar_power_production(
        delivery_date_from=date(2025, 8, 1),
        delivery_date_to=date(2025, 8, 31)
    )

    # Find peak solar hours
    df = pd.DataFrame(solar_data['data'])
    peak_hours = df.nlargest(10, 'generation')
    print("Top 10 Solar Hours:")
    print(peak_hours[['deliveryDate', 'hourEnding', 'generation']])

**Data Fields:**

* ``deliveryDate`` - Date of generation
* ``hourEnding`` - Hour ending (1-24)
* ``generation`` - Solar generation in MW

**Use Cases:**

* Solar generation analysis
* Capacity factor calculations
* Peak solar hours identification
* Renewable portfolio tracking

---

Fuel Mix
--------

**Report:** NP6-785-ER
**Method:** ``get_fuel_mix()``
**Update Frequency:** 5-minute intervals

Real-time generation by fuel type including Natural Gas, Coal, Nuclear, Wind, Solar, Hydro, and Other.

.. code-block:: python

    # Get fuel mix for a specific day
    fuel_data = client.get_fuel_mix(
        delivery_date_from=date(2025, 8, 15),
        delivery_date_to=date(2025, 8, 15)
    )

    # Aggregate by fuel type
    df = pd.DataFrame(fuel_data['data'])
    by_fuel = df.groupby('fuelType').agg({
        'generation': ['mean', 'min', 'max', 'sum']
    }).round(0)

    print("\nFuel Mix Summary:")
    print(by_fuel)

    # Calculate percentages
    total_gen = df.groupby('fuelType')['generation'].sum()
    pct = (total_gen / total_gen.sum() * 100).round(1)
    print("\nGeneration Share:")
    print(pct.sort_values(ascending=False))

**Data Fields:**

* ``timestamp`` - 5-minute SCED timestamp
* ``fuelType`` - Fuel category (Natural Gas, Coal, Nuclear, Wind, Solar, Hydro, Other)
* ``generation`` - Generation in MW

**Fuel Types:**

* **Natural Gas** - Combined cycle and combustion turbine
* **Coal** - Coal-fired generation
* **Nuclear** - Nuclear generation
* **Wind** - Wind generation
* **Solar** - Solar PV and thermal
* **Hydro** - Hydroelectric
* **Other** - Biomass, waste, storage discharge, etc.

**Use Cases:**

* Emissions analysis
* Fuel diversity studies
* Carbon intensity calculations
* Energy mix trends
* Environmental reporting

---

Unplanned Resource Outages
---------------------------

**Report:** NP3-233-CD
**Method:** ``get_unplanned_resource_outages()``
**Update Frequency:** Real-time (every 5 minutes)

Real-time tracking of unplanned (forced) generation resource outages.

.. code-block:: python

    from datetime import datetime

    # Get unplanned outages for a specific day
    start_dt = datetime(2025, 8, 15, 0, 0, 0)
    end_dt = datetime(2025, 8, 15, 23, 59, 59)

    outages = client.get_unplanned_resource_outages(
        sced_timestamp_from=start_dt,
        sced_timestamp_to=end_dt
    )

    # Aggregate by fuel type
    df = pd.DataFrame(outages['data'])
    if not df.empty:
        by_fuel = df.groupby('fuelType')['outageMW'].agg(['count', 'sum', 'mean'])
        print("\nUnplanned Outages by Fuel Type:")
        print(by_fuel)

        print(f"\nTotal Outage MW: {df['outageMW'].sum():,.0f} MW")
    else:
        print("No unplanned outages during this period")

**Data Fields:**

* ``scedTimestamp`` - SCED timestamp
* ``resourceName`` - Resource identifier
* ``fuelType`` - Fuel category
* ``outageMW`` - Outage capacity in MW
* ``outageType`` - Type of outage (Forced, etc.)

**Use Cases:**

* Reliability analysis
* Generation fleet health monitoring
* Risk assessment
* Forced outage rate (FOR) calculations
* Capacity planning

---

System Wide Actual Load vs Forecast
------------------------------------

**Report:** NP6-346-CD
**Method:** ``get_system_wide_actual_load_vs_forecast()``
**Update Frequency:** Hourly

Comparison of actual system load versus forecasted load for accuracy evaluation.

.. code-block:: python

    # Get load vs forecast for a week
    load_forecast_data = client.get_system_wide_actual_load_vs_forecast(
        delivery_date_from=date(2025, 8, 1),
        delivery_date_to=date(2025, 8, 7)
    )

    # Calculate forecast accuracy
    df = pd.DataFrame(load_forecast_data['data'])
    df['error_MW'] = df['actualLoad'] - df['forecastLoad']
    df['error_pct'] = abs(df['error_MW'] / df['actualLoad'] * 100)

    print("\nForecast Accuracy Metrics:")
    print(f"MAPE: {df['error_pct'].mean():.2f}%")
    print(f"Max Error: {df['error_MW'].abs().max():,.0f} MW")
    print(f"Mean Bias: {df['error_MW'].mean():,.0f} MW")

    # Find largest forecast misses
    worst = df.nlargest(5, 'error_pct')[['deliveryDate', 'hourEnding', 'actualLoad', 'forecastLoad', 'error_pct']]
    print("\nLargest Forecast Errors:")
    print(worst)

**Data Fields:**

* ``deliveryDate`` - Date of load
* ``hourEnding`` - Hour ending (1-24)
* ``actualLoad`` - Actual system load in MW
* ``forecastLoad`` - Forecasted load in MW
* ``dstFlag`` - Daylight saving time indicator

**Use Cases:**

* Forecast accuracy evaluation
* Load prediction model validation
* Real-time vs forecast deviation analysis
* Operational planning assessment
* Forecasting model improvement

---

CLI Usage
---------

All generation data products can be accessed via the command-line interface:

.. code-block:: bash

    # Wind power production
    python isodart.py ercot generation --gen-type wind --start 2025-08-01 --duration 30

    # Solar power production
    python isodart.py ercot generation --gen-type solar --start 2025-08-01 --duration 30

    # Fuel mix (5-minute data)
    python isodart.py ercot generation --gen-type fuel_mix --start 2025-08-15 --duration 1

    # Unplanned outages
    python isodart.py ercot generation --gen-type unplanned_outages --start 2025-08-15 --duration 1

---

Example Scripts
---------------

See ``examples/ercot_generation_analysis.py`` for a comprehensive example demonstrating:

* Generation mix analysis
* Renewable energy trends
* Outage tracking
* Scarcity pricing correlation

.. code-block:: bash

    # Run all analyses
    python examples/ercot_generation_analysis.py --date 2025-08-15

    # Run specific analysis
    python examples/ercot_generation_analysis.py --date 2025-08-15 --analysis renewables
    python examples/ercot_generation_analysis.py --date 2025-08-15 --analysis mix
