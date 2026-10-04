--- 1. Memory Optimization ---
Initial Memory:   3.374 MB
Optimized Memory: 0.902 MB
Memory Saved:     73.27%

--- 2. Column Profiling Table ---
                     dtype  missing_count  missing_pct unique_count most_frequent_val  frequency
column                                                                                              
transaction_id      string              0         0.00         5800          TX-10000          2
sale_date           string              0         0.00         5848   INVALID_DATE_99         45
borough             string              0         0.00            8         Brooklyn         768
neighborhood      category              0         0.00            5         Downtown        1248
bldg_class        category              0         0.00            4               C3        1534
lease_contract      string              0         0.00            6  Lease: 12 month        1028
gross_sqft         float32            150         2.50         4935           1204.0           3
sale_price         float32            350         5.83         5650         245123.0           1
year_built           int16              0         0.00            5             1950        1242

--- 3. Duplicate Records ---
Exact duplicates across all columns: 0
Duplicates on key ['transaction_id']: 200
Kept 'first' occurrence; remaining rows: 5800

--- 4. Clean Text Column ---
Before cleaning value counts:
 Brooklyn    742
BROOKLYN     729
Queens       725
Name: borough, dtype: int64

After cleaning value counts:
Brooklyn     2193
Manhattan    2168
Queens       1439
Name: borough_clean, dtype: int64

--- 5. Regex Extraction ---
Unmatched rows without digits: 978 (16.86%)
     lease_contract  lease_term_num
0  Lease: 12 months            12.0
1       Term 24 mos            24.0
2    36 mo contract            36.0

--- 6. Date Parsing ---
Failed / coerced date count: 43
             sale_date         parsed_date  month day_of_week  quarter
0  2022-04-12 14:00:00 2022-04-12 14:00:00      4     Tuesday        2
1  2021-11-03 09:00:00 2021-11-03 09:00:00     11   Wednesday        4

--- 7. Named Groupby Aggregation ---
               transaction_count  mean_price  max_sqft  price_iqr
borough_clean                                                    
Brooklyn                    2062   499214.34    7998.0  580124.50
Manhattan                   2043   498112.56    7995.0  576890.25
Queens                      1355   503418.12    7992.0  583210.75

--- 8. Transform Output ---
  borough_clean  sale_price  diff_from_borough_mean
0      Brooklyn   432100.50               -67113.84
1     Manhattan   980200.25               482087.69
2        Queens   120400.12              -383018.00

--- 9. Pivot Table & Melt Round-Trip ---
Pivot Table:
 bldg_class          A1        B2        C3        R4
neighborhood                                         
Downtown      512400.2  498210.5  489312.1  505412.3
Industrial    495120.4  501230.1  510200.6  492100.8
Midtown       488900.1  504100.2  496700.5  512000.4
Suburbs       503100.8  491200.3  508900.2  499800.1
Uptown        499500.6  515400.8  492300.4  487600.9
Round-trip lossless check: True

--- 10. Split & Merge Comparison ---
Inner Merge rows: 3000 (intersection)
Left Merge rows:  3800 (all left keys)
Outer Merge rows: 4800 (union of keys)
Caught expected validation error: MergeError triggered on duplicate keys.

--- 11. Rolling Mean & Group Rank ---
  neighborhood         parsed_date  sale_price  rolling_7row_mean  price_rank_in_group
0     Downtown 2021-01-01 02:00:00   320140.00          320140.00                742.0
1     Downtown 2021-01-01 19:00:00   680450.00          500295.00                271.0
2     Downtown 2021-01-02 04:00:00   150200.00          383596.67                980.0

--- 12. Suspicious Rows Audit ---
Row TX-10482: Extremely low price ($12.00) indicates an unrecorded consideration or deed gift transfer.
Row TX-12940: Missing gross square footage on high-value asset ($2,840,192.00) prevents per-sqft pricing.
Row TX-15821: Unparseable date string 'INVALID_DATE_99/99' prevents proper temporal sequencing.

--- 13. Storage & Speed Benchmark ---
CSV:     Size = 558.14 KB | Load Time = 13.92 ms
Parquet: Size = 162.30 KB | Load Time = 2.85 ms
Parquet size reduction: 70.92%
Parquet read speedup:   4.88x faster
