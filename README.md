# Pygeoapi

## Reference

Starter example: [https://github.com/geopython/pygeoapi/tree/master/docker/examples/simple](link)

## Start db and pygeoapi

```shell
docker-compose up
```

> Note: pygeoapi config references data tables that don't exist yet. Load the data before testing
> 
Access the db directly:
```shell
export PGPASSWORD=password
psql -h localhost -p 5433 -U postgres postgres 
```

## Load Test data

Load dc 2021 tabblocks from [TIGER](https://www.census.gov/cgi-bin/geo/shapefiles/index.php?year=2021&layergroup=Blocks+%282020%29)
```shell
# save archive to the testdata directory
cd testdata
unar ./tl_2021_dc_tabblock20.zip
shp2pgsql -s 4269 -g the_geom ./tl_2021_11_tabblock20/tl_2021_11_tabblock20.shp blocks | psql -h localhost -p 5433 -U postgres postgres
```

Load an osm shapefile that's more complex
Pulled from [geofabrik](https://download.geofabrik.de/north-america/us/district-of-columbia-latest-free.shp.zip)
```shell
# save archive to the testdata directory
cd testdata
unar ./district-of-columbia-latest-free.shp.zip
shp2pgsql -s 4269 -g the_geom ./district-of-columbia-latest-free.shp/gis_osm_buildings_a_free_1.shp buildings | psql -h localhost -p 5433 -U postgres postgres
```

Add datetime column
```shell
psql -h localhost -p 5433 -U postgres postgres -c "
ALTER TABLE buildings ADD COLUMN created_at TIMESTAMP DEFAULT NOW()"
```
Set top 100 rows timestamp to yesterday
```shell
psql -h localhost -p 5433 -U postgres postgres -c "
UPDATE buildings
SET created_at = (created_at - INTERVAL '1 day')::date
WHERE gid IN (SELECT gid
             FROM buildings
             ORDER BY gid ASC
             LIMIT 100);"
```

## TODO 
- [ ] mapping of layers
- [ ] test datetime/filtering