# ZIPs to county finder

This is an effort to convert most ZIP codes into a county, an effort that's deceptively complex.

There are the obvious problems of ZIP codes themselves, as they don't really represent a geography so much as a route. Some might represent a building. Or post office boxes.

But as much as they can represent geographies, it's often one that's a little nutty, e.g., ZIP codes can cross county borders and even state borders. This makes some reasonable assumptions, such as the county with the most land in a ZIP code should get to claim that ZIP code.

## Methodology
Some of the methodology is partially documented in the notebooks that drive this. On to usage!

## Usage
This was built for another project in Python. The main file is `geo/zip-lookup.csv`, intended as a lookup table you could import into a program of your choice. Note that a small percentage of ZIP codes are not in this dataset at all, perhaps largely the ZIP codes that represent a building or a post office. However you implement it, you'll want to make sure you don't drop rows that don't have matches.

Some code to make things easier:

```
import csv


def get_zip_lookup():
    """Build a ZIP/ZCTA lookup table, if needed
    
    Arguments:
        None
    Returns:
        global dictionary named ziplookup
    Uses:
        zip-lookup.csv from mable-raw.csv via parse-zips.ipynb
    """
    if 'ziplookup' in globals():
        pass
        # print("ZIP lookup table already initialized.")
    else:
        print("ZIP lookup table being initialized")
        global ziplookup
        ziplookup = {}
        with open("zip-lookup.csv", "r", encoding="utf-8") as infile:
            reader = csv.DictReader(infile)
            for row in reader:
                ziplookup[row['zip_code']] = row
    return()


def add_county_details(row: dict):
    """Take a list returned from the API or read from a CSV, and append geographic details.
    
    Arguments:
        row, a dictionary
    Returns:
        row, still a dictionary
    """
    get_zip_lookup()            # Read in table    
    if "geo_fips" not in row or row['geo_fips'] == "Unknown":     # If we need to do a lookup
        if not row["placeOfPerformanceZIPCode"]:
            for item in ["geo_fips", "geo_county_name", "geo_zip_name"]:
                row[item] = "Unknown"
        else:
            rowzip = row['placeOfPerformanceZIPCode'][0:5]             # Lose the extension for 9-digit ZIP codes
            if rowzip not in ziplookup:     # If we can't look up
                for item in ["geo_fips", "geo_county_name", "geo_zip_name"]:
                    row[item] = "Unknown"
            else:     # We need to look up, and we can look up
                row['geo_fips'] = ziplookup[rowzip]['zip_fips']
                row['geo_county_name'] = ziplookup[rowzip]['zip_county_name']
                row['geo_zip_name'] = ziplookup[rowzip]['zip_place_name']
    return(row)

```

Note in these examples a field of `geo_fips` set to *Unknown* will lead to the lookup being done again. If you update the geographical data here, you can set all of the `geo_fips` entries to *Unknown* for a fresh match.

## Questions? Problems?

Mike Stucka is exclusively to blame for anything weird or awkward or nonsensical here.
