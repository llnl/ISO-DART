Load Data
=========

ERCOT Load Products
-------------------

ERCOT publishes several load-related data products covering system-wide and zonal loads.

Actual System Load
------------------

NP6-345-CD: Actual System Load by Weather Zone
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

5-minute actual load data by weather zone.

.. code-block:: python

   from lib.iso.ercot import ERCOTClient, ERCOTConfig
   from datetime import date

   config = ERCOTConfig.from_ini_file()
   client = ERCOTClient(config)

   # Get load data for January 2026
   load_data = client.get_actual_system_load_by_weather_zone(
       operating_day_from=date(2026, 1, 1),
       operating_day_to=date(2026, 1, 31),
       dst_flag=False
   )

   # Save to CSV
   client.save_report_to_csv(load_data, "system_load_jan2026.csv")
   client.cleanup()

**Weather Zones:**

* COAST - Coastal region
* EAST - East Texas
* FWEST - Far West Texas
* NORTH - North Central Texas
* NCENT - North Central
* SOUTH - South Texas
* SCENT - South Central
* WEST - West Texas

NP6-346-CD: Actual System Load by Forecast Zone
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Load data organized by forecast zones.

.. code-block:: python

   load_data = client.get_actual_system_load_by_forecast_zone(
       operating_day_from=date(2026, 1, 1),
       operating_day_to=date(2026, 1, 31)
   )

Native Load (Historical)
------------------------

Historical hourly load data by weather zone from ERCOT's public archive.

.. code-block:: python

   # Get native load data
   native_load = client.get_native_load(
       operating_day_from=date(2024, 10, 1),
       operating_day_to=date(2024, 10, 31)
   )

**Data Columns:**

* ``operatingDay`` - Date
* ``hourEnding`` - Hour (01:00 to 24:00)
* ``coast``, ``east``, ``farWest``, ``north``, ``northC``, ``southern``, ``southC``, ``west`` - Zone loads (MW)
* ``total`` - System-wide total (MW)

.. note::

   Native load data is scraped from ERCOT's public website archives, not the API.
   Data is cached locally after first download.

Load Resource Data
------------------

NP3-965-ER: Load Resource Data in SCED
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

60-day rolling window of load resource data in SCED.

.. code-block:: python

   from datetime import datetime

   load_res_data = client.get_load_resource_data_in_sced(
       start=datetime(2026, 1, 1, 0, 0, 0),
       end=datetime(2026, 1, 31, 23, 59, 59)
   )

NP3-966-ER: DAM Load Resource Data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

60-day Day-Ahead Market load resource data.

.. code-block:: python

   dam_load_res = client.get_dam_load_resource_data(
       delivery_date_from=date(2026, 1, 1),
       delivery_date_to=date(2026, 1, 31)
   )

Aggregated Load Data
--------------------

NP3-910-ER: 2-Day Aggregated Load
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**DSR Loads:**

.. code-block:: python

   dsr_loads = client.get_dsr_loads_2day_aggregated(
       start=datetime(2026, 1, 1, 0, 0, 0),
       end=datetime(2026, 1, 2, 23, 59, 59)
   )

**Load Summary (All Regions):**

.. code-block:: python

   load_summary = client.get_load_summary_2day_aggregated(
       start=datetime(2026, 1, 1, 0, 0, 0),
       end=datetime(2026, 1, 2, 23, 59, 59)
   )

**Load Summary by Region:**

.. code-block:: python

   # Available regions: HOUSTON, NORTH, SOUTH, WEST
   houston_load = client.get_load_summary_2day_aggregated(
       start=datetime(2026, 1, 1, 0, 0, 0),
       end=datetime(2026, 1, 2, 23, 59, 59),
       region="HOUSTON"
   )

Common Analysis Tasks
---------------------

Calculate Peak Load
~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   import pandas as pd

   # Get load data
   load_data = client.get_actual_system_load_by_weather_zone(
       operating_day_from=date(2026, 1, 1),
       operating_day_to=date(2026, 1, 31)
   )

   # Find peak
   df = pd.DataFrame(load_data['data'])
   peak_load = df['total'].max()
   peak_row = df[df['total'] == peak_load].iloc[0]

   print(f"Peak Load: {peak_load:.0f} MW")
   print(f"Date: {peak_row['operatingDay']}")
   print(f"Hour: {peak_row['hourEnding']}")

Daily Load Profile
~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Calculate average hourly load
   df['hour'] = pd.to_datetime(df['hourEnding'], format='%H:%M').dt.hour
   hourly_avg = df.groupby('hour')['total'].mean()

   # Plot
   import matplotlib.pyplot as plt
   hourly_avg.plot(kind='line', title='Average Daily Load Profile')
   plt.xlabel('Hour of Day')
   plt.ylabel('Load (MW)')
   plt.show()

Compare Weather Zones
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Sum loads by zone
   zones = ['coast', 'east', 'farWest', 'north', 'northC', 'southern', 'southC', 'west']
   zone_totals = df[zones].sum()

   # Calculate percentages
   zone_pct = 100 * zone_totals / zone_totals.sum()

   print("Load Distribution by Weather Zone:")
   for zone, pct in zone_pct.items():
       print(f"{zone:10s}: {pct:5.1f}%")

Month-over-Month Growth
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Get two months of data
   jan_data = client.get_actual_system_load_by_weather_zone(
       operating_day_from=date(2026, 1, 1),
       operating_day_to=date(2026, 1, 31)
   )
   dec_data = client.get_actual_system_load_by_weather_zone(
       operating_day_from=date(2025, 12, 1),
       operating_day_to=date(2025, 12, 31)
   )

   # Calculate averages
   jan_avg = pd.DataFrame(jan_data['data'])['total'].mean()
   dec_avg = pd.DataFrame(dec_data['data'])['total'].mean()

   growth = 100 * (jan_avg - dec_avg) / dec_avg
   print(f"Month-over-month load growth: {growth:+.1f}%")

See Also
--------

* :doc:`overview` - ERCOT overview and configuration
* :doc:`demand-response` - Demand response data
* :doc:`api-guide` - Complete API reference
