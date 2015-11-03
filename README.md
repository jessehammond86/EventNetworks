`phoenixNet`
=====

Download, munge, and transform Phoenix and ICEWS auto-coded event data to daily event-networks.

`phoenixNet` includes a set of functions designed to gather event data, format and munge
these data, convert them into daily networks of interactions between states in the 
international system, and return a large set of network-level, dyad-level, and node-level
statistics over time. 

These functions rely on the public release of the [ICEWS data](https://dataverse.harvard.edu/dataverse/harvard?q=icews) for events occurring between 
1995 and October of 2014. For events occurring from June 2014 through the present, they
utilize the [Phoenix daily data sets](http://phoenixdata.org/data/current) created and
released through the [Open Event Data Project](http://openeventdata.org). Note that this
means there are some time periods (currently June-October 2014) where events will be constructed
using *both* Phoenix and ICEWS; in these cases, event records are de-duplicated and some diagnostics
are reported showing the level of overlap between the two data sources.

The main function 'phoenix_net' intakes raw data files. It will look for them on disk, intaking
the paths of Phoenix and ICEWS files separately. Due to API issues, these data are treated differently:
- If Phoenix data are not found in the source path, they are downloaded and saved automatically. This is
done using [Andrew Halterman's](https://github.com/ahalterman/phoxy) excellent `phoxy` package.
- ICEWS data have to be downloaded manually and stored on disk to be used in the functions. The
aforementioned `phoxy' package can be used to automatically access ICEWS data through 2013, but
cannot yet access the most recent year of data. Hopefully, this will be fixed soon once I better 
understand the new version of the Harvard Dataverse API.

The current development of the Phoenix Data Project is a collaborative effort between 
Caerus Associates (Erin Simpson, Andrew Halterman, and John Beieler), Parus Analytics 
(Phil Schrodt), The University of Texas at Dallas (Patrick Brandt), The Cline Center 
for Democracy at the University of Illinois at Urbana-Champaign, and The University 
of Oklahoma. Visit the Phoenix website here: [http://phoenixdata.org/](http://phoenixdata.org/)
and the website of the Open Event Data Alliance here: [http://openeventdata.org/](http://openeventdata.org).

The Integrated Crisis Early Warning System (ICEWS) public-release data is an ongoing data-coding
and analysis initiative hosted at Lockheed Martin, originally funded through DARPA, and more recently 
funded through the Office of Naval Research. ICEWS data is being released monthly through the Harvard
Dataverse, with a 12-month (give or take a few months) lag: as of 11/3/2015, ICEWS data is available
through the end of June 2014. For more information, visit [The W-ICEWS site](http://www.lockheedmartin.com/us/products/W-ICEWS/W-ICEWS_Team/Publications.html) at Lockheed Martin.

`phoenixNet` is still in a very early stage of development, and is likely to change
significantly over time.

Installation
------------
`devtools::install_github("jrhammond/phoenixNet")`

Usage
-----

'phoenixNet' contains the following functions:

* 'phoenix_net' intakes a start date, end date, and event aggregation level
  as arguments, along with a pair of file directories containing Phoenix and ICEWS
  data sets as tabular files. This function outputs a two-element list of data. The
  first element is a data.table object giving daily event counts and overlapping reports
  between the two data sets. The second is a list of dynamic network objects (tsna::networkDynamic). 
  Each dynamic network describes daily interactions along one CAMEO-coded event category
  or event root category between the 255 ISO-coded states in the international system. 
  These are directed, binarized (1/0) networks, in which a tie between two states i and j
  indicates that state i initiated at least one event of a given type towards state j.

* 'phoenix_stats' intakes a list of network objects generated by 'phoenix_net', and generates
  an array of network, dyad, and nodal statistics for each event-day-network in the input.
  The resulting object is a list of lists: layer (1) contains the event or root code,
  and layer (2) contains a set of tables containing statistics of interest organized
  by level of analysis and date.
  
Example:

```
> # Create a set of dynamic event-networks, one for each event root category
> data <- phoenix_net(start_date = 20140620, end_date = 20140622, level = 'rootcode')
trying URL 'https://s3.amazonaws.com/openeventdata/current/events.full.20140620.txt.zip'
Content type 'application/zip' length 179107 bytes (174 KB)
downloaded 174 KB

trying URL 'https://s3.amazonaws.com/openeventdata/current/events.full.20140621.txt.zip'
Content type 'application/zip' length 341968 bytes (333 KB)
downloaded 333 KB

trying URL 'https://s3.amazonaws.com/openeventdata/current/events.full.20140622.txt.zip'
Content type 'application/zip' length 300094 bytes (293 KB)
downloaded 293 KB

Warning message:
In eval(expr, envir, enclos) : NAs introduced by coercion
> # Examine one dynamic network element containing data for event root code 10
> data$code10
NetworkDynamic properties:
  distinct change times: 3 
  maximal time range: 20140620 until  20140622 

Includes optional net.obs.period attribute:
 Network observation period info:
  Number of observation spells: 3 
  Maximal time range observed: 20140620 until 20140622 
  Temporal mode: continuous 
  Time unit: unknown 
  Suggested time increment: NA 

 Network attributes:
  vertices = 255 
  directed = TRUE 
  hyper = FALSE 
  loops = FALSE 
  multiple = FALSE 
  bipartite = FALSE 
  net.obs.period: (not shown)
  total edges= 15 
    missing edges= 0 
    non-missing edges= 15 

 Vertex attribute names: 
    active vertex.names 

 Edge attribute names: 
    active 
> # Extract one day's worth of events as a discrete network object
> daily_network <- network.collapse(data$code10, at = 20140620)
> daily_network
 Network attributes:
  vertices = 255 
  directed = TRUE 
  hyper = FALSE 
  loops = FALSE 
  multiple = FALSE 
  bipartite = FALSE 
  total edges= 7 
    missing edges= 0 
    non-missing edges= 7 

 Vertex attribute names: 
    vertex.names 

No edge attributes

> data2 <- phoenix_stats(data)
> data2$code10$netstats
       date mean_degree      density modularity num_communities mean_commsize cross_tieshare dyadMut dyadAsym
1: 20140620  0.05490196 1.080747e-04  0.4591837               3      3.000000      0.1428571       0        7
2: 20140621  0.03921569 7.719623e-05  0.5600000               3      2.333333      0.0000000       1        3
3: 20140622  0.04705882 9.263548e-05  0.5000000               2      4.000000      0.0000000       0        6
   dyadNull triad003 triad012 triad102 triad021D triad021U triad021C triad111D triad111U triad030T triad030C
1:    32378  2729373     1753        0         6         3         0         0         0         0         0
2:    32381  2730124      758      252         0         0         0         1         0         0         0
3:    32379  2729622     1508        0         2         0         3         0         0         0         0
   triad201 triad120D triad120U triad120C triad210 triad300
1:        0         0         0         0        0        0
2:        0         0         0         0        0        0
3:        0         0         0         0        0        0

