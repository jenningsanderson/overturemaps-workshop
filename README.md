<img width="1375" alt="image" src="https://github.com/user-attachments/assets/b45b188b-7712-4502-8ccf-601513fc81f6">Querying the Planet: Leveraging GeoParquet to work with global scale open geospatial data locally and in the cloud
===

> Consisting of open data from OpenStreetMap, Meta, Esri, Microsoft, Google, and more, Overture Maps data is conflated and converted to a consistent schema before being distributed as geoparquet files in the cloud. This workshop will explore the advantages of GeoParquet and cloud-native geospatial technologies for researchers working with the data both locally and in the cloud.

### Resources

| Name | Description |
| ---- | ----------- |
| [Overture Explore Page](//explore.overturemaps.org) | Easiest place to get an overview of Overture data in an X-Ray map view  |
| [Overture Documentation](//docs.overturemaps.org/) | Schema definitiona nd examples of how to access and work with Overture data.  |
| [Fused.io](//fused.io) | A new cloud-based GIS with user-defined functions
| [DuckDB](https://duckdb.org/) | An fast in-process database system for analytics and data manipulation |

# Workshop Agenda
- [1. What is Overture Maps?](#1-what-is-overture-maps)
    - [Explore Overture Data](#explore-overture-data)
- [2. Fused.io](#2-fusedio)
    - [1. Getting started with Fused: The Overture Maps Example UDF](#1-getting-started-with-fused-the-overture-maps-example-udf)
    - [2. _Fusing_ Datasets with Overture in the browser](#2-fusing-datasets-with-overture-in-the-browser)
- [3. DuckDB](#3-duckdb)
  - [Part I. Places Theme](#part-i-places-theme)
    - [Step 1: Query for places in a particular location](#step-1-query-for-places-in-a-particular-location)
    - [Step 2: Use DuckDB `spatial` extension to convert to common spatial data formats](#step-2-use-duckdb-spatial-extension-to-convert-to-common-spatial-data-formats)
  - [Part II: Buildings Theme](#part-ii-buildings-theme)
  - [Part III: Transportation Theme](#part-iii-transportation-theme)
  - [Part IV: Base Theme](#part-iv-base-theme)
- [4. Bring the Analysis to the Data in the cloud with Fused](#4-bring-the-analysis-to-the-data-in-the-cloud-with-fused)
  - [Overture \& Oakridge Comparision](#overture--oakridge-comparision)
  - [H3 Aggregated Skyline](#h3-aggregated-skyline)


<br /><br /><br /><br /><br /><br />

# 1. What is Overture Maps?

[Back to Agenda](#workshop-agenda)

![image](https://github.com/user-attachments/assets/9100d3ee-beb9-479c-a257-ef87502cbe1e)

The [Overture Maps Foundation](//overturemaps.org) is an open data project within the Linux Foundation that aims to "Power current and next-generation map products by creating reliable, easy-to-use, and interoperable open map data."

Primarily, "Overture is for developers who build map services or use geospatial data." Additionally, Overture is a fantastic resource for researchers looking to work with one of the most complete and computationally efficient open geospatial datasets.

![image](https://github.com/user-attachments/assets/c80345b7-e0d8-471d-9f03-af6979ef9645)


### Explore Overture Data

1. Visit [explore.overturemaps.org](//explore.overturemaps.org) and poke around. This site offers an "x-ray" view of Overture data.
2. Overture has **6** data themes: 
    - Divisions, 
    - Base, 
    - Transportation
    - Buildings
    - Places
    - Addresses.
   
   The explore page lets you inspect the properties of each feature and links out to the overture schema: [docs.overturemaps.org/schema](//docs.overturemaps.org/schema) where you can learn more about the attributes available for each theme.

The explore page helps us get an overview of what's in Overture by rendering pre-processed PMTiles archives on a web map. Next, we'll look at the different ways we can interact with Overture data in the raw, Geoparquet format. 


<br /><br /><br /><br /><br /><br />

# 2. Fused.io

[Back to Agenda](#workshop-agenda)

![image](https://github.com/user-attachments/assets/ff9f8a75-7b9d-4039-89d6-0001ac8c952c)

Fused is a new analytical platform with powerful capabilities to read and visualize geoparquet right in your browser. The Fused workbench allows you to run any number of public User-Defined Functions, or UDFs. 

### 1. Getting started with Fused: [The Overture Maps Example UDF](https://www.fused.io/workbench/catalog/Overture_Maps_Example-64071fb8-2c96-4015-adb9-596c3bac6787)

![image](https://github.com/user-attachments/assets/2978542d-186a-4950-b09e-75f1b131b7a5)

1. In a new browser window, navigate to: [Overture Maps Example](https://www.fused.io/workbench/catalog/Overture_Maps_Example-64071fb8-2c96-4015-adb9-596c3bac6787)
2. Click "Add to UDF Builder"
3. On the far left, adjust the parameters to view different types of data from Overture.
4. Hover over features on the map to see the complete, raw, Overture data.
5. Zoom all the way out to see the spatial partitioning:

    ![image](https://github.com/user-attachments/assets/6baf7efd-cad8-4881-ae9a-6fe4c91699fe)


### 2. _Fusing_ Datasets with Overture in the browser

Now that we've seen what's in Overture data, can we combine (or _fuse_) our Overture data with another dataset? 

1. Add the [Overture Nsi](https://www.fused.io/workbench/catalog/Overture_Nsi-dd89972c-ce30-4544-ba0f-81fc09f5bbef) UDF to your fused workbench.
2. Notice the `join with NSI` parameter in this UDF.

Fargo, North  Dakota: 
![image](https://github.com/user-attachments/assets/4350628c-5e36-4bab-9938-d1968757d6df)

![image](https://github.com/user-attachments/assets/86ed00d9-39dd-4869-ab2e-56dca5e7359b)




<br /><br /><br /><hr><br /><br /><br />

# 3. DuckDB

[Back to Agenda](#workshop-agenda)

Now that we've seen Overture data in the browser, let's dig in further. Since the data is hosted in the cloud as GeoParquet files, we will interface with it via a tool called DuckDB that can take advantage of this cloud-native format.

First, [Install DuckDB](https://duckdb.org/docs/installation/?version=stable&environment=cli&platform=macos&download_method=package_manager) version >= 1.1.1

Next, you'll need a A GIS environment of your choice. Both QGIS and Esri ArcMap or similar should work. To load geoparquet directly into QGIS, you will need a [version of QGIS with the latest GDAL](https://docs.overturemaps.org/examples/QGIS/). Alternatively, most of this workshop can be visualized with [kepler.gl](kepler.gl)

_If you do not want to install DuckDB locally, you can sign up for MotherDuck, a cloud-based DuckDB, however, you will not be able to use the `COPY TO` commands nor the `h3` extension._

## Part I. Places Theme

_**Tip**: When launching DuckDB, specify a persistent DB, such as `duckdb my_db.duckdb`. This way if you create tables, you can access them later._

### Step 1: Query for places in a particular location

1. Obtain a bounding box of interest (<https://boundingbox.klokantech.com>) is a great tool for creating a bounding box. Specifically, it lets you copy the coordinates in the following format (DublinCore) which is very human-readable.
Here is a bounding box for Montréal:

    ```python
    westlimit=-73.974157
    southlimit=45.410076
    eastlimit=-73.474295
    northlimit=45.70479
    ```

    (I recommend a smaller bounding box, like just a small city or neighborhood for now so you're not working with a lot of data in the example).

2. A basic places query looks like this:

    ```sql
    SELECT
        id,
        names.primary as name,
        confidence,
        geometry
    FROM read_parquet('s3://overturemaps-us-west-2/release/2024-10-23.0/theme=places/type=place/*', filename=true, hive_partitioning=1)
    WHERE
        bbox.xmin BETWEEN X_WEST AND X_EAST
        AND bbox.ymin BETWEEN Y_SOUTH AND Y_NORTH
    LIMIT 10;
    ```

3. Update the query with the proper values in the `WHERE` clause for `X` and `Y` from your bounding box. Remember, east/west = longitude = X and north/south = latitude = Y.

4. Paste your query into DuckDB and run it.

    You should see something similar to this:

        ┌──────────────────────────────────┬───────────────────────────────────────────┬─────────────────────┬────────────────────────────────┐
        │                id                │                   name                    │     confidence      │            geometry            │
        │             varchar              │                  varchar                  │       double        │            geometry            │
        ├──────────────────────────────────┼───────────────────────────────────────────┼─────────────────────┼────────────────────────────────┤
        │ 08f2b81b7170b90803a1b4376438169b │ Hôtel de ville de Senneville              │  0.9544565521095278 │ POINT (-73.9600016 45.4136658) │
        │ 08f2b81b7171c0db037e3818c87b6c02 │ charles rivers                            │ 0.30856423173803527 │ POINT (-73.96118 45.41467)     │
        │ 08f2b81b714f2a2603f85a7ea76652ac │ Club De Voile Senneville                  │  0.5592783505154639 │ POINT (-73.9685221 45.4187574) │
        │ 08f2b81b714ad4e5037c634d2494f50e │ Vignoble Souffle de Vie                   │                0.77 │ POINT (-73.9676663 45.4201718) │
        │ 08f2b81b71583049039f215b4d3b9a7b │ Souffle de Vie Vineyard                   │  0.9567577686259828 │ POINT (-73.9678019 45.4228501) │
        │ 08f2b81b7158c219034200ed79cfbc13 │ Tenaquip Limited                          │  0.9826508620689655 │ POINT (-73.9657541 45.4229399) │
        │ 08f2b81b7152e4140398b266e3eab10b │ Cimetière et Complexe Funéraire Belvédère │  0.9797443181818182 │ POINT (-73.9618394 45.4233901) │
        │ 08f2b81b703aa099031e13858e363b78 │ Les Écuries de Senneville | Senneville QC │  0.9537408699085346 │ POINT (-73.9693036 45.4291791) │
        │ 08f2b81b700551a2039ae5eda64366c7 │ Ferme GUSH Farm                           │  0.9213349225268176 │ POINT (-73.96907 45.43491)     │
        │ 08f2b81b70b90148038bf5f719a95574 │ Braeside Golf Club                        │  0.9588815789473685 │ POINT (-73.9626141 45.4377414) │
        ├──────────────────────────────────┴───────────────────────────────────────────┴─────────────────────┴────────────────────────────────┤
        │ 10 rows                                                                                                                   4 columns │
        └─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────┘

    Notice the type of the geometry column is `geometry`. DuckDB recognizes the geo metadata in the source parquet files and automatically converts the column to a geometry type.

3. Consult the [places schema](https://docs.overturemaps.org/schema/reference/places/place/) to learn more about which columns can be accessed and their data types.

### Step 2: Use DuckDB `spatial` extension to convert to common spatial data formats

1. Ensure the `spatial` extension is installed:  `install spatial;`.
2. Load the spatial extension with `load spatial;`

3. Now we can use our same query again, but this time we add the `COPY TO` command to write GeoJSON. We can also remove the `LIMIT` argument. The complete query looks like this:

    ```sql
    INSTALL spatial;
    LOAD spatial;

    COPY(
    SELECT
        id,
        names.primary as name,
        confidence,
        geometry
    FROM read_parquet('s3://overturemaps-us-west-2/release/2024-10-23.0/theme=places/type=place/*', filename=true, hive_partitioning=1)
    WHERE
        bbox.xmin BETWEEN -73.974157 AND -73.474295
        AND bbox.ymin BETWEEN 45.410076 AND 45.70479
    ) TO 'montreal.geojson' WITH (FORMAT GDAL, DRIVER GeoJSON);
    ```

4. Now open that GeoJSON file in your preferred GIS environment to inspect the attributes (fastest to drag-n-drop into kepler.gl)

5. Are there other columns that would be useful? Try adding `categories.primary as category,` to the query to get the category for each place.



## Part II: Buildings Theme

1. Overture contains more than 2B building footprints. Fetching them all to our local machine is not very valuable. However, we can interact with their metadata in the cloud:

    Overutre data is available both on Amazon S3 and Microsoft Azure Blob Storage. In this example, we'll use the data from Azure:

    ```sql
    INSTALL azure;
    LOAD azure;
    SET azure_storage_connection_string = 'DefaultEndpointsProtocol=https;AccountName=overturemapswestus2;AccountKey=;EndpointSuffix=core.windows.net';

    LOAD spatial;

    COPY(
        SELECT
            id,
            names.primary as primary_name,
            height,
            sources[1].dataset AS primary_source,
            sources[1].record_id AS source_id,
            geometry
        FROM read_parquet('azure://release/2024-10-23.0/theme=buildings/type=building/*', filename=true, hive_partitioning=1)
        WHERE bbox.xmin BETWEEN -122.352055 AND -122.316697
        AND bbox.ymin BETWEEN 47.593064 AND 47.619655
    ) TO 'seattle_buildings.geojson' WITH (FORMAT GDAL, DRIVER GeoJSON);
    ```

2. Use DuckDB's h3 extension to calculate building densities for a larger area:

    ```sql
    COPY(
        SELECT
            h3_latlng_to_cell_string(ST_Y(ST_CENTROID(geometry)), ST_X(ST_CENTROID(geometry)), 10) as h3,
            count(1) as _count
    FROM read_parquet('buildings.parquet')
    GROUP BY h3_latlng_to_cell_string(ST_Y(ST_CENTROID(geometry)), ST_X(ST_CENTROID(geometry)), 10)
    ) TO 'buildings_h3.csv';
    ```

## Part III: Transportation Theme

The transportation theme has 2 types of data, connectors and segments.

1. Get started by looking at the segments in Paris:

    ```sql
    COPY(
        SELECT
        id,
        names.primary as name,
        class,
        geometry
        FROM read_parquet('s3://overturemaps-us-west-2/release/2024-09-18.0/theme=transportation/type=segment/*', filename=true, hive_partitioning=1)
        WHERE bbox.xmin > 2.276
        AND bbox.ymin > 48.865
        AND bbox.xmax < 2.314
        AND bbox.ymax < 48.882
    ) TO 'paris_roads.geojson' WITH (FORMAT GDAL, DRIVER 'GeoJSON');
    ```

2. Connectors make for a good proxy of road density. First we'll download a bunch of connectors to a local table:

    ```sql
    load h3;
    CREATE OR REPLACE TABLE connectors AS (
        SELECT
            h3_latlng_to_cell_string(ST_Y(geometry), ST_X(geometry), 8) as h3,
        id,
        geometry
        FROM read_parquet('s3://overturemaps-us-west-2/release/2024-09-18.0/theme=transportation/type=connector/*', filename=true, hive_partitioning=1)
        WHERE bbox.xmin > 8.82
        AND bbox.ymin > 48.5
        AND bbox.xmax < 13.36
        AND bbox.ymax < 50.39
    );
    ```

3. Now aggregate by h3 cell:

    ```sql
    COPY(
        SELECT
            h3,
            count(id)
        FROM connectors
        GROUP BY
            h3
    ) TO 'connectors_h3.csv';
    ```

## Part IV: Base Theme

What is the base theme?

1. Query Overture for all of the peaks in Europe:
    ```sql
    SET s3_region='us-west-2';

    COPY(
        SELECT
        id,
        names.primary as name,
        elevation,
        geometry
        FROM read_parquet('s3://overturemaps-us-west-2/release/2024-10-23.0/theme=base/type=land/*', filename=true, hive_partitioning=1)
        WHERE subtype = 'physical' AND class IN ('peak','volcano') AND elevation IS NOT NULL
        AND bbox.xmin BETWEEN -12.8 AND 29.7
        AND bbox.ymin BETWEEN 34.7 AND 58.2
    ) TO 'european_peaks.parquet';

2. We can build an h3-gridded DEM for European high points from this file:
    ```sql
    COPY(
        SELECT
            h3_latlng_to_cell_string(ST_Y(ST_CENTROID(geometry)), ST_X(ST_CENTROID(geometry)), 6) as h3,
            max(elevation) as _max,
            min(elevation) as _min,
            avg(elevation) as _avg
    FROM read_parquet('european_peaks.parquet')
    GROUP BY h3_latlng_to_cell_string(ST_Y(ST_CENTROID(geometry)), ST_X(ST_CENTROID(geometry)), 6)
    ) TO 'peaks_h3.csv';
    ```

What is the base theme? Consult the documention here: docs.overturemaps.org/guides/base/


What other types of features from OSM are you interested in exploring? The logic for how features convert from OSM to Overture is here: <https://docs.overturemaps.org/schema/concepts/by-theme/base/>






<br /><br /><br /><hr><br /><br /><br />

# 4. Bring the Analysis to the Data in the cloud with Fused
[Back to Agenda](#workshop-agenda)

Since our data lives in the cloud, let's bring our analysis to the data, not the other way around.

## Overture & Oakridge Comparision

   1. Load the [Overture OakRidge Comparison](https://www.fused.io/workbench/catalog/Overture_OakRidge_Comparison-0ebc66b4-fbd5-4b44-ab97-af6d30757891) into your Fused workbench.
   2. Compare the building footprints between ORNL and Overture. Which one has more accurate building footprints?
   3. Now which dataset has more accurate _class_ information?
   4. Fused lets us combine these datasets.

## H3 Aggregated Skyline

   1. Additionally, we can perform the same H3 aggregations in the cloud: The [Overture H3 Skyline](https://www.fused.io/workbench/catalog/Overture_H3_Skyline-1b1b240c-f378-4737-856b-18d9568fd8f1) UDF aggregates Overture buildings at any h3 resolution — allowing you to "approximate" a city skyline without having to render all of the buildings.
