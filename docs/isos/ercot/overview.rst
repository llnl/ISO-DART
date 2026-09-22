ERCOT Data Guide
================

Overview
--------

The Electric Reliability Council of Texas (ERCOT) manages the flow of electric power to approximately 26 million Texas customers, representing about 90% of the state's electric load. ISO-DART provides comprehensive access to ERCOT's Public API, covering pricing, load, demand response, ancillary services, and market operations data.

Quick Reference
---------------

.. list-table::
   :header-rows: 1
   :widths: 25 20 25 30

   * - Data Category
     - Update Frequency
     - Historical Availability
     - Typical File Size
   * - LMP (DAM)
     - Daily
     - 2021-present
     - 20-40 MB/day
   * - LMP (SCED)
     - 5-min intervals
     - 2021-present
     - 100-200 MB/day
   * - LMP (RTD)
     - Real-time
     - 2021-present
     - 150-300 MB/day
   * - System Load
     - 5-min intervals
     - 2021-present
     - 5-10 MB/day
   * - Native Load
     - Hourly
     - Historical archives
     - Varies by year
   * - Demand Response
     - Monthly
     - 2014-present
     - <1 MB/month
   * - Ancillary Services
     - 2-day rolling
     - Recent data
     - 10-20 MB/2-days

ERCOT Markets
-------------

ERCOT operates a unique electricity market structure with several key components:

Day-Ahead Market (DAM)
~~~~~~~~~~~~~~~~~~~~~~

* **Purpose**: Schedule generation and establish hourly prices for next day
* **Timeline**: Bids due at 10 AM, results posted by 1:30 PM
* **Settlement**: Financial and physical
* **Price Formation**: Security-Constrained Economic Dispatch (SCED)

Real-Time Market (SCED/RTD)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

* **SCED**: Security-Constrained Economic Dispatch
* **Interval**: Every 5 minutes
* **Purpose**: Balance supply and demand in real-time
* **RTD**: Real-Time Dispatch (pricing)
* **Settlement**: Real-time energy imbalances

Ancillary Services Market
~~~~~~~~~~~~~~~~~~~~~~~~~~

ERCOT procures several types of ancillary services:

* **Regulation Up/Down (REGUP/REGDN)**: Frequency regulation
* **Responsive Reserve (RRS)**: Fast-responding emergency reserves

  * **RRSFFR**: Firm Fuel Response
  * **RRSPFR**: Primary Frequency Response
  * **RRSUFR**: Uninterruptible Fuel Response

* **Non-Spinning Reserve (NSPIN/NSPNM)**: 10 and 30-minute reserves
* **ECRS**: Emergency Condition Response Service

Load Zones
----------

ERCOT divides Texas into distinct load zones and weather zones:

**Four Primary Load Zones** (for Demand Response):

* **Houston**: Greater Houston metropolitan area
* **North**: North Texas including Dallas-Fort Worth
* **South**: South Texas including San Antonio and Corpus Christi
* **West**: West Texas

**Eight Weather Zones** (for Native Load):

* **COAST**: Coastal region
* **EAST**: East Texas
* **FWEST**: Far West Texas
* **NORTH**: North Central Texas
* **NCENT**: North Central
* **SOUTH**: South Texas
* **SCENT**: South Central
* **WEST**: West Texas

API Access
----------

ERCOT Public API Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To access ERCOT's Public API, you need:

1. **Subscription Key**: Register at https://developer.ercot.com/
2. **OAuth2 Credentials**: Create account at https://apiexplorer.ercot.com
3. **API Base URL**: ``https://api.ercot.com/api/public-reports``

Configuration
~~~~~~~~~~~~~

Create a ``user_config.ini`` file with your credentials:

.. code-block:: ini

   [ercot]
   # Subscription key from ERCOT Developer Portal
   api_key = your-ercot-subscription-key-here

   # OAuth2 credentials from API Explorer
   username = your-email@example.com
   password = your-password

   # Optional: customize settings
   data_dir = data/ERCOT
   max_retries = 3
   rate_limit_delay = 0.35
   default_page_size = 2000

Basic Usage
-----------

Initialize the Client
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   from lib.iso.ercot import ERCOTClient, ERCOTConfig
   from datetime import date, datetime

   # Load configuration from user_config.ini
   config = ERCOTConfig.from_ini_file()
   client = ERCOTClient(config)

Download Price Data
~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # DAM Hourly LMPs
   lmp_data = client.get_dam_hourly_lmps(
       delivery_date_from=date(2026, 1, 1),
       delivery_date_to=date(2026, 1, 31),
       bus_name="HB_HOUSTON"
   )

   # SCED LMPs (5-minute)
   sced_data = client.get_sced_lmps_node_zone_hub(
       start=datetime(2026, 1, 1, 0, 0, 0),
       end=datetime(2026, 1, 1, 23, 59, 59),
       settlement_point="HB_NORTH"
   )

Download Load Data
~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Actual system load by weather zone
   load_data = client.get_actual_system_load_by_weather_zone(
       operating_day_from=date(2026, 1, 1),
       operating_day_to=date(2026, 1, 31)
   )

   # Native load (historical, 8 weather zones)
   native_load = client.get_native_load(
       operating_day_from=date(2024, 10, 1),
       operating_day_to=date(2024, 10, 31)
   )

Save to CSV
~~~~~~~~~~~

.. code-block:: python

   # Save any report data to CSV
   csv_path = client.save_report_to_csv(
       lmp_data,
       "houston_lmps_jan2026.csv"
   )
   print(f"Saved to: {csv_path}")

   # Clean up when done
   client.cleanup()

Data Products
-------------

ERCOT publishes data through several "NP" (Node Protocol) numbered reports:

Price Data
~~~~~~~~~~

* **NP4-183-CD**: DAM Hourly LMPs
* **NP6-788-CD**: SCED LMPs (Node/Zone/Hub)
* **NP6-970-CD**: RTD LMPs (Node/Zone/Hub)
* **NP6-787-CD**: SCED LMPs (Electrical Bus)
* **NP6-905-CD**: Settlement Point Prices

Ancillary Services
~~~~~~~~~~~~~~~~~~

* **NP3-911-ER**: 2-Day DAM Ancillary Services (cleared & offers)
* **NP3-906-EX**: 2-Day SCED Ancillary Services (offers)
* **NP3-990-EX**: 60-Day SASM Ancillary Services (offers & awards)
* **NP6-328-CD**: Total AS Resource Capacity

Load & Generation
~~~~~~~~~~~~~~~~~

* **NP6-345-CD**: Actual System Load by Weather Zone
* **NP6-346-CD**: Actual System Load by Forecast Zone
* **NP3-965-ER**: 60-Day Load Resource Data in SCED
* **NP3-966-ER**: 60-Day DAM Load Resource Data
* **NP3-910-ER**: 2-Day Aggregated Load Data

Demand Response
~~~~~~~~~~~~~~~

* **NP3-108**: Monthly ERCOT Demand Response from Load Resources
* **NP3-107**: Monthly ERCOT Demand Response from ERS (limited availability)

See :doc:`demand-response` for detailed documentation.

Outages
~~~~~~~

* **NP3-233-CD**: Hourly Resource Outage Capacity

Common Use Cases
----------------

Price Analysis
~~~~~~~~~~~~~~

.. code-block:: python

   # Get DAM prices for all Houston hubs
   houston_prices = client.get_dam_hourly_lmps(
       delivery_date_from=date(2026, 1, 1),
       delivery_date_to=date(2026, 1, 31),
       bus_name="HB_HOUSTON"
   )

   # Calculate average prices
   import pandas as pd
   df = pd.DataFrame(houston_prices['data'])
   avg_price = df['lmp'].mean()
   print(f"Average LMP: ${avg_price:.2f}/MWh")

Load Forecasting
~~~~~~~~~~~~~~~~

.. code-block:: python

   # Get historical load patterns
   load_data = client.get_actual_system_load_by_weather_zone(
       operating_day_from=date(2025, 1, 1),
       operating_day_to=date(2025, 12, 31)
   )

   # Analyze peak demand
   df = pd.DataFrame(load_data['data'])
   peak_load = df.groupby('operatingDay')['total'].max()

Demand Response Analysis
~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Get monthly DR participation
   dr_data = client.get_monthly_demand_response(date(2026, 1, 1))

   # Analyze by load zone
   df = pd.DataFrame(dr_data['data'])
   zone_totals = df.groupby('resourceType')[['houston', 'north', 'south', 'west']].sum()
   print(zone_totals)

Rate Limits and Best Practices
-------------------------------

API Rate Limits
~~~~~~~~~~~~~~~

* Default rate limit: ~3 requests per second
* Configurable via ``rate_limit_delay`` in config
* Client automatically handles rate limiting

Pagination
~~~~~~~~~~

* Default page size: 2000 records
* Automatic pagination with ``fetch_all_pages=True`` (default)
* Large datasets may take several minutes

Token Management
~~~~~~~~~~~~~~~~

* OAuth2 tokens cached in ``~/.ercot/token.json``
* Automatic token refresh on expiration
* Tokens valid for ~1 hour by default

Best Practices
~~~~~~~~~~~~~~

1. **Use specific date ranges**: Narrow queries improve performance
2. **Filter at API level**: Use filter parameters when available
3. **Cache results**: Save to CSV/database to avoid re-downloading
4. **Handle 404s gracefully**: Not all reports have data for all dates
5. **Monitor for 429s**: Respect rate limits

Troubleshooting
---------------

Authentication Issues (401)
~~~~~~~~~~~~~~~~~~~~~~~~~~~

* Verify API key is correct in ``user_config.ini``
* Check username/password at https://apiexplorer.ercot.com
* Ensure token hasn't expired (cleared from cache)

Not Found (404)
~~~~~~~~~~~~~~~

* Report may not have data for requested date
* Check report ID spelling
* Verify date format (yyyy-MM-dd for dates, yyyy-MM-ddTHH:mm:ss for timestamps)

Rate Limited (429)
~~~~~~~~~~~~~~~~~~

* Increase ``rate_limit_delay`` in config
* Reduce concurrent requests
* Wait 60 seconds before retrying

No Data Returned
~~~~~~~~~~~~~~~~

* Check date range is valid
* Verify report is published for that period
* Some reports have limited historical data

Additional Resources
--------------------

* **ERCOT Public API**: https://developer.ercot.com/
* **API Explorer**: https://apiexplorer.ercot.com
* **Data Product Catalog**: https://www.ercot.com/mp/data-products
* **Market Information**: https://www.ercot.com/gridinfo/generation

Next Steps
----------

* :doc:`demand-response` - Comprehensive guide to Demand Response data
* :doc:`pricing` - Detailed pricing data documentation
* :doc:`load` - Load data products and analysis
* :doc:`api-guide` - Complete API reference
