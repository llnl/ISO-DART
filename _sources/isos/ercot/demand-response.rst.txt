Demand Response Data
====================

Overview
--------

ISO-DART supports downloading ERCOT Demand Response (DR) data through two monthly report products:

1. **NP3-108**: Monthly ERCOT Demand Response from Load Resources ✅ **Available**
2. **NP3-107**: Monthly ERCOT Demand Response from ERS ⚠️ **Limited Availability**

These reports provide insights into:

* Geographic distribution of DR across ERCOT load zones
* Temporal patterns of DR activation (months, days, hours)
* Breakdown by resource type and ancillary service participation
* Total DR capacity and activation frequency

NP3-108: Demand Response from Load Resources
---------------------------------------------

What It Contains
~~~~~~~~~~~~~~~~

**Coverage:**

* Emergency Response Service (ERS) participation
* Ancillary Services as a Load Resource
* Pilot projects permitted by P.U.C. Subst. R. 25.361

**Granularity:**

* Monthly data with hourly breakdown
* Four load zones: Houston, North, South, West
* Resource type classification:

  * **CLR**: Controllable Load Resources
  * **NCLR**: Non-Controllable Load Resources

* Ancillary Service type breakdown (ECRS, NSPIN, RRS, etc.)

Data Structure
~~~~~~~~~~~~~~

.. code-block:: python

   {
       "_meta": {
           "totalRecords": 124,
           "totalPages": 1,
           "currentPage": 1,
           "pageSize": 124
       },
       "report": "np3-108",
       "fields": [
           {"name": "month", "dataType": "VARCHAR"},
           {"name": "hour", "dataType": "INTEGER"},
           {"name": "asType", "dataType": "VARCHAR"},
           {"name": "houston", "dataType": "DOUBLE"},
           {"name": "north", "dataType": "DOUBLE"},
           {"name": "south", "dataType": "DOUBLE"},
           {"name": "west", "dataType": "DOUBLE"},
           {"name": "resourceType", "dataType": "VARCHAR"}
       ],
       "data": [
           {
               "month": "JAN-26",
               "hour": 1,
               "asType": "ECRS",
               "houston": 21.0,
               "north": 15.0,
               "south": 7.0,
               "west": 0.0,
               "resourceType": "CLR"
           }
       ]
   }

Basic Usage
~~~~~~~~~~~

.. code-block:: python

   from lib.iso.ercot import ERCOTClient, ERCOTConfig
   from datetime import date

   # Initialize client
   config = ERCOTConfig.from_ini_file()
   client = ERCOTClient(config)

   # Download January 2026 data
   dr_data = client.get_monthly_demand_response(date(2026, 1, 1))

   # Access the data
   for row in dr_data['data']:
       print(f"{row['month']} Hour {row['hour']}")
       print(f"  AS Type: {row['asType']}")
       print(f"  Resource Type: {row['resourceType']}")
       print(f"  Houston: {row['houston']} MW")
       print(f"  North: {row['north']} MW")
       print(f"  South: {row['south']} MW")
       print(f"  West: {row['west']} MW")

   # Save to CSV
   client.save_report_to_csv(dr_data, "demand_response_jan_2026.csv")
   client.cleanup()

Analysis Examples
~~~~~~~~~~~~~~~~~

**Geographic Distribution**

Analyze DR capacity by load zone:

.. code-block:: python

   zones = ['houston', 'north', 'south', 'west']
   for zone in zones:
       zone_total = sum(r.get(zone, 0) or 0 for r in dr_data['data'])
       print(f"{zone.capitalize()}: {zone_total:.1f} MW-hours")

**Output:**

.. code-block:: text

   Houston: 171232.0 MW-hours
   North: 44910.0 MW-hours
   South: 147797.0 MW-hours
   West: 83324.0 MW-hours

**Ancillary Service Breakdown**

Count activations and total capacity by AS type:

.. code-block:: python

   import pandas as pd

   df = pd.DataFrame(dr_data['data'])

   # Total MW-hours by AS type
   as_summary = df.groupby('asType')[['houston', 'north', 'south', 'west']].sum()
   as_summary['total'] = as_summary.sum(axis=1)

   print(as_summary)

**Output:**

.. code-block:: text

   asType  houston    north    south     west     total
   ECRS     15000.0   9800.0   12400.0   5207.0   42407.0
   NSPIN    28000.0  15100.0   18800.0  10730.0   72630.0
   RRS     128232.0  20010.0  116597.0  67387.0  332226.0

**Resource Type Comparison**

Compare CLR vs NCLR participation:

.. code-block:: python

   clr_rows = [r for r in dr_data['data'] if r['resourceType'] == 'CLR']
   nclr_rows = [r for r in dr_data['data'] if r['resourceType'] == 'NCLR']

   print(f"Controllable Load Resources: {len(clr_rows)} hourly entries")
   print(f"Non-Controllable Load Resources: {len(nclr_rows)} hourly entries")

   # Total capacity by type
   zones = ['houston', 'north', 'south', 'west']
   clr_total = sum(sum(r.get(z, 0) or 0 for z in zones) for r in clr_rows)
   nclr_total = sum(sum(r.get(z, 0) or 0 for z in zones) for r in nclr_rows)

   print(f"CLR Total: {clr_total:.1f} MW-hours")
   print(f"NCLR Total: {nclr_total:.1f} MW-hours")

**DR Activation Count**

Count distinct DR activation events:

.. code-block:: python

   # Define activation as any row with non-zero MW
   zones = ['houston', 'north', 'south', 'west']
   activations = [
       r for r in dr_data['data']
       if any((r.get(zone, 0) or 0) > 0 for zone in zones)
   ]

   print(f"Total DR activations: {len(activations)} hour-events")

   # Group by day
   df = pd.DataFrame(activations)
   daily_activations = df.groupby(['month']).size()
   print(f"\nActivations per day:\n{daily_activations}")

**Peak DR Hours**

Identify peak DR usage periods:

.. code-block:: python

   import pandas as pd

   df = pd.DataFrame(dr_data['data'])
   df['total_mw'] = df[['houston', 'north', 'south', 'west']].sum(axis=1)

   # Peak hours
   peak_hours = df.groupby('hour')['total_mw'].sum().sort_values(ascending=False)
   print("Top 5 DR hours:")
   print(peak_hours.head())

NP3-107: Demand Response from ERS
----------------------------------

What It Contains
~~~~~~~~~~~~~~~~

**Coverage:**

* MWs of Demand Response participating specifically in Emergency Response Service (ERS)
* Monthly data by load zone

**Granularity:**

* Monthly data with hourly breakdown
* Load zones (typically Houston, North, South, West, and potentially additional zones)

Current Status
~~~~~~~~~~~~~~

.. warning::

   NP3-107 is **NOT currently available** through the ERCOT Public API's archive endpoint.

The implementation is ready and will work automatically once ERCOT makes this report available through their Public API.

Usage Options
~~~~~~~~~~~~~

**Option 1: Try the API Method (Recommended)**

.. code-block:: python

   from lib.iso.ercot import ERCOTClient, ERCOTConfig
   from datetime import date

   config = ERCOTConfig.from_ini_file()
   client = ERCOTClient(config)

   # Attempt to download
   ers_data = client.get_monthly_demand_response_ers(date(2026, 1, 1))

   if ers_data:
       # Data available!
       print(f"Retrieved {len(ers_data['data'])} rows")
       client.save_report_to_csv(ers_data, "ers_dr_jan_2026.csv")
   else:
       # Not available yet
       print("NP3-107 not accessible through API")

   client.cleanup()

**Option 2: Manual Download & Parse**

If you manually download the Excel file from ERCOT:

.. code-block:: python

   from lib.iso.ercot import ERCOTClient, ERCOTConfig

   config = ERCOTConfig.from_ini_file()
   client = ERCOTClient(config)

   # Parse manually downloaded file
   with open('downloaded_np3_107.xlsx', 'rb') as f:
       content = f.read()
       rows = client._parse_ers_demand_response_xlsx(content)
       payload = client._format_ers_demand_response_payload(rows)

       # Save to CSV
       client.save_report_to_csv(payload, "np3_107_data.csv")

   client.cleanup()

**Option 3: MIS Portal Access**

If you discover the ERCOT MIS portal reportTypeId for NP3-107:

.. code-block:: python

   ers_data = client.get_monthly_demand_response_ers(
       date(2026, 1, 1),
       report_type_id=YOUR_REPORT_TYPE_ID  # Replace with actual ID
   )

Manual Download Location
~~~~~~~~~~~~~~~~~~~~~~~~

Visit: https://www.ercot.com/mp/data-products/data-product-details?id=np3-107

Expected Data Structure
~~~~~~~~~~~~~~~~~~~~~~~

When available, NP3-107 will return:

.. code-block:: python

   {
       "_meta": {
           "totalRecords": int,
           "totalPages": 1,
           "currentPage": 1,
           "pageSize": int
       },
       "report": "np3-107",
       "fields": [
           {"name": "month", "dataType": "VARCHAR"},
           {"name": "hour", "dataType": "INTEGER"},
           {"name": "houston", "dataType": "DOUBLE"},
           {"name": "north", "dataType": "DOUBLE"},
           {"name": "south", "dataType": "DOUBLE"},
           {"name": "west", "dataType": "DOUBLE"}
           # Additional load zones may be present
       ],
       "data": [
           {
               "month": "JAN-26",
               "hour": 1,
               "houston": 100.0,
               "north": 50.0,
               "south": 75.0,
               "west": 25.0
           }
       ]
   }

Comparison: NP3-108 vs NP3-107
-------------------------------

.. list-table::
   :header-rows: 1
   :widths: 25 35 40

   * - Feature
     - NP3-108 (Load Resources)
     - NP3-107 (ERS)
   * - **Availability**
     - ✅ Public API
     - ⚠️ Limited (Manual/MIS Portal)
   * - **Coverage**
     - All DR (ERS, AS, Pilots)
     - ERS only
   * - **Resource Types**
     - CLR and NCLR
     - Combined
   * - **AS Type Breakdown**
     - Yes (ECRS, NSPIN, etc.)
     - No
   * - **Load Zones**
     - Houston, North, South, West
     - Houston, North, South, West (+others)
   * - **Granularity**
     - Hourly by AS type
     - Hourly total
   * - **Best For**
     - Detailed AS analysis
     - Simple ERS totals

Recommendations
~~~~~~~~~~~~~~~

**For Most Use Cases:**
  Use **NP3-108** - it's currently available and provides more detailed information including ERS data.

**When NP3-107 is Needed:**
  For specific ERS-only analysis without AS type breakdown, use NP3-107 when it becomes available via the API.

Use Cases
---------

Geographic Distribution Analysis
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Question:** How is DR capacity distributed across ERCOT load zones?

.. code-block:: python

   import pandas as pd
   import matplotlib.pyplot as plt

   # Get data
   dr_data = client.get_monthly_demand_response(date(2026, 1, 1))
   df = pd.DataFrame(dr_data['data'])

   # Calculate zone totals
   zone_totals = df[['houston', 'north', 'south', 'west']].sum()

   # Plot
   zone_totals.plot(kind='bar', title='DR Capacity by Load Zone')
   plt.ylabel('MW-hours')
   plt.xlabel('Load Zone')
   plt.show()

Temporal Activation Patterns
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

**Question:** When is DR most frequently activated?

.. code-block:: python

   # Calculate hourly activation frequency
   hourly_activations = df.groupby('hour').size()

   # Plot daily pattern
   hourly_activations.plot(
       kind='line',
       title='DR Activation Frequency by Hour of Day',
       xlabel='Hour',
       ylabel='Number of Activations'
   )
   plt.show()

   # Peak hours
   print("Top 5 peak DR hours:")
   print(hourly_activations.sort_values(ascending=False).head())

Annual Activation Count
~~~~~~~~~~~~~~~~~~~~~~~

**Question:** How many DR activations occurred in a year?

.. code-block:: python

   from datetime import date
   import pandas as pd

   # Download all 12 months
   all_data = []
   for month in range(1, 13):
       dr_data = client.get_monthly_demand_response(date(2025, month, 1))
       if dr_data:
           all_data.extend(dr_data['data'])

   # Count activations (non-zero MW)
   zones = ['houston', 'north', 'south', 'west']
   activations = [
       r for r in all_data
       if any((r.get(zone, 0) or 0) > 0 for zone in zones)
   ]

   print(f"Total 2025 DR activations: {len(activations)} hour-events")

   # Monthly breakdown
   df = pd.DataFrame(activations)
   monthly_counts = df.groupby('month').size()
   print("\nMonthly activation counts:")
   print(monthly_counts)

Resource Type Performance
~~~~~~~~~~~~~~~~~~~~~~~~~

**Question:** Which resource type (CLR vs NCLR) provides more DR?

.. code-block:: python

   # Separate by resource type
   clr_data = df[df['resourceType'] == 'CLR']
   nclr_data = df[df['resourceType'] == 'NCLR']

   # Calculate totals
   zones = ['houston', 'north', 'south', 'west']
   clr_total = clr_data[zones].sum().sum()
   nclr_total = nclr_data[zones].sum().sum()

   print(f"CLR Total: {clr_total:,.0f} MW-hours")
   print(f"NCLR Total: {nclr_total:,.0f} MW-hours")
   print(f"CLR Percentage: {100 * clr_total / (clr_total + nclr_total):.1f}%")

AS Type Utilization
~~~~~~~~~~~~~~~~~~~

**Question:** Which ancillary services use DR most frequently?

.. code-block:: python

   # Count by AS type
   as_counts = df.groupby('asType').size().sort_values(ascending=False)

   # Total MW-hours by AS type
   as_mw = df.groupby('asType')[zones].sum()
   as_mw['total'] = as_mw.sum(axis=1)
   as_mw = as_mw.sort_values('total', ascending=False)

   print("DR Activations by AS Type:")
   print(as_counts)
   print("\nDR Capacity by AS Type (MW-hours):")
   print(as_mw['total'])

Complete Example
----------------

Here's a complete script that downloads and analyzes DR data:

.. code-block:: python

   from lib.iso.ercot import ERCOTClient, ERCOTConfig
   from datetime import date
   import pandas as pd

   def analyze_demand_response(month: date):
       """Download and analyze ERCOT demand response data."""

       # Initialize client
       config = ERCOTConfig.from_ini_file()
       client = ERCOTClient(config)

       # Download data
       print(f"Downloading DR data for {month.strftime('%Y-%m')}...")
       dr_data = client.get_monthly_demand_response(month)

       if not dr_data:
           print("No data available")
           return

       # Convert to DataFrame
       df = pd.DataFrame(dr_data['data'])
       zones = ['houston', 'north', 'south', 'west']

       # Analysis 1: Geographic distribution
       print("\n=== Geographic Distribution ===")
       zone_totals = df[zones].sum()
       for zone, total in zone_totals.items():
           pct = 100 * total / zone_totals.sum()
           print(f"{zone.capitalize():8s}: {total:10,.0f} MW-hours ({pct:5.1f}%)")

       # Analysis 2: Resource types
       print("\n=== Resource Types ===")
       for rtype in ['CLR', 'NCLR']:
           subset = df[df['resourceType'] == rtype]
           total = subset[zones].sum().sum()
           count = len(subset)
           print(f"{rtype:5s}: {count:3d} entries, {total:10,.0f} MW-hours")

       # Analysis 3: AS types
       print("\n=== Ancillary Service Types ===")
       as_summary = df.groupby('asType')[zones].sum()
       as_summary['total'] = as_summary.sum(axis=1)
       as_summary = as_summary.sort_values('total', ascending=False)
       for as_type, row in as_summary.iterrows():
           print(f"{as_type:8s}: {row['total']:10,.0f} MW-hours")

       # Analysis 4: Peak hours
       print("\n=== Peak DR Hours ===")
       df['total_mw'] = df[zones].sum(axis=1)
       hourly = df.groupby('hour')['total_mw'].sum().sort_values(ascending=False)
       for hour, mw in hourly.head(5).items():
           print(f"Hour {hour:2d}: {mw:10,.0f} MW")

       # Save to CSV
       csv_path = client.save_report_to_csv(
           dr_data,
           f"demand_response_{month.strftime('%Y_%m')}.csv"
       )
       print(f"\nSaved to: {csv_path}")

       client.cleanup()

   if __name__ == "__main__":
       analyze_demand_response(date(2026, 1, 1))

See Also
--------

* :doc:`overview` - General ERCOT documentation
* :doc:`pricing` - Price data documentation
* :doc:`load` - Load data documentation
* :doc:`api-guide` - Complete API reference

External Links
~~~~~~~~~~~~~~

* `NP3-108 Data Product <https://www.ercot.com/mp/data-products/data-product-details?id=NP3-108>`_
* `NP3-107 Data Product <https://www.ercot.com/mp/data-products/data-product-details?id=np3-107>`_
* `ERCOT Market Information <https://www.ercot.com/mp/data-products>`_
