Pricing Data
============

ERCOT Price Products
--------------------

ERCOT publishes several price-related data products through its Public API.

Day-Ahead Market (DAM) Prices
------------------------------

NP4-183-CD: DAM Hourly LMPs
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Hourly Locational Marginal Prices for the Day-Ahead Market.

.. code-block:: python

   from lib.iso.ercot import ERCOTClient, ERCOTConfig
   from datetime import date

   config = ERCOTConfig.from_ini_file()
   client = ERCOTClient(config)

   # Get DAM LMPs for January 2026
   lmp_data = client.get_dam_hourly_lmps(
       delivery_date_from=date(2026, 1, 1),
       delivery_date_to=date(2026, 1, 31),
       bus_name="HB_HOUSTON"
   )

   # Save to CSV
   client.save_report_to_csv(lmp_data, "dam_lmps_houston_jan2026.csv")
   client.cleanup()

**Filters:**

* ``hour_ending``: Filter by specific hour (1-24)
* ``bus_name``: Filter by specific bus/settlement point
* ``lmp_from`` / ``lmp_to``: Price range filters
* ``dst_flag``: Daylight Saving Time flag

Real-Time Market Prices
-----------------------

NP6-788-CD: SCED LMPs (Node/Zone/Hub)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

5-minute Security-Constrained Economic Dispatch LMPs.

.. code-block:: python

   from datetime import datetime

   # Get SCED LMPs for a specific day
   sced_data = client.get_sced_lmps_node_zone_hub(
       start=datetime(2026, 1, 1, 0, 0, 0),
       end=datetime(2026, 1, 1, 23, 59, 59),
       settlement_point="HB_NORTH"
   )

**Filters:**

* ``settlement_point``: Specific node, zone, or hub
* ``lmp_from`` / ``lmp_to``: Price range
* ``repeat_hour_flag``: Include/exclude DST repeat hours

NP6-970-CD: RTD LMPs (Node/Zone/Hub)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Real-Time Dispatch LMPs.

.. code-block:: python

   rtd_data = client.get_rtd_lmps_node_zone_hub(
       start=datetime(2026, 1, 1, 0, 0, 0),
       end=datetime(2026, 1, 1, 23, 59, 59),
       settlement_point="HB_SOUTH",
       settlement_point_type="Hub"
   )

NP6-787-CD: SCED LMPs (Electrical Bus)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

SCED LMPs at electrical bus level.

.. code-block:: python

   bus_data = client.get_sced_lmps_electrical_bus(
       start=datetime(2026, 1, 1, 0, 0, 0),
       end=datetime(2026, 1, 1, 23, 59, 59),
       electrical_bus="BUS_ABC"
   )

Settlement Point Prices
-----------------------

NP6-905-CD: Settlement Point Prices
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Settlement point prices by node, zone, and hub.

.. code-block:: python

   spp_data = client.get_settlement_point_prices(
       delivery_date_from=date(2026, 1, 1),
       delivery_date_to=date(2026, 1, 31),
       settlement_point="HB_NORTH",
       settlement_point_type="Hub"
   )

**Filters:**

* ``delivery_hour_from`` / ``delivery_hour_to``: Hour range (1-24)
* ``delivery_interval_from`` / ``delivery_interval_to``: 15-minute interval (1-4)
* ``spp_from`` / ``spp_to``: Price range
* ``dst_flag``: DST flag

Common Analysis Tasks
---------------------

Calculate Average Prices
~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   import pandas as pd

   # Get data
   lmp_data = client.get_dam_hourly_lmps(
       delivery_date_from=date(2026, 1, 1),
       delivery_date_to=date(2026, 1, 31)
   )

   # Convert to DataFrame and calculate average
   df = pd.DataFrame(lmp_data['data'])
   avg_price = df['lmp'].mean()
   print(f"Average January LMP: ${avg_price:.2f}/MWh")

Find Peak Price Hours
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Sort by price
   df_sorted = df.sort_values('lmp', ascending=False)
   print("Top 5 highest LMP hours:")
   print(df_sorted[['deliveryDate', 'hourEnding', 'busName', 'lmp']].head())

Compare Hub Prices
~~~~~~~~~~~~~~~~~~

.. code-block:: python

   hubs = ['HB_NORTH', 'HB_SOUTH', 'HB_HOUSTON', 'HB_WEST']
   hub_prices = {}

   for hub in hubs:
       data = client.get_dam_hourly_lmps(
           delivery_date_from=date(2026, 1, 1),
           delivery_date_to=date(2026, 1, 31),
           bus_name=hub
       )
       df = pd.DataFrame(data['data'])
       hub_prices[hub] = df['lmp'].mean()

   print("Average Hub Prices:")
   for hub, price in hub_prices.items():
       print(f"{hub}: ${price:.2f}/MWh")

See Also
--------

* :doc:`overview` - ERCOT overview and configuration
* :doc:`api-guide` - Complete API reference
