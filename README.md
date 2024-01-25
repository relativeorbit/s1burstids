# s1burstids
sentinel1 burst ids

from https://sentinels.copernicus.eu/web/sentinel/-/publication-of-brust-id-maps-for-copernicus-sentinel-1

USAGE: 

```python
url = 'https://github.com/relativeorbit/s1burstids/raw/main/burst_map_IW_000001_375887_brotli.parquet'
with fsspec.open(url) as file:
    gfA = gpd.read_parquet(file)
```

```
<class 'geopandas.geodataframe.GeoDataFrame'>
RangeIndex: 1127661 entries, 0 to 1127660
Data columns (total 6 columns):
 #   Column                 Non-Null Count    Dtype   
---  ------                 --------------    -----   
 0   burst_id               1127661 non-null  int32   
 1   subswath_name          1127661 non-null  object  
 2   relative_orbit_number  1127661 non-null  int16   
 3   time_from_anx_sec      1127661 non-null  float32 
 4   orbit_pass             1127661 non-null  object  
 5   geometry               1127661 non-null  geometry
dtypes: float32(1), geometry(1), int16(1), int32(1), object(2)
memory usage: 36.6+ MB
```


### Creation notes

experimenting with https://github.com/opengeospatial/geoparquet

```python
In [34]: %time gf = gpd.read_file('burst_map_IW_000001_375887.sqlite3')
CPU times: user 53.6 s, sys: 1.3 s, total: 54.9 s
Wall time: 55.5 s
```

```
gf['burst_id'] = gf.burst_id.astype(np.int32)
gf['subswath_name'] = gf.subswath_name.astype('|S') #sets max string length
gf['relative_orbit_number'] = gf.relative_orbit_number.astype(np.int16)
gf['time_from_anx_sec'] = gf.time_from_anx_sec.astype(np.float32)
gf['orbit_pass'] = gf.orbit_pass.astype('|S') #}S10
```

```
def simplify(multipolyz):
  return shapely.ops.transform(lambda x, y, z: (x, y), multipolyz.geoms[0])
  
gf['geometry'] = gf.geometry.apply(simplify)


gf.to_parquet('burst_map_IW_000001_375887_brotli.parquet', compression='brotli', index=False)
```

```
-rwxrwxrwx@ 1 scott  staff   279M Jun  8 11:13 burst_map_IW_000001_375887.sqlite3
-rw-r--r--  1 scott  staff   375M Jun 29 14:20 burst_map_IW_000001_375887.gpkg
-rw-r--r--  1 scott  staff   125M Jun 29 14:25 burst_map_IW_000001_375887.parquet
-rw-r--r--  1 scott  staff    74M Jun 29 16:05 burst_map_IW_000001_375887_brotli.parquet
```

### Update 1/25/2024

Used OGR CLI (3.8.3) to create SOZIP gpkg:
```
time ogr2ogr burst_map_IW_000001_375887.gpkg burst_map_IW_000001_375887.sqlite3
# 7.09s user 0.99s system 100% cpu 8.004 total
time sozip burst_map_IW_000001_375887.gpkg.zip burst_map_IW_000001_375887.gpkg
# 11.32s user 0.60s system 851% cpu 1.400 total
```

```
# Best performance if you're able to hone in on a subregion first
gf = gpd.read_file('burst_map_IW_000001_375887.gpkg.zip', bbox=[80.0, 26.3, 88.2, 30.5])
gf

     burst_id subswath_name  relative_orbit_number  time_from_anx_sec  orbit_pass                                           geometry
0      180577           IW2                     85         417.029296   ASCENDING  MULTIPOLYGON Z (((85.37449 25.98202 0.00000, 8...
1      180578           IW2                     85         419.787569   ASCENDING  MULTIPOLYGON Z (((85.33970 26.14839 0.00000, 8...
2      180579           IW2                     85         422.545842   ASCENDING  MULTIPOLYGON Z (((85.30489 26.31477 0.00000, 8...
```


