# awk implementation of geohash

requirements: awk > 4.1 (support for the `--bignum` flag)

Inspired by [a blog post](https://yatmanwong.medium.com/geohash-implementation-explained-2ed9627a61ff).

`geohash` accepts all arguments permitted by awk as well as the following:
  * `lat`, `lon` (columns number with latitudes and longitudes)
  * `precision` (number of characters in the geohash)
  * `geohash` (column number with a geohash)

`geohash` accepts a text file with columns of data. To encode
coordinates, specify `lat` and `lon` columns (`precision` is
optional). To decode geohash to coordinates specify `geohash` column.
The output columns are added to the original data and printed in
stdout. lat and lon are printed with 16 digits precision.

# Examples

```
% echo "19.28924560546875 33.5467529296875 berrybush" | geohash lat=1 lon=2 geohash=3 precision=7
19.28924560546875 33.5467529296875 berrybush servers 64.6483612060546875 -147.0017051696777344
%
% echo "othercolumn,-47.17529296875,68.35693359375" | geohash -F, lat=3 lon=2
othercolumn,-47.17529296875,68.35693359375,funkys000000
```

Note that invalid characters in geohash (invalid symbols in
coordinates) are treated as zero.
```
% echo dog | geohash geohash=1 | geohash lat=2 lon=3 precision=3
dog 4.2187500000000000 -85.7812500000000000 d0g
%
% echo "0 5\ndog 5" | geohash lat=1 lon=2
0   5 s0581b0bh2n0
dog 5 s0581b0bh2n0
```

# Benchmarking

Comparing to an equivalent python script the awk version works ~3
times slower. I attribute it to less efficient bitwise operations in
awk.

```
% cat > script.py
import geohash

with open('test.csv', 'r') as f:
    for line in f:
        print(geohash.encode\
              (*[float(x) for x in line.rstrip().split(',')]))
%
% wc -l test.csv
1000000 test.csv
% /bin/time -v geohash -F, lat=1 lon=2 test.csv > /dev/null
    ...
    Elapsed (wall clock) time (h:mm:ss or m:ss): 0:12.06
    ...
    Maximum resident set size (kbytes): 4996
    ...
% /bin/time -v python3 script.py > /dev/null
    ...
    Elapsed (wall clock) time (h:mm:ss or m:ss): 0:04.00
    ...
    Maximum resident set size (kbytes): 8580
    ...
```
