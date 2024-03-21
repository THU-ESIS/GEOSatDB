<!-- @import "[TOC]" {cmd="toc" depthFrom=2 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [1. Namespaces](#1-namespaces)
- [2. Satellite property](#2-satellite-property)
  - [2.1. name and alternateName](#21-name-and-alternatename)
  - [2.2. launchDate](#22-launchdate)
  - [2.3. Owner](#23-owner)
    - [2.3.1. Owner count](#231-owner-count)
    - [2.3.2. Designated owner](#232-designated-owner)
  - [2.4. Orbit Type](#24-orbit-type)
    - [2.4.1. Orbit Type count](#241-orbit-type-count)
    - [2.4.2. Designated Orbit Type](#242-designated-orbit-type)
- [3. Sensor property](#3-sensor-property)
  - [3.1. name and alternateName](#31-name-and-alternatename)
  - [3.2. Satellites and on-board sensors](#32-satellites-and-on-board-sensors)
  - [3.3. waveband region](#33-waveband-region)
  - [3.4. sensor type](#34-sensor-type)
  - [3.5. Wavelength/Frequency](#35-wavelengthfrequency)
  - [3.6. Horizontal resolution (m)](#36-horizontal-resolution-m)
- [4. Use cases](#4-use-cases)
  - [4.1. Orbiting satellites in a specific waveband](#41-orbiting-satellites-in-a-specific-waveband)
  - [4.2. Number of orbiting satellites with NIR capability](#42-number-of-orbiting-satellites-with-nir-capability)
  - [4.3. Maximum horizontal resolution of microwave sensors](#43-maximum-horizontal-resolution-of-microwave-sensors)
  - [4.4. Superior TIR resolution compared to Landsat-9 TIRS](#44-superior-tir-resolution-compared-to-landsat-9-tirs)
    - [4.4.1. Subqueries](#441-subqueries)
    - [4.4.2. Other](#442-other)

<!-- /code_chunk_output -->

# SPARQL query examples for GEOSatDB<!-- omit from toc -->

## 1. Namespaces

```sparql
PREFIX eo-ont: <https://www.eoknowledgehub.cn/eo/ontology/>
PREFIX mrc: <https://schemas.isotc211.org/19115/-1/mrc/1.3.0/>
PREFIX mac: <https://schemas.isotc211.org/19115/-2/mac/2.2/>
PREFIX smi: <https://schemas.isotc211.org/19130/-3/smi/1.1/>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX schema: <http://schema.org/>
PREFIX skos: <http://www.w3.org/2004/02/skos/core#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
```

## 2. Satellite property

### 2.1. name and alternateName

```sparql
select DISTINCT ?satellite ?name ?launchDate where {
    ?satellite a eo-ont:Satellite ;
               schema:name ?name;
               schema:alternateName ?alternateName;
               eo-ont:launchDate ?launchDate.
    FILTER(CONTAINS(LCASE(?name),"landsat")||
        CONTAINS(LCASE(?alternateName),"landsat"))
}
ORDER BY DESC(?launchDate)
```

### 2.2. launchDate

```sparql
select DISTINCT ?satellite ?name ?launchDate where {
    ?satellite a eo-ont:Satellite ;
               schema:name ?name;
               schema:alternateName ?alternateName;
               eo-ont:launchDate ?launchDate.
    FILTER(?launchDate>="2023-01-01"^^xsd:dateTime && ?launchDate<"2024-01-01"^^xsd:dateTime)
}
ORDER BY DESC(?launchDate)
```

### 2.3. Owner

#### 2.3.1. Owner count

```sparql
select ?owner ?owner_name (COUNT(DISTINCT ?satellite) AS ?count) where {
    ?satellite a eo-ont:Satellite ;
               eo-ont:owner ?owner.
    ?owner rdfs:label ?owner_name.
}
GROUP BY ?owner ?owner_name
ORDER BY DESC(?count)
```

#### 2.3.2. Designated owner

```sparql
select DISTINCT ?satellite ?name ?launchDate ?owner ?owner_name where {
    ?satellite a eo-ont:Satellite ;
               schema:name ?name;
               eo-ont:launchDate ?launchDate;
               eo-ont:owner ?owner.
    ?owner rdfs:label ?owner_name.
#    FILTER(LCASE(?owner_name)="china"@en)
    FILTER(CONTAINS(LCASE(?owner_name),"china"))
}
ORDER BY DESC(?launchDate)
```

### 2.4. Orbit Type

#### 2.4.1. Orbit Type count

```sparql
select ?orbitType ?orbitType_name (COUNT(DISTINCT ?satellite) AS ?count) where {
    ?satellite a eo-ont:Satellite ;
               eo-ont:orbitType ?orbitType.
    ?orbitType rdfs:label ?orbitType_name.
}
GROUP BY ?orbitType ?orbitType_name
ORDER BY DESC(?count)
```

#### 2.4.2. Designated Orbit Type

```sparql
select DISTINCT ?satellite ?name ?launchDate where {
    ?satellite a eo-ont:Satellite ;
               schema:name ?name;
               eo-ont:launchDate ?launchDate;
               eo-ont:orbitType eor:orbit_type.LLEO_S.
}
ORDER BY DESC(?launchDate)
```

## 3. Sensor property

### 3.1. name and alternateName

```sparql
select DISTINCT ?sensor ?name where {
    ?sensor a eo-ont:Sensor ;
               schema:name ?name;
               schema:alternateName ?alternateName.
    FILTER(CONTAINS(LCASE(?name),"modis")||
        CONTAINS(LCASE(?alternateName),"modis"))
}
```

### 3.2. Satellites and on-board sensors

```sparql
select DISTINCT ?satellite ?satellite_name ?sensor ?sensor_name where {
    ?satellite a eo-ont:Satellite ;
               schema:name ?satellite_name ;
               eo-ont:launchDate ?launchDate.
    ?sensor mac:mountedOn ?satellite ;
            schema:name ?sensor_name .
}
ORDER BY DESC(?launchDate)
```

### 3.3. waveband region

```sparql
select ?wavebandRegion ?wavebandRegion_name (COUNT(DISTINCT ?sensor) AS ?sensor_count) where {
    ?sensor a eo-ont:Sensor ;
            eo-ont:operationalBand ?operationalBand .
    ?operationalBand eo-ont:wavebandRegion ?wavebandRegion.
    ?wavebandRegion rdfs:label ?wavebandRegion_name.
}
GROUP BY ?wavebandRegion ?wavebandRegion_name
ORDER BY DESC(?sensor_count)
```

### 3.4. sensor type

```sparql
select ?sensorType ?sensorType_name (COUNT(DISTINCT ?sensor) AS ?sensor_count) where {
    ?sensor a eo-ont:Sensor ;
            eo-ont:sensorType ?sensorType .
    ?sensorType rdfs:label ?sensorType_name.
}
GROUP BY ?sensorType ?sensorType_name
ORDER BY DESC(?sensor_count)
```

### 3.5. Wavelength/Frequency

```sparql
select DISTINCT ?sensor ?sensor_name ?boundMin ?boundMax where {
    ?sensor a eo-ont:Sensor ;
            schema:name ?sensor_name ;
            eo-ont:operationalBand ?operationalBand.
    ?operationalBand mrc:boundUnit "micrometer"@en ;
                     mrc:boundMin ?boundMin ;
                     mrc:boundMax ?boundMax .
    FILTER((?boundMin>=0.40) && (?boundMax<=0.75))
}
ORDER BY DESC(?sensor)
```

### 3.6. Horizontal resolution (m)

```sparql
select DISTINCT ?sensor ?sensor_name ?wavebandRegion ?horizontalResolution where {
    ?sensor a eo-ont:Sensor ;
            schema:name ?sensor_name ;
            eo-ont:operationalBand ?operationalBand.
    ?operationalBand eo-ont:operation ?operation ;
                     eo-ont:wavebandRegion ?wavebandRegion .
    ?operation eo-ont:horizontalResolution ?horizontalResolution.
    FILTER(?horizontalResolution<100)
}
ORDER BY DESC(?horizontalResolution)
```

## 4. Use cases

### 4.1. Orbiting satellites in a specific waveband

```sparql
select DISTINCT ?satellite ?satellite_name where {
    ?satellite a eo-ont:Satellite ;
               schema:name ?satellite_name ;
               eo-ont:operationalStatus ?operationalStatus .
    FILTER (?operationalStatus in (eor:operational_status.operational, eor:operational_status.partially_operational, eor:operational_status.spare, eor:operational_status.extended_mission)) .
    ?sensor mac:mountedOn ?satellite ;
            schema:name ?sensor_name ;
            eo-ont:operationalBand ?operationalBand .
    ?operationalBand eo-ont:wavebandRegion eor:waveband_region.Red.
}
```

### 4.2. Number of orbiting satellites with NIR capability

```sparql
select DISTINCT ?satellite ?satellite_name ?sensor ?sensor_name where {
    ?satellite a eo-ont:Satellite ;
               schema:name ?satellite_name ;
               eo-ont:operationalStatus ?operationalStatus .
    FILTER (?operationalStatus in (eor:operational_status.operational, eor:operational_status.partially_operational, eor:operational_status.spare, eor:operational_status.extended_mission)) .
    ?sensor mac:mountedOn ?satellite ;
            schema:name ?sensor_name ;
            eo-ont:operationalBand ?operationalBand .
    ?operationalBand eo-ont:wavebandRegion eor:waveband_region.NIR .
}
```

### 4.3. Maximum horizontal resolution of microwave sensors

```sparql
SELECT ?satellite ?satellite_name ?sensor ?sensor_name ?launchDate ?year ?maxRes WHERE {
    {
        SELECT ?sensor (MIN(?band_tir_res) AS ?maxRes) WHERE {
            ?sensor rdf:type eo-ont:Sensor;
                    schema:name ?sensor_name;
                    eo-ont:operationalBand ?band.
            ?band mrc:boundUnit "gigahertz"@en;
                                              eo-ont:operation ?band_tir_op.
            ?band_tir_op eo-ont:horizontalResolution ?band_tir_res.
        }
        GROUP BY ?sensor
    }
    ?sensor mac:mountedOn ?satellite;
            schema:name ?sensor_name.
    ?satellite schema:name ?satellite_name;
               eo-ont:launchDate ?launchDate.
    BIND(YEAR(?launchDate) AS ?year)
}
order by ?maxRes
```

### 4.4. Superior TIR resolution compared to Landsat-9 TIRS

#### 4.4.1. Subqueries

```sparql
SELECT DISTINCT ?satellite ?satellite_name ?sensor ?sensor_name ?band_tir_res WHERE {
    # Spatial resolution of Landsat-9 TIRS
    {
        SELECT ?tirs_band_res WHERE {
            ?landsat_9 rdf:type eo-ont:Satellite;
                       schema:name ?name;
                       schema:alternateName ?alternateName.
            FILTER(((LCASE(?name)) = "landsat-9"@en) || 
                ((LCASE(?alternateName)) = "landsat-9"@en))
            ?tirs mac:mountedOn ?landsat_9;
                  eo-ont:operationalBand ?tirs_band.
            ?tirs_band eo-ont:wavebandRegion eor:waveband_region.TIR ;
                       eo-ont:operation ?tirs_band_op.
            ?tirs_band_op eo-ont:horizontalResolution ?tirs_band_res.
        }
    }
    # Spatial resolution for all thermal infrared
    ?satellite rdf:type eo-ont:Satellite;
               schema:name ?satellite_name.
    ?sensor mac:mountedOn ?satellite;
            schema:name ?sensor_name;
            eo-ont:operationalBand ?band.
    ?band eo-ont:wavebandRegion eor:waveband_region.TIR ;
          eo-ont:operation ?band_tir_op.
    ?band_tir_op eo-ont:horizontalResolution ?band_tir_res.
    FILTER(?band_tir_res <= ?tirs_band_res)
}
```

#### 4.4.2. Other

```sparql
select DISTINCT ?satellite ?satellite_name ?sensor ?sensor_name ?waveband_tir_resolution where {
    ?l8 a eo-ont:Satellite ;
        schema:name "LANDSAT 8"@en .
    ?l8_ins mac:mountedOn ?l8 ;
            eo-ont:operationalBand ?l8_ins_band .
    ?l8_ins_band eo-ont:wavebandRegion eor:waveband_region.TIR ;
                 eo-ont:operation ?l8_ins_band_mode .
    ?l8_ins_band_mode eo-ont:horizontalResolution ?l8_tir_resolution .
    ?satellite a eo-ont:Satellite ;
               schema:name ?satellite_name .
    ?sensor mac:mountedOn ?satellite ;
            schema:name ?sensor_name ;
            eo-ont:operationalBand ?waveband .
    ?waveband eo-ont:wavebandRegion eor:waveband_region.TIR ;
              eo-ont:operation ?waveband_mode .
    ?waveband_mode eo-ont:horizontalResolution ?waveband_tir_resolution .
    FILTER (?waveband_tir_resolution <= ?l8_tir_resolution) .
}
```
