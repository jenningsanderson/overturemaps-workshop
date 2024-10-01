Querying the Planet: Leveraging GeoParquet to work with global scale open geospatial data locally and in the cloud.
===

> Consisting of open data from OpenStreetMap, Meta, Esri, Microsoft, Google, and more, Overture Maps data is conflated and converted to a consistent schema before being distributed as geoparquet files in the cloud. This workshop will explore the advantages of GeoParquet and cloud-native geospatial technologies for researchers working with the data both locally and in the cloud.


The easiest way to get started with Overture data is via the explore page: [explore.overturemaps.org](https://explore.overturemaps.org).


### Prerequisites
1. Install [DuckDB](https://duckdb.org/docs/installation/?version=stable&environment=cli&platform=macos&download_method=package_manager)
2. A GIS environment of your choice.
   1. Esri ArcMap and QGIS are great options for desktop software.
   2. Most of this workshop can be visualized with [kepler.gl](kepler.gl) if you do not have either of the above software.
3. Optional: If you do not want to install DuckDB locally, you can sign up for MotherDuck, a cloud-based DuckDB.

> When launching DuckDB, be sure to specify a persistent DB, such as `duckdb my_db.duckdb`. This way if you create tables, you can access them later.

### Resources
The Overture [documentation website](https://docs.overturemaps.org/) is the primary guide for this workshop. Especially important is the [Data Schema](docs.overturemaps.org/schema).


## Part 1. Places

Construct a query for all of the places in Nürnberg:

First, obtain a bounding box of interest https://boundingbox.klokantech.com/ is a great tool for creating a bounding box. Here is one for the old city:

```python
west_limit=11.069371
south_limit=49.451761
east_limit=11.090204
north_limit=49.461126
```

A basic places query looks like this:

```sql
SELECT
    id,
    names.primary as name,
    geometry
FROM read_parquet('s3://overturemaps-us-west-2/release/2024-09-18.0/theme=places/type=place/*', filename=true, hive_partitioning=1)
WHERE
    bbox.xmin BETWEEN X AND X
    AND bbox.ymin BETWEEN Y AND Y
LIMIT 10;
```

1. Consult the [places schema](https://docs.overturemaps.org/schema/reference/places/place/) to learn more about which columns can be accessed and their data types.
2. Update the query with the proper values in the WHERE clause for X and Y.
3. Paste the query into DuckDB and run it.

You should get something similar to this:

```
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
```
#### Step 2: Use DuckDB to create common spatial data formats.

First, ensure the `spatial` extension is installed by running `install spatial;`.
Next, load the spatial extension by running `load spatial;`

Now we can use our same query again, but this time we add the COPY command to write GeoJSON. We can also remove the `LIMIT` argument.

```sql
COPY(
    <query from above>
) TO 'old_city_places.geojson' WITH (FORMAT GDAL, DRIVER 'GeoJSON')
```

Now open that GeoJSON file in your preferred GIS environment to inspect the attributes (fastest to drag-n-drop into kepler.gl)

**Bonus:** Can I just download all of the places data for Germany?

```sql
COPY(
    SELECT
    id,
    names.primary as name,
    geometry
FROM read_parquet('s3://overturemaps-us-west-2/release/2024-09-18.0/theme=places/type=place/*', filename=true, hive_partitioning=1)
WHERE
    bbox.xmin BETWEEN 5.87 AND 15.04
    AND bbox.ymin BETWEEN 47.27 AND 55.1
) TO 'germany.parquet';
```

This will write a geoparquet file to your computer. QGIS can also read GeoParquet (See these instructions if you're stuck)[https://docs.overturemaps.org/examples/QGIS/].

Additionally, you can access this file at any time with `FROM read_parquet('germany.parquet')`

## Part 2: Divisions
Now is a good time to save a Countries table for boundaries to reference later.

```sql
CREATE OR REPLACE TABLE countries AS (
    SELECT
        names.primary as name,
        names.common['en'][1] as name_en,
        names.common['de'][1] as name_de,
        geometry AS border
    FROM read_parquet('s3://overturemaps-us-west-2/release/2024-09-18.0/theme=divisions/type=division_area/*', hive_partitioning=1)
    WHERE subtype = 'country'
);
```
This might take some time, but in the end, you should high resolution boundaries from OpenStreetMap for 234 countries.

If you'd like, create a Country boundary geoparquet file for reference:
```sql
COPY(
    SELECT
        *,
    border as geometry
FROM countries
) TO 'countries.parquet';
```

```sql
COPY(
    SELECT h3_h3_to_string(h3) AS h3,
    name_en
    FROM (
        SELECT name_en,
               s.geom as geom
        FROM countries
        CROSS JOIN UNNEST(ST_DUMP(border)) AS t(s)
    )
    cross join unnest(h3_polygon_wkt_to_cells(ST_ASTEXT(geom), 4)) as t(h3)
) TO 'h3_countries.csv';
```

```sql
load spatial;
COPY(
   SELECT
   id,
   name,
   geometry
FROM read_parquet('germany.parquet')
WHERE ST_CONTAINS(
   (SELECT ST_SIMPLIFY(border, 0.1) FROM countries WHERE name_en = 'Germany'),
   geometry
   )
) TO 'places_cut.parquet';
```

#### Quick spatial analysis with h3
The [h3 grid system](https://h3geo.org/) is available in duckdb via the h3 plugin:
```sql
install h3 from community;
load h3;
```

```sql
COPY(
    SELECT
        h3_latlng_to_cell_string(ST_Y(geometry), ST_X(geometry), 8) as h3,
        count(1) as _count
FROM read_parquet('germany_places.parquet')
GROUP BY h3_latlng_to_cell_string(ST_Y(geometry), ST_X(geometry), 8)
) TO 'h3.csv';
```

Now drag-n-drop this result directly into kepler.gl, which will know the geometry from cell.


## Part 3:  Buildings
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

Building Densities

```sql
COPY(
    SELECT
        h3_latlng_to_cell_string(ST_Y(ST_CENTROID(geometry)), ST_X(ST_CENTROID(geometry)), 10) as h3,
        count(1) as _count
FROM read_parquet('buildings.parquet')
GROUP BY h3_latlng_to_cell_string(ST_Y(ST_CENTROID(geometry)), ST_X(ST_CENTROID(geometry)), 10)
) TO 'buildings_h3.csv';
```

## Part 4: Transportation

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

## Part 5: Base
What is the base theme?

```sql
SET s3_region='us-west-2';

COPY(
    SELECT
       id,
       names.primary as name,
       elevation,
       geometry
    FROM read_parquet('s3://overturemaps-us-west-2/release/2024-09-18.0/theme=base/type=land/*', filename=true, hive_partitioning=1)
    WHERE subtype = 'physical' AND class IN ('peak','volcano') AND elevation IS NOT NULL
    AND bbox.xmin BETWEEN -12.8 AND 29.7
    AND bbox.ymin BETWEEN 34.7 AND 58.2
) TO 'european_peaks.parquet';
```


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
