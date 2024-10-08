Querying the Planet: Leveraging GeoParquet to work with global scale open geospatial data locally and in the cloud.
===

> Consisting of open data from OpenStreetMap, Meta, Esri, Microsoft, Google, and more, Overture Maps data is conflated and converted to a consistent schema before being distributed as geoparquet files in the cloud. This workshop will explore the advantages of GeoParquet and cloud-native geospatial technologies for researchers working with the data both locally and in the cloud.

### Resources

| Name | Description |
| ---- | ----------- |
| [Overture Explore Page](//explore.overturemaps.org) | Best place to get started looking at Overture data |
| [Overture Documentation](//docs.overturemaps.org/) | Continuously updating resource with examples of how to access Overture data |
| [Overture Data Schema](//docs.overturemaps.org/schema/) | The schema-specific subpage of the documentation |
| [Discussions on Github](https://github.com/OvertureMaps/data/discussions) | We welcome feedback and discussions on Github. |
| [DuckDB spatial functions](https://duckdb.org/docs/extensions/spatial/functions) | A list of spatial functions available in the DuckDB spatial extension |
| [DuckDB H3 extension](https://community-extensions.duckdb.org/extensions/h3.html) | Documentation of the DuckDB h3 extension |

### Workshop Prerequisites:
_If you do not want to install DuckDB locally, you can sign up for MotherDuck, a cloud-based DuckDB, however, you will not be able to use the `COPY TO` commands nor the `h3` extension._

1. [Install DuckDB](https://duckdb.org/docs/installation/?version=stable&environment=cli&platform=macos&download_method=package_manager) version >= 1.1.1
2. A GIS environment of your choice:
   1. Both QGIS and Esri ArcMap or similar should work. To load geoparquet directly into QGIS, you will need a [version of QGIS with the latest GDAL](https://docs.overturemaps.org/examples/QGIS/).
   2. Most of this workshop can be visualized with [kepler.gl](kepler.gl)


_**Tip**: When launching DuckDB, specify a persistent DB, such as `duckdb my_db.duckdb`s. This way if you create tables, you can access them later._


## Part 1. Places

### Step 1: Query for places in a particular location:

1. Obtain a bounding box of interest https://boundingbox.klokantech.com/ is a great tool for creating a bounding box. Specifically, it lets you copy the coordinates in the following format (DublinCore) which is very human-readable. Here is one for the old city in Nürnberg:

    ```python
    west_limit=11.069371
    south_limit=49.451761
    east_limit=11.090204
    north_limit=49.461126
    ```

    (I recommend a smaller bounding box, like just a small city or neighborhood for now so you're not working with a lot of data in the example).

2. A basic places query looks like this:

    ```sql
    SELECT
        id,
        names.primary as name,
        confidence,
        geometry
    FROM read_parquet('s3://overturemaps-us-west-2/release/2024-09-18.0/theme=places/type=place/*', filename=true, hive_partitioning=1)
    WHERE
        bbox.xmin BETWEEN X AND X
        AND bbox.ymin BETWEEN Y AND Y
    LIMIT 10;
    ```

    Consult the [places schema](https://docs.overturemaps.org/schema/reference/places/place/) to learn more about which columns can be accessed and their data types.

3. Update the query with the proper values in the `WHERE` clause for `X` and `Y` from your bounding box. Remember, east/west = longitude = X and north/south = latitude = Y.

4. Paste your query into DuckDB and run it.

You should see something similar to this:

    ┌──────────────────────┬─────────────────────────────────┬────────────────────┬───────────────────────────────┐
    │          id          │              name               │     confidence     │           geometry            │
    │       varchar        │             varchar             │       double       │           geometry            │
    ├──────────────────────┼─────────────────────────────────┼────────────────────┼───────────────────────────────┤
    │ 08f1fabab502179403…  │ Bella Ciao                      │ 0.4907481898632341 │ POINT (11.06968 49.45177)     │
    │ 08f1fabab502174a03…  │ glore Nuremberg                 │ 0.9504067173970087 │ POINT (11.0699612 49.4518074) │
    │ 08f1fabab5021d4503…  │ Jimmy Ray's Barber Shop Barber  │ 0.9504067173970087 │ POINT (11.0695916 49.4520211) │
    │ 08f1fabab502110b03…  │ Beautyshop 24                   │ 0.9573247870456271 │ POINT (11.0696863 49.4519865) │
    │ 08f1fabab50256da03…  │ Thorsten Staudt Friseursalon    │ 0.9589055910024873 │ POINT (11.0696001 49.452039)  │
    │ 08f1fabab502120a03…  │ Kaffino                         │ 0.9504067173970087 │ POINT (11.0700699 49.4518776) │
    │ 08f1fabab5028d6103…  │ Unysono Immobilienkultur        │               0.77 │ POINT (11.0706167 49.4517711) │
    │ 08f1fabab502cd5c03…  │ Milan Kreitlein                 │ 0.9504067173970087 │ POINT (11.0702838 49.4519829) │
    │ 08f1fabab502cd5c03…  │ Games Garden Video Game Store…  │ 0.9573247870456271 │ POINT (11.0703253 49.4520171) │
    │ 08f1fabab502cb8403…  │ Parkhaus Wöhrl                  │               0.77 │ POINT (11.0705119 49.4521137) │
    ├──────────────────────┴─────────────────────────────────┴────────────────────┴───────────────────────────────┤
    │ 10 rows                                                                                           4 columns │
    └─────────────────────────────────────────────────────────────────────────────────────────────────────────────┘

    Notice the type of the geometry column is `geometry`. DuckDB recognizes the geo metadata in the source parquet files and automatically converts the column to a geometry type.


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
