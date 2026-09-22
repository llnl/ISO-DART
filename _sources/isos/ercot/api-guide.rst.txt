API Reference
=============

ERCOTClient
-----------

The main client class for interacting with ERCOT's Public API.

Configuration
~~~~~~~~~~~~~

ERCOTConfig
^^^^^^^^^^^

.. code-block:: python

   from lib.iso.ercot import ERCOTConfig

   # Load from INI file
   config = ERCOTConfig.from_ini_file()

   # Or create manually
   config = ERCOTConfig(
       api_key="your-key-here",
       username="your-email@example.com",
       password="your-password",
       data_dir=Path("data/ERCOT"),
       max_retries=3,
       rate_limit_delay=0.35,
       default_page_size=2000
   )

**Parameters:**

* ``api_key`` (str): ERCOT subscription key
* ``username`` (str): OAuth2 username
* ``password`` (str): OAuth2 password
* ``base_url`` (str): API base URL (default: https://api.ercot.com/api/public-reports)
* ``data_dir`` (Path): Directory for saving data (default: data/ERCOT)
* ``max_retries`` (int): Maximum retry attempts (default: 3)
* ``retry_delay`` (int): Seconds between retries (default: 5)
* ``timeout`` (int): Request timeout in seconds (default: 30)
* ``rate_limit_delay`` (float): Delay between requests (default: 0.35)
* ``default_page_size`` (int): Default page size (default: 2000)

Client Initialization
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   from lib.iso.ercot import ERCOTClient, ERCOTConfig

   config = ERCOTConfig.from_ini_file()
   client = ERCOTClient(config)

   # ... use client ...

   client.cleanup()  # Always cleanup when done

Core Methods
------------

Generic Report Access
~~~~~~~~~~~~~~~~~~~~~

get_report()
^^^^^^^^^^^^

Fetch any ERCOT report endpoint.

.. code-block:: python

   payload = client.get_report(
       report_path="np6-788-cd/lmp_node_zone_hub",
       params={"settlementPoint": "HB_NORTH"},
       fetch_all_pages=True,
       page=1,
       size=2000
   )

**Parameters:**

* ``report_path`` (str): Report endpoint path
* ``params`` (dict, optional): Query parameters
* ``fetch_all_pages`` (bool): Auto-paginate (default: True)
* ``page`` (int, optional): Specific page number
* ``size`` (int, optional): Page size

**Returns:** Report payload dict or None

get_report_data_only()
^^^^^^^^^^^^^^^^^^^^^^^

Convenience wrapper that returns only the data list.

.. code-block:: python

   data = client.get_report_data_only(
       report_path="np6-788-cd/lmp_node_zone_hub",
       params={"settlementPoint": "HB_NORTH"}
   )

**Returns:** List of data rows or empty list

get_report_by_timerange()
^^^^^^^^^^^^^^^^^^^^^^^^^^

Helper for reports with From/To date parameters.

.. code-block:: python

   payload = client.get_report_by_timerange(
       report_path="np6-788-cd/lmp_node_zone_hub",
       from_param="SCEDTimestampFrom",
       to_param="SCEDTimestampTo",
       start="2025-01-01T00:00:00",
       end="2025-01-02T00:00:00",
       params={"settlementPoint": "HB_NORTH"},
       param_format="timestamp"
   )

**Parameters:**

* ``from_param`` (str): Name of "from" parameter
* ``to_param`` (str): Name of "to" parameter
* ``start`` (DateLike): Start date/datetime
* ``end`` (DateLike): End date/datetime
* ``params`` (dict, optional): Additional parameters
* ``param_format`` (str): "date", "timestamp", or "auto" (default: "auto")

Price Methods
-------------

See :doc:`pricing` for detailed pricing documentation.

get_dam_hourly_lmps()
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   lmp_data = client.get_dam_hourly_lmps(
       delivery_date_from=date(2026, 1, 1),
       delivery_date_to=date(2026, 1, 31),
       hour_ending=None,
       bus_name=None,
       lmp_from=None,
       lmp_to=None,
       dst_flag=None
   )

get_sced_lmps_node_zone_hub()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   sced_data = client.get_sced_lmps_node_zone_hub(
       start=datetime(2026, 1, 1, 0, 0, 0),
       end=datetime(2026, 1, 1, 23, 59, 59),
       settlement_point=None,
       lmp_from=None,
       lmp_to=None,
       repeat_hour_flag=None
   )

get_rtd_lmps_node_zone_hub()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   rtd_data = client.get_rtd_lmps_node_zone_hub(
       start=datetime(2026, 1, 1, 0, 0, 0),
       end=datetime(2026, 1, 1, 23, 59, 59),
       settlement_point=None,
       settlement_point_type=None,
       lmp_from=None,
       lmp_to=None,
       repeat_hour_flag=None
   )

get_settlement_point_prices()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   spp_data = client.get_settlement_point_prices(
       delivery_date_from=date(2026, 1, 1),
       delivery_date_to=date(2026, 1, 31),
       settlement_point=None,
       settlement_point_type=None,
       delivery_hour_from=None,
       delivery_hour_to=None,
       spp_from=None,
       spp_to=None,
       dst_flag=None
   )

Load Methods
------------

See :doc:`load` for detailed load documentation.

get_actual_system_load_by_weather_zone()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   load_data = client.get_actual_system_load_by_weather_zone(
       operating_day_from=date(2026, 1, 1),
       operating_day_to=date(2026, 1, 31),
       dst_flag=None
   )

get_actual_system_load_by_forecast_zone()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   load_data = client.get_actual_system_load_by_forecast_zone(
       operating_day_from=date(2026, 1, 1),
       operating_day_to=date(2026, 1, 31),
       dst_flag=None
   )

get_native_load()
~~~~~~~~~~~~~~~~~

.. code-block:: python

   native_load = client.get_native_load(
       operating_day_from=date(2024, 10, 1),
       operating_day_to=date(2024, 10, 31)
   )

Demand Response Methods
------------------------

See :doc:`demand-response` for detailed DR documentation.

get_monthly_demand_response()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Download NP3-108: Monthly ERCOT Demand Response from Load Resources.

.. code-block:: python

   dr_data = client.get_monthly_demand_response(
       month=date(2026, 1, 1)
   )

**Parameters:**

* ``month`` (DateLike): Any date within the target month

**Returns:** Report payload with fields:

* ``month`` (str): Month identifier (e.g., "JAN-26")
* ``hour`` (int): Hour of day (1-24)
* ``asType`` (str): Ancillary Service type (ECRS, NSPIN, RRS, etc.)
* ``houston`` (float): Houston zone MW
* ``north`` (float): North zone MW
* ``south`` (float): South zone MW
* ``west`` (float): West zone MW
* ``resourceType`` (str): "CLR" or "NCLR"

get_monthly_demand_response_ers()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Download NP3-107: Monthly ERCOT Demand Response from ERS.

.. code-block:: python

   ers_data = client.get_monthly_demand_response_ers(
       month=date(2026, 1, 1),
       report_type_id=None
   )

**Parameters:**

* ``month`` (DateLike): Any date within the target month
* ``report_type_id`` (int, optional): MIS portal report type ID

**Returns:** Report payload or None if not available

.. note::

   NP3-107 is currently not available through the Public API.
   The method will return None with an informative error message.

Ancillary Services Methods
---------------------------

get_dam_cleared_ancillary_service()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   # Available services: ECRSM, ECRSS, NSPIN, NSPNM, REGDN, REGUP, RRSFFR, RRSPFR, RRSUFR
   as_data = client.get_dam_cleared_ancillary_service(
       service="REGUP",
       delivery_date_from=date(2026, 1, 1),
       delivery_date_to=date(2026, 1, 2)
   )

get_dam_ancillary_service_offers()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   offers_data = client.get_dam_ancillary_service_offers(
       service="REGUP",
       delivery_date_from=date(2026, 1, 1),
       delivery_date_to=date(2026, 1, 2)
   )

get_sced_ancillary_service_offers()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   sced_offers = client.get_sced_ancillary_service_offers(
       service="REGUP",
       start=datetime(2026, 1, 1, 0, 0, 0),
       end=datetime(2026, 1, 2, 23, 59, 59)
   )

Utility Methods
---------------

save_report_to_csv()
~~~~~~~~~~~~~~~~~~~~

Save report payload to CSV file.

.. code-block:: python

   csv_path = client.save_report_to_csv(
       report_payload=payload,
       filename="my_report.csv"
   )

**Parameters:**

* ``report_payload`` (dict): Report payload from any get method
* ``filename`` (str): Output filename (relative to data_dir)

**Returns:** Path to saved CSV file or None

cleanup()
~~~~~~~~~

Close the HTTP session. Always call when done.

.. code-block:: python

   client.cleanup()

Archive Methods
---------------

get_archive_entries()
~~~~~~~~~~~~~~~~~~~~~

List available archive files for a report.

.. code-block:: python

   entries = client.get_archive_entries("np3-108")

**Returns:** List of archive entry dicts with keys:

* ``docId``: Document ID
* ``friendlyName``: File name
* ``postDatetime``: Publication timestamp

download_archive()
~~~~~~~~~~~~~~~~~~

Download an archive file by report ID and document ID.

.. code-block:: python

   content = client.download_archive(
       report_id="np3-108",
       doc_id=1234567
   )

**Returns:** Bytes content or None

Data Types
----------

DateLike
~~~~~~~~

Accepts any of:

* ``datetime.date`` object
* ``datetime.datetime`` object
* ISO-format string (e.g., "2026-01-01" or "2026-01-01T12:00:00")

Example Usage Patterns
----------------------

Context Manager Pattern
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   from lib.iso.ercot import ERCOTClient, ERCOTConfig

   config = ERCOTConfig.from_ini_file()
   client = ERCOTClient(config)

   try:
       data = client.get_dam_hourly_lmps(
           delivery_date_from=date(2026, 1, 1),
           delivery_date_to=date(2026, 1, 31)
       )
       client.save_report_to_csv(data, "lmps.csv")
   finally:
       client.cleanup()

Batch Processing
~~~~~~~~~~~~~~~~

.. code-block:: python

   from datetime import date, timedelta

   config = ERCOTConfig.from_ini_file()
   client = ERCOTClient(config)

   try:
       # Download a year of monthly DR data
       start_date = date(2025, 1, 1)
       for month_offset in range(12):
           month = start_date + timedelta(days=30 * month_offset)
           dr_data = client.get_monthly_demand_response(month)
           if dr_data:
               filename = f"dr_{month.strftime('%Y_%m')}.csv"
               client.save_report_to_csv(dr_data, filename)
               print(f"Saved {filename}")
   finally:
       client.cleanup()

Error Handling
~~~~~~~~~~~~~~

.. code-block:: python

   config = ERCOTConfig.from_ini_file()
   client = ERCOTClient(config)

   try:
       data = client.get_dam_hourly_lmps(
           delivery_date_from=date(2026, 1, 1),
           delivery_date_to=date(2026, 1, 31)
       )

       if data is None:
           print("Failed to retrieve data (check logs)")
       elif not data.get('data'):
           print("No data available for this period")
       else:
           print(f"Retrieved {len(data['data'])} rows")
           client.save_report_to_csv(data, "lmps.csv")

   except Exception as e:
       print(f"Error: {e}")
   finally:
       client.cleanup()

See Also
--------

* :doc:`overview` - ERCOT overview and setup
* :doc:`demand-response` - Demand response guide
* :doc:`pricing` - Pricing data guide
* :doc:`load` - Load data guide
