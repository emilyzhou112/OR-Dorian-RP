# Metadata
Organize and store documentation and metadata in this folder.

Metadata files should be listed for relevant data sources in [data/data_metadata.csv](../data_metadata.csv)

# Twitter Data for Hurricane Ida

This data was acquried with the `rtweet` package and `search tweets` Twitter API with five searches.

The `tevent_raw`, `tevent_raw2`, `tevent_raw3`, `tevent_raw4` data frame contains tweets for hurricane ida, searched on September 2, 5, 10 in 2021 with the following code. The first three searches correspond to the first code chunk and the last search correspond to the second code chunk. This data is searched provided by Professor Jospeh Holler for replication.

```r
tevent_raw <- search_tweets("ida OR hurricane",
  n = 200000, include_rts = TRUE,
  token = twitter_token,
  geocode = "36,-87,1000mi",
  retryonratelimit = TRUE)
```

```r
tevent_raw4 <- search_tweets("ida OR flood OR electricity OR recovery OR outage",
  n = 200000, include_rts = TRUE,
  token = twitter_token,
  geocode = "36,-87,1000mi",
  retryonratelimit = TRUE)
```

The `tdcontrol_raw` data frame contains tweets without any text filter for the same geographic region, searched after the hurricane with the following code:

```r
tdcontrol_raw <- search_tweets("-filter:verified OR filter:verified",
  n = 200000, include_rts = FALSE,
  token = twitter_token,
  geocode = "32,-78,1000mi",
  retryonratelimit = TRUE
)
```

Note that the code requires a valid `twitter_token` object in order to run correctly, and the `search_tweets` function cannot conduct a historical search. If you need to reproduce these results, you will need historic access to archived twitter data, and some tweets may have been edited or removed since the search was conducted.

Following the search, the data was also filtered, combined, reorganized, and saved for analysis and future reproduction with the following code.

```r
tevent_raw <- dplyr::union(tevent_raw1, tevent_raw2)
tevent_raw <- dplyr::union(tevent_raw, tevent_raw3)

tevent <- tevent_raw %>%
  lat_lng(coords = c("coords_coords")) %>%
  subset(place_type == "city" | place_type == "neighborhood" |
    place_type == "poi" | !is.na(lat)
  ) %>%
  lat_lng(coords = c("bbox_coords"))

  tdcontrol <- tdcontrol_raw %>%
    lat_lng(coords = c("coords_coords")) %>%
    subset(place_type == "city" | place_type == "neighborhood" |
      place_type == "poi" | !is.na(lat)
    ) %>%
    lat_lng(coords = c("bbox_coords"))

write.table(tevent$status_id,
  here("data", "derived", "public", "teventids.txt"),
  append = FALSE, quote = FALSE, row.names = FALSE, col.names = FALSE
  )
write.table(tdcontrol$status_id,
  here("data", "derived", "public", "tdcontrolids.txt"),
  append = FALSE, quote = FALSE, row.names = FALSE, col.names = FALSE
  )
```
