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

TODO: figure out how to map efficiently...
https://observablehq.com/@kylebarron/geoparquet-on-the-web

