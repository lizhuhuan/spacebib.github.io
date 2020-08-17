# MySQL 空间数据类型(Spatial Data Types)

## 数据类型

- 单值数据类型
  - GEOMETRY（基类，可以表示一下所有）
  - POINT（零维，可以表示一个点）
  - LINESTRING（一维，可以表示一条线）
  - POLYGON（二维，可以表示一个多边形）

GEOMETRY 可以存储所有的几何值

- 集合数据类型
  - MULTIPOINT
  - MULTILINESTRING
  - MULTIPOLYGON
  - GEOMETRYCOLLECTION

GEOMETRYCOLLECTION 可以存储所有的几何值集合

- SRID 属性
  指定空间参照系（[SRS](https://dev.mysql.com/doc/refman/8.0/en/spatial-reference-systems.html)）
  SRID 属性可以不指定，默认为 0，这样意味着不会限制 值，但是也无法使用索引（SPATIAL indexes）

## 获取空间数据

- 以[WKT](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry)格式(字符串)获取空间数据:

  `SELECT ST_AsText(g) FROM geom;`

- 以WKB格式(二进制)获取空间数据:

  `SELECT ST_AsBinary(g) FROM geom;`

## 存储空间数据

- [ST_GeomFromText](https://dev.mysql.com/doc/refman/8.0/en/gis-wkt-functions.html#function_st-geomfromtext)

```sql
ST_GeomFromText('POINT(1 1)', 4326)
ST_GeomFromText('LINESTRING(0 0,1 1,2 2)')
ST_GeomFromText('POLYGON((0 0,10 0,10 10,0 10,0 0),(5 5,7 5,7 7,5 7, 5 5)))
```

- [ST_GeomFromWKB](https://dev.mysql.com/doc/refman/8.0/en/gis-wkb-functions.html#function_st-geomfromwkb)

```sql
ST_GeomFromWKB(X'0101000000000000000000F03F000000000000F03F')
```

## SPATIAL 索引

SPATIAL 列必须 NOT NULL 而且 指定了 SRID。

也可以在空间列上创建普通索引。在非空间索引中，必须为除 POINT 列之外的任何空间列声明前缀。
MyISAM和InnoDB支持空间索引和非空间索引。其他存储引擎支持非空间索引。

### 创建 SPATIAL 索引

```sql
# With CREATE TABLE:
CREATE TABLE geom (g GEOMETRY NOT NULL SRID 4326, SPATIAL INDEX(g));

# With ALTER TABLE:
ALTER TABLE geom ADD SPATIAL INDEX(g);

# With CREATE INDEX:
CREATE SPATIAL INDEX g ON geom (g);
```

### 删除 SPATIAL 索引

```sql
# With ALTER TABLE:
ALTER TABLE geom DROP INDEX g;

# With DROP INDEX:
DROP INDEX g ON geom;
```

## Spatial 求地球上两点之间的直线距离

Function：ST_Distance_Sphere(g1, g2 [, radius])  

返回球体上 Point 或 MultiPoint 自变量之间的最小球面距离，以米为单位。可选的 radius 参数应以米为单位。

如果两个几何参数都是 SRID 0 中的有效笛卡尔点或多点值，则返回值是具有所提供半径的球体上两个几何之间的最短距离。如果省略，则默认半径为 6,370,986 米，即地球半径，点 X 和 Y 坐标分别解释为经度和纬度（以度为单位）。

如果两个几何参数在地理空间参考系统（SRS）中都是有效的 Point 或 MultiPoint 值，则返回值是具有所提供半径的球体上两个几何之间的最短距离。如果省略，则默认半径等于平均半径，定义为（2a + b）/ 3，其中 a 是 SRS 的半长轴，b 是 SRS 的半短轴，点 X 和 Y 坐标分别解释为纬度和经度（以度为单位）。

```sql
SELECT
    id,
    ST_Distance_Sphere(
        lat_lng,
        ST_GeomFromText('POINT(lat lng)', 4326)
    )
    AS distance
FROM geotable
ORDER BY distance;
```