
# Analysis of airfare data between Frankfurt-New York
------
## Background Information
------
Today’s internet-based distribution of tickets makes it easy for passengers to compare fares and book tickets from different airlines. At the same time, it allows airlines to assess competitors’ prices accurately and in real time. This project presents a method to support airline pricing strategies through the analysis of fares collected by the developed prototype. 

On the route Frankfurt–New York City 54,000 nonstop fare quotes were collected to investigate the dynamics of price-setting behaviour as the departure date draws nearer. The method incorporates a descriptive analysis and a correlational research which are used to reveal patterns or other contiguities within the data set. 
The intertemporal profile of airlines suggests similar pricing strategies across airlines throughout the observation period. Airlines with lower market shares seem to follow the price-setting of airlines with higher market shares. This becomes particularly evident as the departure date approaches. Moreover, the correlational research carried out provides further evidence for competitive analysis in the airline industry as it reveals linear and non-linear correlations between most airlines.

_Keywords: airfare, pricing strategies, price dispersion, fare analysis_

## Data Collection
------
With the observation of a fixed time window on a specific route, it is possible to detected changes within fares as time
passes. This not only allows an effective investigation of pricing behaviour, but offers as well insights into other areas such as flight frequencies and schedules, code-share services or operated aircraft.

| Days before departure 	| Intervals               	|
|-----------------------	|-------------------------	|
| 59 days               	| $(I_0)$                 	|
| 58 days               	| $(I_0, I_1)$            	|
| 57 days               	| $(I_0, I_1, I_2)$       	|
| ...                   	| ...                     	|
| 0 days                	| $(I_0, I_1, ..., I_59)$ 	|

### Interval Sample Size Determination
------
When grouping observations into an interval such as days before departure, a varying number of fare quotes will be collected, which however, usually increases as the departure date draws nearer. Assuming that flights will never be sold out, the smallest interval (e.g. same-day departures) would have the most observations compared to any other interval – the highest interval on the other hand (start of the data collection) will have lowest number of observations.
<img src="figures/FlightCollection.png" />
This is the result of how the method collects data within a fixed time window – in essence, it "fills up" the intervals every day.
As the above Table shows, every collected fare quote presents at some point a different interval ($i_\Delta$) depending only on the date the fare quote was collected.

For instance, assume an identical collection window as provided in the Figure above.
Basically, each $r_n$ belongs to a different interval which is dependent on the collection date. Hence, an interval can be described as follows: 

$I = s − c$  

where

- $I$ : interval (days before departure)
- $s$ : date of the fare (flight departure)
- $c$ : date on which the fare quote was collected (date of purchase)

For example, the fare quotes for the flights departing on 25/02/2015 and returning on 04/03/2015 would fall into two intervals. First, the fare quotes collected on 24/02/2015 would be assigned to the interval hI1i. As for the fare quotes collected a day later (25/02/2015), they would belong to the interval hI0i. Therefore, when collecting fare quotes for 60 days, the initial requests would fill fare quotes into all intervals $(I_0, I_1, ..., I_59)$  whilst on the next day of collection one interval less would be filled $(I_0, I_1, ..., I_58)$. When reaching the last day of collection, the fare quotes collected are same-day flights and therefore within the interval $I_0$. With every day, the amount of fare quotes per interval increases and therefore strengthens both the reliability and the accuracy of the analysis. 

The calculation of observations per day would be problematic and very tedious not only because the number of scheduled flights and published fares per route varies, but also because of the constantly changing availability of flights and fares. 
However, under the assumption of unlimited availability and schedules it is possible to create a positive increasing linear model:

$obsI = f*x + (t*f)$ where $I = s − c$

where

- $obsI$ : observations on day d
- $f$ : fare quotes per day
- $I$ : interval
- $s$ : date of the fare (flight departure)
- $c$ : date on which the fare was collected


## Data Background
------
The collected data set contains fare quotes for a return trip between **FRA–NYC**, spans from 60 to one day prior to the flight departure and contains 54,547 fare observations.
All fare quotes were obtained on a daily basis and always at the same time (midnight 12:00a.m. UTC+1) starting from **February 24 to April 24, 2015 (60 days)**.

Each collected fare applies to a **7-day stay non-stop return trip** and only **economy class** fares are taken into account. All the flights scheduled and offered between February 24 and April 24, 2015 were taken into account for the observation.

During the collection period, six major carriers were offering fares: 
- Lufthansa (LH)
- United Airlines (UA)
- Singapore Airlines (SQ) 
- Delta Air Lines (AL) 
- Air France (AF) 
- KLM Royal Dutch Airlines (KLM) 

However KLM and AF were selling tickets operated by DL only under a code-share agreement – rather than operating non-stop flights themselves. DL flights are not included as this carrier restricted access to its fare quotes for commercial purposes only. Controversially, KLM and AF offered the restricted flights operated by DL under code-share agreements. However, it remains unclear to which extent these prices are similar to those published by DL. A weekly manual verification
during the collection period showed no or only slight differences between the prices of DL, AF and KLM.
The cause of the discrepancy between the summary statistics of AF and KLM can only be partially explained. As DL offered only one service per day on the route FRA–JFK and vice versa, AF and KLM usually offered one fare each too.

However, AF did not offer these fares on several days on what seemed to be random occasions. This could mean that KLM had sold less seats than AF or that the realised SLF was already so high that no more seats were made available to AF.
<p style="text-align:center;margin:10px;"><img src="figures/DataFlightPaths.png" style="padding:10px;" alt="Airlines Operating on FRA-NYC Market (non-stop)"/><em>Airlines Operating on FRA-NYC Market (non-stop)</em></p>

As shown in the Figure all airlines were offering fares on the O-D market FRA–JFK, only two of them, LH and UA, were offering flights to EWR and none had scheduled flights to LGA. LH, UA and SQ are members of the Star Alliance while
DL, KLM and AF are members of SkyTeam and no airline in the data set was without an alliance.

It is notable that LH and UA as well as AF, KLM and DL had existing code-share agreements, which resulted not only in multiple offers for the exact same flight, but also made it possible that each segment of a trip was operated by the other codesharing partner. Multiple carrier flights without code-share agreements between the carriers are excluded from the data set as such flights would include two one-way fares. Within the data set, a specific distinction was made between these two scenarios by recording both the publishing and operating airline of a fare.

## Data Analysis
------
### Data Structure
------
Our data set contains of every request collected, an each request lead to several trips. Each trip as two rows, a trip to the destination and a trip back to the origin of the trip.


```python
# import libraries
import numpy as np
import pandas as pd
from pandas import DataFrame, Series

 # fix random seed to get same numbers
np.random.seed(555)

# path to the csv
path = r'/home/ubuntu/notebooks/AirFare/Data/tripRequest.csv'

# set dataFrame
dataFrame = pd.read_csv(path, 
                        sep=';', 
                        usecols=['trip_id', 'tripStartDeparture', 'tripDestinationDeparture', 'origin', 'destination',
                                 'flightCarrier', 'flightNumber', 'cabin', 'saleTotal', 'mileage', 'meal',
                                 'tripDuration', 'saleTotal', 'operatingDisclosure', 
                                 'tripReturn', 'bookingCode', 'bookingCodeCount'],
                        parse_dates=['tripStartDeparture','tripDestinationDeparture']
                       )
dataFrame.head()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>trip_id</th>
      <th>saleTotal</th>
      <th>tripStartDeparture</th>
      <th>tripReturn</th>
      <th>tripDestinationDeparture</th>
      <th>tripDuration</th>
      <th>cabin</th>
      <th>bookingCode</th>
      <th>bookingCodeCount</th>
      <th>flightCarrier</th>
      <th>flightNumber</th>
      <th>origin</th>
      <th>destination</th>
      <th>operatingDisclosure</th>
      <th>mileage</th>
      <th>meal</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>103973</td>
      <td>477.99</td>
      <td>2015-02-24 13:45:01</td>
      <td>1</td>
      <td>2015-03-03 19:31:05</td>
      <td>540</td>
      <td>COACH</td>
      <td>V</td>
      <td>9</td>
      <td>KL</td>
      <td>6107</td>
      <td>FRA</td>
      <td>JFK</td>
      <td>OPERATED BY DELTA</td>
      <td>3844</td>
      <td>Lunch</td>
    </tr>
    <tr>
      <th>1</th>
      <td>103973</td>
      <td>477.99</td>
      <td>2015-02-24 13:45:01</td>
      <td>1</td>
      <td>2015-03-03 19:31:05</td>
      <td>499</td>
      <td>COACH</td>
      <td>V</td>
      <td>9</td>
      <td>KL</td>
      <td>6106</td>
      <td>JFK</td>
      <td>FRA</td>
      <td>OPERATED BY DELTA</td>
      <td>3844</td>
      <td>Dinner</td>
    </tr>
    <tr>
      <th>2</th>
      <td>103976</td>
      <td>477.99</td>
      <td>2015-02-24 13:45:01</td>
      <td>1</td>
      <td>2015-03-03 19:31:05</td>
      <td>540</td>
      <td>COACH</td>
      <td>V</td>
      <td>9</td>
      <td>AF</td>
      <td>3660</td>
      <td>FRA</td>
      <td>JFK</td>
      <td>OPERATED BY DELTA</td>
      <td>3844</td>
      <td>Lunch</td>
    </tr>
    <tr>
      <th>3</th>
      <td>103976</td>
      <td>477.99</td>
      <td>2015-02-24 13:45:01</td>
      <td>1</td>
      <td>2015-03-03 19:31:05</td>
      <td>499</td>
      <td>COACH</td>
      <td>V</td>
      <td>9</td>
      <td>AF</td>
      <td>3581</td>
      <td>JFK</td>
      <td>FRA</td>
      <td>OPERATED BY DELTA</td>
      <td>3844</td>
      <td>Dinner</td>
    </tr>
    <tr>
      <th>4</th>
      <td>103977</td>
      <td>556.31</td>
      <td>2015-02-24 08:20:01</td>
      <td>1</td>
      <td>2015-03-03 20:25:05</td>
      <td>535</td>
      <td>COACH</td>
      <td>K</td>
      <td>9</td>
      <td>SQ</td>
      <td>26</td>
      <td>FRA</td>
      <td>JFK</td>
      <td>NaN</td>
      <td>3844</td>
      <td>Meal</td>
    </tr>
  </tbody>
</table>
</div>



### Summary
------


```python
# Let's see what basic information we can get out of the data.
dataFrame.describe()
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>saleTotal</th>
      <th>tripReturn</th>
      <th>tripDuration</th>
      <th>bookingCodeCount</th>
      <th>flightNumber</th>
      <th>mileage</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>count</th>
      <td>114080.000000</td>
      <td>114080</td>
      <td>114080.000000</td>
      <td>114080.000000</td>
      <td>114080.000000</td>
      <td>114080.000000</td>
    </tr>
    <tr>
      <th>mean</th>
      <td>862.715791</td>
      <td>1</td>
      <td>496.907556</td>
      <td>5.919968</td>
      <td>4327.491339</td>
      <td>3850.280181</td>
    </tr>
    <tr>
      <th>std</th>
      <td>259.191216</td>
      <td>0</td>
      <td>31.765464</td>
      <td>2.614402</td>
      <td>4022.773340</td>
      <td>6.496310</td>
    </tr>
    <tr>
      <th>min</th>
      <td>446.800000</td>
      <td>1</td>
      <td>455.000000</td>
      <td>1.000000</td>
      <td>25.000000</td>
      <td>3844.000000</td>
    </tr>
    <tr>
      <th>25%</th>
      <td>668.220000</td>
      <td>1</td>
      <td>460.000000</td>
      <td>4.000000</td>
      <td>401.000000</td>
      <td>3844.000000</td>
    </tr>
    <tr>
      <th>50%</th>
      <td>819.540000</td>
      <td>1</td>
      <td>502.500000</td>
      <td>4.000000</td>
      <td>3660.000000</td>
      <td>3844.000000</td>
    </tr>
    <tr>
      <th>75%</th>
      <td>981.460000</td>
      <td>1</td>
      <td>525.000000</td>
      <td>9.000000</td>
      <td>8839.000000</td>
      <td>3857.000000</td>
    </tr>
    <tr>
      <th>max</th>
      <td>2825.220000</td>
      <td>1</td>
      <td>545.000000</td>
      <td>9.000000</td>
      <td>8843.000000</td>
      <td>3857.000000</td>
    </tr>
  </tbody>
</table>
</div>



What we can see here that we have quite a dispersed _saleTotal_ variable, containing the all of our prices (<em>careful: double entries due to the "two-rows-per-trip" constraint</em>). Both, the _tripDuration_ and _mileage_ variables are relitvaily similiar due to the fact that we have only three different airports FRA, JFK and NEW.

The _bookingCodeCount_ variable represents the flight availaibility<sup>1</sup>, which might give us a small glance on the availaibilty of the fare. However, everything above or equal 9 represents a unknown level of availability. Hence, it could be 10 fares left, or even 100.

The _flightNumber_ variable indicates that there is alot of codeshare flights in the database. In most cases you can assume that __international__ codeshare flights have number above 1,000. However, this heaviliy depends on the airlines and might be wrong in other cases.

## Limitations of the Research Design
------
Due to the research design, it is important to note that analysing round-trip fares may only cover a sub-set of the many possible return trips available in theory. A large number of fares are offered for any given departure date dependent on the length of stay. The definition of such a sub-set of fares (7 days return-trip) is necessary for
research design purposes and results are therefore arbitrary. One-way fares were not taken into account for the analysis.

The reason for this is that fares for round-trips offered by full-service carriers (FSCs) are in most cases
cheaper than the sum of the corresponding two one-way fares on the same route. The opposite would be the case if the data set included LCCs as those mainly employ one-way pricing strategies.

Other known limitations which influenced the results of this project are as follows. First, product differentiations were not considered, which, however, may be is captured to some extent by the price of the fare quote. Second, only the cheapest
available fare in the economy class for a specific flight combination were collected – which further limits the data set.  
In order to account for the two the destination airports in the data set, JFK and EWR, it is assumed that both can be treated as city-pairs rather than airportpairs.
Hence, it is possible to treat both airports as one destination.

## Further informations and references
------
<sup>1</sup> Flight Inventory (http://www.travelcodex.com/2012/02/understanding-airline-inventory/)