Querying the Planet: Leveraging GeoParquet to work with global scale open geospatial data locally and in the cloud.
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

1. [What is Overture Maps?](#1-what-is-overture-maps)
   1. Look at some Overture Data [explore.overturemaps.org](//explore.overturemaps.org)

2. Dig deeper into Overture data with Fused.io
   1. Explore raw Overture data by theme and type
   2. _Fuse_ Overture data with an external dataset (National Structures Inventory) to fill in data gaps

3. Interface with Overture data via DuckDB







<br /><br /><br /><hr /><br /><br /><br />

# 1. What is Overture Maps?
[Back to Agenda](#workshop-agenda)

The [Overture Maps Foundation](//overturemaps.org) is an open data project within the Linux Foundation that aims "Power current and next-generation map products by creating reliable, easy-to-use, and interoperable open map data."

Primarily, "Overture is for developers who build map services or use geospatial data." However, Overture is a fantastic resource for researchers looking to work with one of the most complete and computationally efficient open geospatial datasets.

### Explore Overture Data

1. Visit [explore.overturemaps.org](//explore.overturemaps.org) and poke around. This site offers an "x-ray" view of Overture data.
2. Overture data




<br /><br /><br /><hr /><br /><br /><br />

# 2. Fused.io
[Back to Agenda](#workshop-agenda)

### 1. Getting started with Fused: [The Overture Maps Example UDF](https://www.fused.io/workbench/catalog/Overture_Maps_Example-64071fb8-2c96-4015-adb9-596c3bac6787)
1. In a new browser window, navigate to: [Overture Maps Example](https://www.fused.io/workbench/catalog/Overture_Maps_Example-64071fb8-2c96-4015-adb9-596c3bac6787)
2. Click "Add to UDF Builder"
3. On the far left, adjust the parameters to view different types of data from Overture.
4. Hover over features on the map to see the complete, raw, Overture data.
5. Zoom all the way out to see the spatial partitioning.


### 2. _Fusing_ Datasets with Overture in the browser:

1. Add the [Overture Nsi](https://www.fused.io/workbench/catalog/Overture_Nsi-dd89972c-ce30-4544-ba0f-81fc09f5bbef) UDF to your fused workbench.
2. Notice the `join with NSI`  parameter in this UDF. What does this do?

<br /><br /><br /><hr /><br /><br /><br />


# 3. DuckDB
[Back to Agenda](#workshop-agenda)

Now that we've seen Overture data in the browser, let's download some locally with DuckDB.

First, [Install DuckDB](https://duckdb.org/docs/installation/?version=stable&environment=cli&platform=macos&download_method=package_manager) version >= 1.1.1

Next, you'll need a A GIS environment of your choice. Both QGIS and Esri ArcMap or similar should work. To load geoparquet directly into QGIS, you will need a [version of QGIS with the latest GDAL](https://docs.overturemaps.org/examples/QGIS/). Alternatively, most of this workshop can be visualized with [kepler.gl](kepler.gl)

_If you do not want to install DuckDB locally, you can sign up for MotherDuck, a cloud-based DuckDB, however, you will not be able to use the `COPY TO` commands nor the `h3` extension._


## Part 1. Places

_**Tip**: When launching DuckDB, specify a persistent DB, such as `duckdb my_db.duckdb`s. This way if you create tables, you can access them later._


### Step 1: Query for places in a particular location:

1. Obtain a bounding box of interest https://boundingbox.klokantech.com/ is a great tool for creating a bounding box. Specifically, it lets you copy the coordinates in the following format (DublinCore) which is very human-readable. Here is a bounding box for Montréal:

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

    Consult the [places schema](https://docs.overturemaps.org/schema/reference/places/place/) to learn more about which columns can be accessed and their data types.

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


Create a local GeoParquet file:






### Step 2: Use DuckDB `spatial` extension to convert to common spatial data formats.

1. Ensure the `spatial` extension is installed:  `install spatial;`.
2. Load the spatial extension with `load spatial;`

3. Now we can use our same query again, but this time we add the `COPY TO` command to write GeoJSON. We can also remove the `LIMIT` argument.

    ```sql
    COPY(
        "< OUR QUERY FROM ABOVE >"
    ) TO 'old_city_places.geojson' WITH (FORMAT GDAL, DRIVER 'GeoJSON')
    ```

4. Now open that GeoJSON file in your preferred GIS environment to inspect the attributes (fastest to drag-n-drop into kepler.gl)

5. **(Optional):** Download all of the places for a larger region and save them in a table to be accessed later with a `CTAS` command:

    ```sql
    CREATE TABLE germany_places AS (
        SELECT
        id,
        names.primary as name,
        categories.primary as category,
        geometry
    FROM read_parquet('s3://overturemaps-us-west-2/release/2024-09-18.0/theme=places/type=place/*', filename=true, hive_partitioning=1)
    WHERE
        bbox.xmin BETWEEN 5.87 AND 15.04
        AND bbox.ymin BETWEEN 47.27 AND 55.1
    )
    ```

    Now we can reference places in Germany without pulling any more data from Overture (_faster!_).


    You can create a parquet file of places to use locally with the `COPY TO` command:

    ```sql
    load spatial;
    COPY(
        SELECT
            id,
            name,
            geometry
        FROM germany_places
    ) TO 'germany_bbox_places.parquet';
    ```

    This will write a geoparquet file to your computer. Recent versions of QGIS can also read GeoParquet. See [these instructions if you're stuck](https://docs.overturemaps.org/examples/QGIS/).

    Additionally, you can access this file at any time with `FROM read_parquet('germany.parquet')`

## Part 2: Divisions

1. Let's create a table with Country boundaries that we can reference later. In Overture, administrative boundaries are known as divisions:

    ```sql
    CREATE OR REPLACE TABLE countries AS (
        SELECT
            names.primary as name,
            names.common['en'][1] as name_en,
            names.common['de'][1] as name_de,
            names.common['fr'][1] as name_fr,
            geometry AS border
        FROM read_parquet('s3://overturemaps-us-west-2/release/2024-09-18.0/theme=divisions/type=division_area/*', hive_partitioning=1)
        WHERE subtype = 'country'
    );
    ```

    This might take some time, depending on your internet connection speed, but in the end, you should high resolution boundaries from OpenStreetMap for 234 countries.

2. We can export that file as parquet with the `COPY TO` command again:
    ```sql
    COPY(
        SELECT
            *,
        border as geometry
    FROM countries
    ) TO 'countries.parquet';
    ```

    DuckDB Spatial can perform many spatial operations, so we can use it to clip our places data to a more exact bounding box of a country. Remember that these are very high-fidelity country borders, so use the `ST_SIMPLIFY` command to improve performance:

    ```sql
    load spatial;
    COPY(
        SELECT
            id,
            name,
            geometry
    FROM read_parquet('germany_bbox_places.parquet')
    WHERE ST_CONTAINS(
        (
            SELECT ST_SIMPLIFY(border, 0.01)
            FROM read_parquet('countries.parquet')
            WHERE name_en = 'Germany'
        ),
        geometry
        )
    ) TO 'germany_places.parquet';
    ```

## Part 3: H3
We can do spatial analysis with the [h3 grid system](https://h3geo.org/), available in duckdb via the h3 plugin:

1. Install the extension:
    ```sql
    install h3 from community;
    load h3;
    ```

2. Density of places per h3 cell:
    ```sql
    COPY(
        SELECT
            h3_latlng_to_cell_string(ST_Y(geometry), ST_X(geometry), 8) as h3,
            count(1) as _count
    FROM read_parquet('germany_places.parquet')
    GROUP BY h3_latlng_to_cell_string(ST_Y(geometry), ST_X(geometry), 8)
    ) TO 'h3.csv';
    ```

    Now drag-n-drop h3.csv directly into kepler.gl, which will turn the h3 index into a polygon automatically:


3. Just to demonstrate the power of this extension, let's find all of the h3 polyons that cover our country boundarie:
    ```sql
    load h3;
    COPY(
        SELECT h3_h3_to_string(h3) AS h3,
        name_en
        FROM (
            SELECT name_en,
                s.geom as geom
            FROM read_parquet('countries.parquet')
            CROSS JOIN UNNEST(ST_DUMP(border)) AS t(s)
        )
        cross join unnest(h3_polygon_wkt_to_cells(ST_ASTEXT(geom), 4)) as t(h3)
    ) TO 'h3_countries.csv';
    ```

    You now have a file that maps h3 cell to Country name that you can join other h3-aggregated data to for per-Country analysis.


## Part 4:  Buildings

1. Overture contains more than 2B building footprints. Fetching them all to our local machine is not very valuable. However, we can interact with their metadata in the cloud:

    Overutre data is available both on Amazon S3 and Microsoft Azure Blob Storage. In this example, we'll use the data form Azure:

    ```sql
    SET azure_storage_connection_string = 'DefaultEndpointsProtocol=https;AccountName=overturemapswestus2;AccountKey=;EndpointSuffix=core.windows.net';

    COPY(
        SELECT
            id,
            names.primary as primary_name,
            height,
            sources[1].dataset AS primary_source,
            sources[1].record_id AS source_id,
            geometry
        FROM read_parquet('azure://release/2024-09-18.0/theme=buildings/type=building/*', filename=true, hive_partitioning=1)
        WHERE bbox.xmin BETWEEN 10.959934 AND 11.211332
        AND bbox.ymin BETWEEN 49.397337 AND 49.501401
    ) TO 'buildings.parquet';
    ```

    Now compute h3 densities for buildings:

    ```sql
    COPY(
        SELECT
            h3_latlng_to_cell_string(ST_Y(ST_CENTROID(geometry)), ST_X(ST_CENTROID(geometry)), 10) as h3,
            count(1) as _count
    FROM read_parquet('buildings.parquet')
    GROUP BY h3_latlng_to_cell_string(ST_Y(ST_CENTROID(geometry)), ST_X(ST_CENTROID(geometry)), 10)
    ) TO 'buildings_h3.csv';
    ```

2. What if you wanted to do this for a much larger area? Now we can _really_ leverage Parquet's predicate pushdown to fetch a subset of the building data.

    Since each feature has a `bbox`, we won't use the buildlings actual geometry, but just the bbox.xmin and bbox.ymin. This is a fine proxy for the location of the building.

    By only requesting these columns, we dramatically shrink the amount of data we must download.

    ```sql
    COPY(
        SELECT
            h3_latlng_to_cell_string(bbox.ymin, bbox.xmin, 8) as h3,
            count(1) as building_count
    FROM read_parquet('buildings.parquet')
    WHERE bbox.xmin BETWEEN 5.87 AND 15.04
        AND bbox.ymin BETWEEN 47.27 AND 55.1
    GROUP BY h3_latlng_to_cell_string(bbox.ymin, bbox.xmin, 8)
    ) TO 'building_density_h3_res8.csv';
    ```


## Part 5: Transportation Theme
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

## Part 6: Base
What is the base theme? Consult the documention here: docs.overturemaps.org/guides/base/

1. Download all of the peaks in Europe:
    ```sql
    SET s3_region='us-west-2';

    COPY(
        SELECT
            id,
            names.primary as name,
            elevation,
            geometry
        FROM read_parquet('s3://overturemaps-us-west-2/release/2024-09-18.0/theme=base/type=land/*', filename=true, hive_partitioning=1)
        WHERE subtype = 'physical'
            AND class IN ('peak','volcano')
            AND elevation IS NOT NULL
            AND bbox.xmin BETWEEN -12.8 AND 29.7
            AND bbox.ymin BETWEEN 34.7 AND 58.2
    ) TO 'european_peaks.parquet';
    ```

2. We can build an h3-gridded DEM for European high points from this file:
    ```sql
        COPY(
            SELECT
                h3_latlng_to_cell_string(
                    ST_Y(ST_CENTROID(geometry)),
                    ST_X(ST_CENTROID(geometry)),
                6) as h3,
                max(elevation) as _max,
                min(elevation) as _min,
                avg(elevation) as _avg
        FROM read_parquet('european_peaks.parquet')
        GROUP BY h3_latlng_to_cell_string(ST_Y(ST_CENTROID(geometry)), ST_X(ST_CENTROID(geometry)), 6)
        ) TO 'peaks_h3.csv';
    ```

What other types of features from OSM are you interested in exploring? The logic for how features convert from OSM to Overture is here: https://docs.overturemaps.org/schema/concepts/by-theme/base/


## Next Steps
Overture keeps a list of projects from the community. Many of these are great tutorials on working with Overture data: https://docs.overturemaps.org/community/.

For further inspiration, this blog post uses AI for places data conflation: https://www.dbreunig.com/2024/09/27/conflating-overture-points-of-interests-with-duckdb-ollama-and-more.html



###
