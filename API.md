## Census Reporter API

Think of the American Community Survey as a spreadshet with thousands of columns and hundreds of thousands of rows (geographies). At the intersection of each of these is a The goal of this API is to make it easy to access any chunk of that spreadsheet with simple HTTP calls and get the result as an easy-to-parse JSON object.

To continue with the spreadsheet metaphor, the endpoints for this API can be roughly broken into 3 pieces:

1. information about columns
2. information about geographies (rows)
3. data at the intersection of one or more of the above

Endpoints labeled *experimental* depend on our project to implement ElasticSearch for CensusReporter data, and may change API and/or return imperfect results. 

### Column, Table, and Tabulations

#### `GET /1.0/table/suggest`
*experimental*

 Query Argument | Type   | Required? | Description
:---------------|:-------|:----------|:-----------
 `q`            | string | Yes       | The string to use for suggestions.

Retrieves a list of table and column suggestions given a search term using `q`. This is meant to support autocomplete text boxes.

Examples:
```bash
$ curl "http://api.censusreporter.org/1.0/table/suggest?q=pover"
{
    "results": [
        {
            "table_id": "B17001",
            "table_title": "Poverty Status in the Past 12 Months by Sex by Age"
        },
        ...
    ]
}

$ curl "http://api.censusreporter.org/1.0/table/suggest?q=esoteric+word"
{
    "results": []
}
```

#### `GET /1.0/table/elasticsearch`
*experimental*

 Query Argument | Type   | Required? | Description
:---------------|:-------|:----------|:-----------
 `q`            | string | Yes       | The string to use for suggestions.
 `start`        | int    | No        | Where in the results list to start.
 `size`         | int    | No        | The number of results to return.
 `topics`       | string | No        | A column-separated list of topics to limit the search by.
 `acs`          | string | No        | The ACS release code to limit the search by.

Returns table and column information relevant to the search term given in `q`. This endpoint supports paging using the `start` and `size` parameters. You can narrow your search by specifying a `topics` parameter. The `acs` parameter limits your search to a specific ACS release. By default it will use the most recent release.

Examples:
```bash
$ curl "http://api.censusreporter.org/1.0/table/elasticsearch?q=children+in+poverty"
{
    "facets": {
        "topics": {
            "_type": "terms",
            "total": 14,
            "terms": [
                {
                    "count": 3,
                    "term": "children"
                },
                ...
            ],
            "other": 0,
            "missing": 0
        }
    },
    "results": [
        {
            "table_title": "Poverty Status in the Past 12 Months of Related Children Under 18 Years by Family Type by Age of Related Children Under 18 Years",
            "uid": "acs2012_5yr_B17006",
            "universe": "Related Children Under 18 Years",
            "denominator_column_id": "B17006001",
            "table_id": "B17006",
            "release": "acs2012_5yr",
            "topics": [
                "families",
                "age",
                "family type",
                "poverty",
                "children"
            ]
        },
        ...
    ]
}
```

#### `GET /1.0/tabulation/<tabulation_id>`

 URL Argument    | Type   | Required? | Description
:----------------|:-------|:----------|:-----------
 `tabulation_id` | string | Yes       | The tabulation ID to retrieve.

Returns information about the specified tabulation. A tabulation is a grouping of tables that share the numeric part of the table ID. Each tabulation can have multiple tables spread across the three yearly American Community Survey releases (1-, 3-, and 5-year).

Examples:
```bash
$ curl "http://api.censusreporter.org/1.0/tabulation/01001"
{
    "tabulation_code": "01001",
    "table_title": "Sex by Age",
    "tables_by_release": {
        "five_yr": [
            "B01001",
            ...
            "B01001I"
        ],
        "one_yr": [
            "B01001",
            ...
            "C01001I"
        ],
        "three_yr": [
            "B01001",
            ...
            "C01001I"
        ]
    },
    "universe": "Total Population",
    "topics": [
        "age",
        "sex"
    ],
    "subject_area": "Age-Sex",
    "simple_table_title": "Sex by Age"
}
```

#### `GET /1.0/table/<table_id>`

 URL Argument    | Type   | Required? | Description
:----------------|:-------|:----------|:-----------
 `table_id`      | string | Yes       | The table ID to retrieve.


 Query Argument | Type   | Required? | Description
:---------------|:-------|:----------|:-----------
 `acs`          | string | No        | The ACS release to use. Defaults to the most recent version.

Returns information about the specified table in the specified release. Information returned includes the table's title, subject area, universe, a list of topics covered by the table, and a list of columns contained in the table.

Examples:
```bash
$ curl "http://api.censusreporter.org/1.0/table/B01001A"
{
    "table_id": "B01001A",
    "table_title": "Sex by Age (White Alone)",
    "simple_table_title": "Sex by Age (White Alone)",
    "subject_area": "Age-Sex",
    "universe": "People Who Are White Alone",
    "denominator_column_id": "B01001A001",
    "topics": [
        "age",
        "race",
        "sex"
    ],
    "columns": {
        "B01001A001": {
            "column_title": "Total:",
            "indent": 0,
            "parent_column_id": null
        },
        "B01001A002": {
            "column_title": "Male:",
            "indent": 1,
            "parent_column_id": "B01001A001"
        },
        ...
    }
}
```

### Geography

#### `GET /1.0/geo/suggest`
*experimental*

 Query Argument | Type   | Required? | Description
:---------------|:-------|:----------|:-----------
 `q`            | string | Yes       | The string to use for suggestions.

Retrieves a list of geography suggestions given a search term using `q`. This is meant to support autocomplete text boxes.

Examples:
```bash
$ curl "http://api.censusreporter.org/1.0/geo/suggest?q=chicag"
{
    "results": [
        {
            "geoid": "16000US1714000",
            "name": "Chicago, IL"
        },
        {
            "geoid": "06000US1703114000",
            "name": "Chicago city, Cook County, IL"
        },
        {
            "geoid": "31000US16980",
            "name": "Chicago-Joliet-Naperville, IL-IN-WI Metro Area"
        },
        {
            "geoid": "33000US176",
            "name": "Chicago-Naperville-Michigan City, IL-IN-WI CSA"
        },
        {
            "geoid": "16000US1714026",
            "name": "Chicago Heights, IL"
        }
    ]
}
```

#### `GET /1.0/geo/elasticsearch`
*experimental*

 Query Argument | Type   | Required? | Description
:---------------|:-------|:----------|:-----------
 `q`            | string | Yes       | The string to use for suggestions.
 `start`        | int    | No        | Where in the results list to start.
 `size`         | int    | No        | The number of results to return.
 `sumlevs`      | string | No        | A column-separated list of 3-digit summary level codes to limit the search by.

Returns geography information relevant to the search term given in `q`. This endpoint supports paging using the `start` and `size` parameters. You can narrow your search by specifying a comma-separated list of 3-digit summary levels with the `sumlevs` parameter.

Examples:
```bash
$ curl "http://api.censusreporter.org/1.0/geo/elasticsearch?q=evanston+il"
{
    "facets": {
        "sumlev": {
            "_type": "terms",
            "total": 1,
            "terms": [
                {
                    "count": 1,
                    "term": "160"
                }
            ],
            "other": 0,
            "missing": 0
        }
    },
    "results": [
        {
            "sumlev": "160",
            "population": 74619,
            "display_name": "Evanston, IL",
            "importance": 141.312802096439,
            "awater": 57390,
            "aland": 20148006,
            "full_geoid": "16000US1724582",
            "geoid": "1724582",
            "location": [
                -87.6943525,
                42.0463909
            ]
        }
    ]
}
```

#### `GET /1.0/geo/tiger2012/tiles/<sumlevel>/<zoom>/<x>/<y>.geojson`

 URL Argument    | Type   | Required? | Description
:----------------|:-------|:----------|:-----------
 `sumlevel`      | string | Yes       | The summary to use in the tile.
 `zoom`          | int    | Yes       | The zoom level for the tile.
 `x`             | int    | Yes       | The x value for the tile.
 `y`             | int    | Yes       | The y value for the tile.

Returns a [GeoJSON](http://geojson.org/) representation of all geographies at summary level `sumlevel` and contained within a [map tile](http://www.maptiler.org/google-maps-coordinates-tile-bounds-projection/) specified by the `zoom`, `x`, and `y` parameters. You can use this to create a map of Census geographies on top of an existing map. The returned GeoJSON data includes attributes for the name and geoid of the geography.

#### `GET /1.0/geo/tiger2012/<geoid>`

 URL Argument    | Type   | Required? | Description
:----------------|:-------|:----------|:-----------
 `geoid`         | string | Yes       | The geography identifier to retrieve data for.

 Query Argument | Type   | Required? | Description
:---------------|:-------|:----------|:-----------
 `geom`         | bool   | No        | Whether or not to include the geography portion of the GeoJSON.

Returns a [GeoJSON](http://geojson.org/) representation of the specified Census geography specified by the `geoid` parameter. By default, the returned GeoJSON only contains the attributes for the geography (including the land and water area, name, and geography ID). You can include the geography by setting the `geom` query argument to `true`. Note that this will usually make the response significantly larger, but will allow you to draw it on a map.

Examples:
```bash
$ curl "http://api.censusreporter.org/1.0/geo/tiger2012/04000US55"
{
    "geometry": null,
    "type": "Feature",
    "properties": {
        "awater": 29365986992,
        "display_name": "Wisconsin",
        "simple_name": "Wisconsin",
        "sumlevel": "040",
        "population": 5664893,
        "full_geoid": "04000US55",
        "aland": 140268861626
    }
}

$ curl "http://api.censusreporter.org/1.0/geo/tiger2012/04000US55?geom=true"
{
    "geometry": {
        "type": "MultiPolygon",
        "coordinates": [
            [
                [
                    [
                        -92.674543,
                        45.382868
                    ],
                    ...
                ]
            ]
        ]
    },
    "type": "Feature",
    "properties": {
        "awater": 29365986992,
        "display_name": "Wisconsin",
        "simple_name": "Wisconsin",
        "sumlevel": "040",
        "population": 5664893,
        "full_geoid": "04000US55",
        "aland": 140268861626
    }
}
```

#### `GET /1.0/geo/tiger2012/<geoid>/parents`

 URL Argument    | Type   | Required? | Description
:----------------|:-------|:----------|:-----------
 `geoid`         | string | Yes       | The geography identifier to retrieve parent geographies for.

Returns a list of geographies that might be considered the parent of the specified geography. The information returned includes the name, geoid, and summary level code for each geography.

In some cases, the requested geography sits in multiple parents of the same summary level. In these cases, the `coverage` attribute returned for each parent of the same summary level will specify how much of this geography sits inside that parent geography (in percent).

This endpoint will also return the specified geography with a `relation` of `this`.

Examples:
```bash
$ curl "http://api.censusreporter.org/1.0/geo/tiger2012/04000US55/parents"
{
    "parents": [
        {
            "sumlevel": "040",
            "relation": "this",
            "coverage": 100,
            "display_name": "Wisconsin",
            "geoid": "04000US55"
        },
        {
            "sumlevel": "010",
            "relation": "nation",
            "coverage": 100,
            "display_name": "United States",
            "geoid": "01000US"
        }
    ]
}

$ curl "http://api.censusreporter.org/1.0/geo/tiger2012/16000US1714000/parents"
{
    "parents": [
        {
            "sumlevel": "160",
            "relation": "this",
            "coverage": 100,
            "display_name": "Chicago, IL",
            "geoid": "16000US1714000"
        },
        {
            "sumlevel": "050",
            "relation": "county",
            "coverage": 0.973951,
            "display_name": "DuPage County, IL",
            "geoid": "05000US17043"
        },
        {
            "sumlevel": "050",
            "relation": "county",
            "coverage": 99.026,
            "display_name": "Cook County, IL",
            "geoid": "05000US17031"
        },
        {
            "sumlevel": "310",
            "relation": "CBSA",
            "coverage": 100,
            "display_name": "Chicago-Joliet-Naperville, IL-IN-WI Metro Area",
            "geoid": "31000US16980"
        },
        {
            "sumlevel": "040",
            "relation": "state",
            "coverage": 100,
            "display_name": "Illinois",
            "geoid": "04000US17"
        },
        {
            "sumlevel": "010",
            "relation": "nation",
            "coverage": 100,
            "display_name": "United States",
            "geoid": "01000US"
        }
    ]
}
```

#### `GET /1.0/geo/show/tiger2012`

 Query Argument | Type   | Required? | Description
:---------------|:-------|:----------|:-----------
 `geo_ids`      | string | Yes       | A comma-separated list of geographies to request information about.

Returns a [GeoJSON](http://geojson.org/) representation of the specified comma-separated list of Census geographies. Each item in the comma-separated list can either be a single geoid or a "geoid grouping" specified by `<child summary level>|<parent geoid>`. A grouping is a shortcut so you don't have to specify individual geoids for contiguous groups of geographies. For example, to get states (summary level `040`) in the United States (geoid `01000US`), you'd use `040|01000US` as an element in your `geo_ids` list.

The data included will always have the geography data included. Unlike the single-geography request above you cannot disable geography representation in the response.

The attributes in the response will only include the geography name and the geoid.

### Data Retrieval

#### `GET /1.0/data/show/<acs>`

 URL Argument    | Type   | Required? | Description
:----------------|:-------|:----------|:-----------
 `acs`           | string | Yes       | The release to use for this data.

 Query Argument | Type   | Required? | Description
:---------------|:-------|:----------|:-----------
 `table_ids`    | string | Yes       | A comma-separated list of table IDs to request data for.
 `geo_ids`      | string | Yes       | A comma-separated list of geographies to request information about.

Returns the data for the given comma-separated list of table IDs in the given geo IDs. The data includes basic information about the specified tables and geographies along with the estimate and error data.

The `acs` parameter specifies which release to use. If you aren't sure, use the word `latest` and we will pick the most recent release that contains data for all the tables across lal the geographies you asked for.

Examples:
```bash
$ curl "http://api.censusreporter.org/1.0/data/show/latest?table_ids=B13016&geo_ids=04000US55"
{
    "release": {
        "id": "acs2012_1yr",
        "name": "ACS 2012 1-year",
        "years": "2012"
    },
    "tables": {
        "B13016": {
            "title": "Women 15 to 50 Years Who Had a Birth in the Past 12 Months by Age",
            "universe": "Women 15 to 50 Years",
            "denominator_column_id": "B13016001",
            "columns": {
                "B13016001": {
                    "name": "Total:",
                    "indent": 0
                },
                ...
            }
        }
    },
    "data": {
        "04000US55": {
            "B13016": {
                "estimate": {
                    "B13016001": 1336183,
                    ...
                },
                "error": {
                    "B13016001": 3559,
                    ...
                }
            }
        }
    },
    "geography": {
        "04000US55": {
            "name": "Wisconsin"
        }
    }
}
```