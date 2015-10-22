phoenixNet
=====

Download, munge, and transform Phoenix auto-coded event data to daily event-networks.

'phoenixNet' includes a small set of functions that download a specified range of daily
event files from the Phoenix data storage (http://phoenixdata.org/data/current), 
process the data to remove possible duplicate events, subset by actors to focus on
interactions between state governments, and convert the resulting data to a set of daily 
event-networks by event code or event root code.

Please note: this tool does not (currently) save the raw Phoenix daily files to disk.
If you are interested in acquiring and saving the raw data to plot or analyze as 
events rather than network ties, I suggest using the excellent `phoxy' package created
by [Andrew Halterman](https://github.com/ahalterman/phoxy) at Caerus Associates.

The current development of the Phoenix Data Project is a collaborative effort between 
Caerus Associates (Erin Simpson, Andrew Halterman, and John Beieler), Parus Analytics 
(Phil Schrodt), The University of Texas at Dallas (Patrick Brandt), The Cline Center 
for Democracy at the University of Illinois at Urbana-Champaign, and The University 
of Oklahoma. Visit the Phoenix website here: [http://phoenixdata.org/](http://phoenixdata.org/)
and the website of the Open Event Data Alliance here: [http://openeventdata.org/](http://openeventdata.org).

`phoenixNet` is still in a very early stage of development, and is likely to change
significantly over time.

Installation
------------
`devtools::install_github("jrhammond/phoenixNet")`

Usage
-----

'phoenixNet' contains the following function:

* `phoenix_net' intakes a start date, end date, and event aggregation level
  as arguments, and outputs a list of dynamic network objects (tsna::networkDynamic). 
  Each dynamic network describes daily interactions along one CAMEO-coded event category
  or event root category between the 255 ISO-coded states in the international system. 
  These are directed, binarized (1/0) networks, in which a tie between two states i and j
  indicates that state i initiated at least one event of a given type towards state j.
  
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