---
layout: post
title: Where geohash breaks down
---

Geohash is the first spatial index most people reach for, and for good reason: it turns a coordinate pair into a string, and string prefixes give you containment for free. It is also the reason a lot of "find players near me" queries quietly return the wrong answer.

## The prefix property

A geohash encodes latitude and longitude by interleaving their bits, then base32-encoding the result. Two points that share a prefix fall in the same cell:

```python
def encode(lat, lon, precision=9):
    lat_range, lon_range = [-90.0, 90.0], [-180.0, 180.0]
    bits, even = [], True
    while len(bits) < precision * 5:
        rng = lon_range if even else lat_range
        mid = sum(rng) / 2
        val = lon if even else lat
        bits.append(1 if val > mid else 0)
        rng[0 if val > mid else 1] = mid
        even = not even
    return to_base32(bits)
```

So `wx4g0ec19` and `wx4g0ec1x` are neighbours, and a `LIKE 'wx4g0ec%'` scan finds everything in that cell. This is genuinely useful and it is why geohash spread so widely.

## Where it falls apart

The problem is that **cell adjacency and string adjacency are not the same thing**. The bit interleaving traces a Z-order curve through space, and a Z-order curve has seams.

> Two points a metre apart can sit on opposite sides of a seam and share no prefix at all. Two points that share a seven-character prefix can be 150 metres apart.

Concretely, a player standing just north of the equator and a player just south of it differ in the very first bit of latitude. Their geohashes diverge at character one. A prefix query finds neither from the other.

### The usual workaround

You compute the eight neighbouring cells and query all nine:

1. Encode the query point
2. Derive the eight neighbours by incrementing and decrementing the interleaved bits
3. Run nine prefix scans
4. Filter the union by true haversine distance

It works. It is also nine queries instead of one, and it still gets the recall wrong near the poles, where cells become extremely non-square.

## What we use instead

| Scheme | Cell shape | Neighbours | Area variance |
| --- | --- | --- | --- |
| Geohash | Rectangular, distorted by latitude | 8, computed | Very high |
| S2 | Projected cube face, near-square | 4 or 8, cheap | About 2x |
| H3 | Hexagonal | 6, uniform distance | About 2x |

We settled on [S2](https://s2geometry.io/) because the cell IDs are 64-bit integers rather than strings, which means the index is an ordinary `BIGINT` B-tree, and because `S2RegionCoverer` will hand you a covering of an arbitrary region at a bounded cell count. Hexagons are prettier, but every hexagon neighbour being equidistant mattered less to us than integer keys.

---

None of this makes geohash wrong. If your query radius is fixed and your data is not near a seam, it is the simplest thing that works. Just do not assume the prefix property means what it looks like it means.

More on the query side in a later post. Back to [all writing]({{ '/writing/' | relative_url }}).
