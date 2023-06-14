# Demo Project for Issue - Duplicate entries in ALP

This CAP-SFLIGHT project was updated to demonstrates an issue we are having with our company project.

The issue we are having is with sorting on an aggregation property. We have an "count distinct" aggregation property on the main entity for the page. We found that when sorting the table on this property the returned results are not correct on Hana. The same queries are correct on SQLite. There appears to be a bug in Hana processing of the request.

The problem can be demonstrated with the following queries:

-   http://localhost:4004/analytics/Bookings?$apply=filter(airline%20eq%20%27GA%27%20and%20status%20eq%20%27B%27)/groupby((BookingID,FlightDate,ID,TravelID),aggregate(FlightPrice,countTravelID))/orderby(countTravelID%20desc)&$skip=100&$top=4
-   http://localhost:4004/analytics/Bookings?$apply=filter(airline%20eq%20%27GA%27%20and%20status%20eq%20%27B%27)/groupby((BookingID,FlightDate,ID,TravelID),aggregate(FlightPrice,countTravelID))/orderby(countTravelID%20desc)&$skip=102&$top=4

Note that the only difference in the above two URLs is the "skip" parameter is different by 2. With 4 items returned, we expect to get back two sets of data that overlap by 2. This works correctly in SQLite but it is not working as expected in Hana. In Hana, the results from both those queries returns the same set and modifying the "skip" and "top" further gets similar strange results.

To reproduce this issue, you will need to deploy the project to BTP with a Hana database. I use "hybrid" mode to execute the above queries on the Hana database. Compare the use of "skip" and "top" between Hana and SQLite using the above URLs. Below, I provide more details on the changes I made to SFlight and more info.

This issue is not related to SAPUI5. Just query the data directly using the URLs. No need to open the application. Within the application, the problem becomes evident when expanding the groupings in the table and then scrolling down multiple times. The scroll-loading of additional data is not showing expected results when sorting by `countrTravelID` ("TEST COUNT"). It uses "skip" and "top" as described earlier.

Again, this works perfectly on SQLite but results are different when deployed to Hana. In our project, we want to have default sorting on a similar aggregation parameter so that the group with the highest aggregated number is at the top of the list.

---

## Changes to original project

I started with the original CAP-SFlight project and made only a few changes.

The ALP data model `Bookings` was updated to have a new field with an aggregation method of "count distinct".

`analytics-service.cds` has a new field called `countTravelID` on line 16.

`annotations.cds` has the additional aggregation properties on lines 6, 12, 30 and 35-41.

---

## Version Info

Output from `cds version`:

-   @sap/cds: 6.8.2
-   @sap/cds-compiler: 3.3.2
-   @sap/cds-dk: 6.7.2
-   @sap/cds-dk (global): 6.7.2
-   @sap/cds-foss: 4.0.0
-   @sap/cds-mtx: -- missing --
-   @sap/xssec: 3.2.17

---

## Running locally with SQLite

### Running Node.js with SQLite

In the root folder of your project, run

```
npm ci
cds watch
```

## Running on Hana

### Build the project from the command line:

```
mbt build
```

### Deploy

1. Log in to the target space.
2. Deploy the MTA archive using the CF CLI: `cf deploy mta_archives/capire.sflight_1.0.0.mtar`
