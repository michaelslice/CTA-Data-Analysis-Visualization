# CTA-Data-Visualization

Undergraduate research I conducted alongside Prof.Koop from 2022-2023 looking into inconsistencies in the CTA's predictions for CTA trains on the 'L' line.

## CTA 'L' DATA

The data we have from the CTA comes from a couple of different sources. The main data that we wish to analyze comes from the CTA Train Tracker Map (https://www.transitchicago.com/traintrackermap/). This is tied to the CTA Train Tracker API (https://www.transitchicago.com/developers/traintracker/), but because it is general, does not face the same limits and key constraints. However, the [documentation](https://www.transitchicago.com/developers/ttdocs/) can be useful in deciphering the data.

The General Transit Feed Specification (GTFS) is an open format for transit schedule data. Chicago's [GTFS Data](https://www.transitchicago.com/developers/gtfs/) provides information about its routes, including both bus and "L" routes. We can download this data from the [feed](https://www.transitchicago.com/downloads/sch_data/), and use the various files to extract "L" specific information. The `l_stops` data below is this extracted infromation. Note that a station (`parent_station`) can have multiple stops.

To run the code, you will need the data but also a python installation with support for pandas and pyarrow. Suggest using [Microsoft VS Code](https://code.visualstudio.com) and/or [Anaconda](https://www.anaconda.com/products/distribution). To generate the maps, you will need [shapely](https://shapely.readthedocs.io/en/stable/) which can be installed via anaconda.

There are many python references, but for pandas, the following reference is open-access: <https://wesmckinney.com/book/>

## Deciphering the Data

![image](https://github.com/michaelslice/CTA-Data-Analysis-Visualization/assets/110714088/a2dea1ed-1e8a-4017-b902-1ff1c5ca428b)

The CTA "L" stations seem to be numbered consecutively according to when they were catalogued or added. They are a five-digit number that starts with four (4) and ends with zero (0). Evidence that these correspond to a chronological order includes the last three stops:

* Oakton-Skokie: 41680 (completed Apr. 2012, https://en.wikipedia.org/wiki/Oakton–Skokie_station)
* Cermak-McCormick Place: 41690 (completed Feb. 2015, https://en.wikipedia.org/wiki/Cermak–McCormick_Place_station)
* Washington/Wabash: 41700 (completed Aug. 2017, https://en.wikipedia.org/wiki/Washington/Wabash_station)

The stop ids start with a three (3) and serve to split a "station" into separate tracks/directions. These, too, are numbered based on chronological order, and new stops can be added to an existing (older) station. Evidence:

* 30383 (N) Washington/Wabash (41700), new station (completed Aug. 2017, https://en.wikipedia.org/wiki/Washington/Wabash_station)
* 30384 (S) Washington/Wabash (41700)
* 30385 (S) Wilson-South Outer (40540), new tracks (completed Feb. 2018, https://en.wikipedia.org/wiki/Wilson_station_(CTA))
* 30386 (N) Wilson-North Outer (40540)

## Train Tracker Data

![image](https://github.com/michaelslice/CTA-Data-Analysis-Visualization/assets/110714088/19c0d05f-93d1-4973-bfb4-ece466361422)

In the collected data, we find a third identifier (`CurrentStationId`) which seems to correspond to a fixed location along the track. Perhaps there are sensors at these locations which allow trains to check in at the locations. In any case, we seem to have an imperfect correspondence between the line and the first digit. Note that because lines overlap and trains will run on the same tracks, this reinforces the idea that these are sensors. The second digit is a 1 or 5 which indicates direction (see [Appendix C](https://www.transitchicago.com/developers/ttdocs/#_Toc296199910) of the API Documentation), and the last three digits tend to mostly be ordered.

## Other Elements

The other fields in the entries data include:

* Line: a code for the line (e.g. B for Blue Line)
* Datetime: a timestamp indicating the time the data was collected (UTC)
* Lat/Lng: latitude and longitude that can be used to plot the position on a map
* RunNumber: a number assigned to a train when it is running. Trains can keep the same run number for multiple trips up and down a line.
* ExitStationId: ??? Numbers seem to match the CurrentStationId format
* Direction: the direction the train is traveling (in radians)
* LineName: an expansion of the Line code
* DirMod: seems to be a language element that specifies how the train is moving
* DestName: the end of the trip (usually the end of the line), important when a route splits 
* IsSched: a boolean, generally always False for our data
* CSSClass: used for browser rending
* Flags: ???

## Predictions

![image](https://github.com/michaelslice/CTA-Data-Analysis-Visualization/assets/110714088/f3ae576e-e4dd-46a0-98fb-87896974a304)

For each train, the CTA calculates the number of minutes until it is expected at the next stops on the line. When that number of minutes is less than two, it instead lists the expected time as "Due". The `ParentStop` attribute matches the 4xxxx `parent_station` numbers that are listed above in the GTFS data. Thus, we have a link between the routes that are listed and stops that are predicted. When a prediction has not changed since it's last set of predictions, the data is not recorded.

In general, we would expect that the stops would be listed in order from the next stop to the one after and so on. However, when the stops are close together and thus expected within a minute of each other, the predictions may be the same. In these cases, the stop may be out of order, meaning that the stops as ordered by `StopOrder` are not as they would be encountered by the train.

## Single Train Run

![image](https://github.com/michaelslice/CTA-Data-Analysis-Visualization/assets/110714088/366ef063-55b6-43fc-a0d5-2983f01dc1fc)

## EDF DATA

Information in the CTA train tracker beta comes from data fed to CTA from its rail infrastructure (unlike buses, our current railcar fleet does not have GPS hardware). This data is then processed by software we use to monitor our rail system which also generates the predictions for train arrivals based on recent train travel times from one point to another. (The software is a product called QuicTrak®.)

## Geo Pandas

![image](https://github.com/michaelslice/CTA-Data-Analysis-Visualization/assets/110714088/0f62f72d-ff6c-4394-b5bc-8fc331233d02)

Using Geo Pandas we can coordinate the trains location using CurrentStationId's in the EDF data, to the scheduled destination of ParentStops in l_stops 

## Parent Station Ids Sorted by Line and Chronological Order Mapping

![image](https://github.com/michaelslice/CTA-Data-Analysis-Visualization/assets/110714088/46545d0d-6a39-4795-ba50-dab49df694c6)

## Null Island Encounter

![image](https://github.com/michaelslice/CTA-Data-Analysis-Visualization/assets/110714088/cfe5a089-4e63-41a9-aaf3-a997d2836d74)

In the train tracker data (edf) some of the CurrentStationId's are routing to Null Island which is at the Earth's surface at zero degrees latitude and and zero degrees longitude (O N, O E)

 ## l_stops & Stations Nearest Joins (sjoin_nearest)

![image](https://github.com/michaelslice/CTA-Data-Analysis-Visualization/assets/110714088/f20f2201-9c36-4ae3-88a1-c74e30bcd489)

Using sjoin_nearest we can configure the data set to connect CurrentStationId's to their closet parent_station along the line

## Null Island Points

![image](https://github.com/michaelslice/CTA-Data-Analysis-Visualization/assets/110714088/c3c0f602-5474-475d-9bd4-803d57e2a75d)

## Standard Deviation of Time in Between Stops on The Blue Line

![image](https://github.com/michaelslice/CTA-Data-Analysis-Visualization/assets/110714088/dfabce12-0bed-4775-842b-303d7517d87b)

## Time to Travel Around all Stops in the Loop

![image](https://github.com/michaelslice/CTA-Data-Analysis-Visualization/assets/110714088/fe2d8e49-c8e8-48c4-8a58-83af56bcf86f)

